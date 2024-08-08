配置k8s：
kubectl apply -f frontend-deployment-service.yaml
kubectl apply -f user-deployment-service.yaml
kubectl apply -f bot-deployment-service.yaml
kubectl apply -f chat-deployment-service.yaml
kubectl apply -f plugin-deployment-service.yaml
kubectl apply -f nginx-deployment-service.yaml
kubectl apply -f redis-deployment-service.yaml

(backend-deployment-service.yaml是旧的单体后端，如果想运行单体请用：
kubectl apply -f frontend-deployment-service.yaml
kubectl apply -f backend-deployment-service.yaml)

删除：
kubectl delete -f frontend-deployment-service.yaml
kubectl delete -f user-deployment-service.yaml
kubectl delete -f bot-deployment-service.yaml
kubectl delete -f chat-deployment-service.yaml
kubectl delete -f plugin-deployment-service.yaml
kubectl delete -f nginx-deployment-service.yaml
kubectl delete -f redis-deployment-service.yaml

(单体：kubectl delete -f frontend-deployment-service.yaml
kubectl delete -f backend-deployment-service.yaml)

或者指定删哪个：
kubectl delete deployment <deployment名> （pods会一同删掉）
kubectl delete service <service名>
千万不要把那个叫kubernetes的service删了

基本概念：
pv和pvc：用来指定一块存储内容的地方，目前只有redis用到，因为我们不把数据库放在k8s上
pod: 可以简单理解为原来的container
deployment: 在这里指定使用什么镜像，一个deployment可以对应一个或多个pods（作负载均衡）
    例如，点进user-deployment-service.yaml，看到replicas: 1,就是pod个数为1，改为replicas: 3,就是3个pod作负载均衡
    指定镜像：
        image: user_service_image:latest    
        imagePullPolicy: Never 
    这两行指定了使用的本地镜像（不是never就会从远端拿镜像），注意不要漏掉latest
    还值得注意的一点是环境变量里对于数据库的环境变量
        env:
        - name: DB_URL
          value: "jdbc:mysql://123.60.187.205:3306/unigpt_user"
    由于数据库并不在k8s上，所以需要通过123.60.187.205访问
service: 和deployment一对一，作为pod的代理入口，对外暴露一个固定的网络地址
    type: NodePort表示能被k8s集群外部访问，type: ClusterIP则不能
    port是service内部用的端口，targetPort是service接到请求后转发到pod上的端口
    例：user_service在docker-compose里是ports:- "8082:8080"
    对应的就是user-deployment-service.yaml里的port: 8082 targetPort: 8080

注意，当image更新时，deployment不会自动更新image，因此CI/CD时，需要首先制作出新的image，再执行那6个delete，再执行那6个apply，才能把k8s上的也更新（todo）

验证是否配置成功，分别输入：
kubectl get pv
kubectl get pvc
kubectl get pods
kubectl get deployments
kubectl get services
查看status是否正常：pv和pvc是BOUND为正常
pod是Running且restarts=0为正常
deployment是available=1为正常