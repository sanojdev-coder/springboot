# springboot

#### Build your Spring Boot / Java microservice JAR locally

mvn clean package -DskipTests
-- output--
target/my-service-0.0.1-SNAPSHOT.jar

#### Copy the JAR to the Ubuntu VM
scp target/my-service-0.0.1-SNAPSHOT.jar sanojz@192.168.56.103:/home/sanojz/my-service/
-- output--

ssh sanojz@192.168.56.103
cd ~/my-service
ls
``

#### Create a Docker image on the Ubuntu VM

-- nano Dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY my-service-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
-- output--
Dockerfile

#### Build Docker Image

docker build -t my-service:1.0 
-- output--
docker images | grep my-service

#### Push the image to your local Kubernetes registry

-- registry running ?
default   registry-7c4dd6644-4t9jk   Running
> then
docker tag my-service:1.0 localhost:5000/my-service:1.
-- output --
docker push localhost:5000/my-service:1.0

#### Deploy the microservice to K3s
-- nano my-service-deployment.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-service
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
        - name: my-service
          image: localhost:5000/my-service:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080

kubectl apply -f my-service-deployment.yaml
-- output--
kubectl get pods
kubectl describe pod <pod-name>

#### Expose Service

-- nano my-service-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-service
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080

-- output--
kubectl apply -f my-service-svc.yaml

#### Access from Windows host
-- output--
From your Windows browser:
Shell http://192.168.56.103:30080

