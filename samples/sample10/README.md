## Sample10: Ballerina module with kubernetes annotations

- This sample runs [foodstore](../sample3) as a module.   
- Following files will be generated from this sample.
    ``` 
    $> docker image
    pizza:latest 
    burger:latest  
    
    $> tree target
    target
    ├── Ballerina.lock
    ├── burger.balx
    ├── kubernetes
    │   ├── burger
    │   │   ├── burger-deployment
    │   │   │   ├── Chart.yaml
    │   │   │   └── templates
    │   │   │       ├── burger_deployment.yaml
    │   │   │       ├── burger_ingress.yaml
    │   │   │       ├── burger_secret.yaml
    │   │   │       └── burger_svc.yaml
    │   │   ├── burger_deployment.yaml
    │   │   ├── burger_ingress.yaml
    │   │   ├── burger_secret.yaml
    │   │   ├── burger_svc.yaml
    │   │   └── docker
    │   │       └── Dockerfile
    │   └── pizza
    │       ├── docker
    │       │   └── Dockerfile
    │       ├── foodstore
    │       │   ├── Chart.yaml
    │       │   └── templates
    │       │       ├── pizza_deployment.yaml
    │       │       ├── pizza_ingress.yaml
    │       │       └── pizza_svc.yaml
    │       ├── pizza_deployment.yaml
    │       ├── pizza_ingress.yaml
    │       └── pizza_svc.yaml
    └── pizza.balx
  
    ```
### How to run:

1. Initialize ballerina project.
```bash
sample10$> ballerina init
Ballerina project initialized
```

2. Compile the project. Command to run kubernetes artifacts will be printed on success:
```bash
sample10$> ballerina build 
Compiling source
    john/burger:0.0.1
    john/pizza:0.0.1

Running tests
    john/burger:0.0.1
	No tests found

    john/pizza:0.0.1
	No tests found

Generating executables
    ./target/burger.balx
	@kubernetes:Service 			 - complete 1/1
	@kubernetes:Ingress 			 - complete 1/1
	@kubernetes:Secret 			 - complete 1/1
	@kubernetes:Deployment 			 - complete 1/1
	@kubernetes:Docker 			 - complete 3/3
	@kubernetes:Helm 			 - complete 1/1

	Run the following command to deploy the Kubernetes artifacts:
	kubectl apply -f /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/burger

	Run the following command to install the application using Helm:
	helm install --name burger-deployment /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/burger/burger-deployment

    ./target/pizza.balx
	@kubernetes:Service 			 - complete 1/1
	@kubernetes:Ingress 			 - complete 1/1
	@kubernetes:Deployment 			 - complete 1/1
	@kubernetes:Docker 			 - complete 3/3
	@kubernetes:Helm 			 - complete 1/1

	Run the following command to deploy the Kubernetes artifacts:
	kubectl apply -f /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/pizza

	Run the following command to install the application using Helm:
	helm install --name foodstore /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/pizza/foodstore
```

3. food_api_pkg.balx, Dockerfile, docker image and kubernetes artifacts will be generated: 
```bash
$> tree target
target
├── Ballerina.lock
├── burger.balx
├── kubernetes
│   ├── burger
│   │   ├── burger-deployment
│   │   │   ├── Chart.yaml
│   │   │   └── templates
│   │   │       ├── burger_deployment.yaml
│   │   │       ├── burger_ingress.yaml
│   │   │       ├── burger_secret.yaml
│   │   │       └── burger_svc.yaml
│   │   ├── burger_deployment.yaml
│   │   ├── burger_ingress.yaml
│   │   ├── burger_secret.yaml
│   │   ├── burger_svc.yaml
│   │   └── docker
│   │       └── Dockerfile
│   └── pizza
│       ├── docker
│       │   └── Dockerfile
│       ├── foodstore
│       │   ├── Chart.yaml
│       │   └── templates
│       │       ├── pizza_deployment.yaml
│       │       ├── pizza_ingress.yaml
│       │       └── pizza_svc.yaml
│       ├── pizza_deployment.yaml
│       ├── pizza_ingress.yaml
│       └── pizza_svc.yaml
└── pizza.balx
  

```

4. Verify the docker image is created:
```bash
$> docker images
REPOSITORY                                                       TAG                               IMAGE ID            CREATED             SIZE
pizza                                                            latest                            983a34711d0d        34 seconds ago      125MB
burger                                                           latest                            3b740094c254        35 seconds ago      125MB
```

5. Run kubectl command to deploy artifacts (Use the command printed on screen in step 2):
```bash
$> kubectl apply -f /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/burger
deployment.extensions/burger-deployment created
ingress.extensions/burgerep-ingress created
secret/burgerep-keystore created
service/burgerep-svc created

$ kubectl apply -f /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample10/target/kubernetes/pizza/
deployment.extensions/foodstore created
ingress.extensions/pizzaep-ingress created
service/pizzaep-svc created
```

6. Verify kubernetes deployment, service, secrets and ingress is deployed:
```bash
$> kubectl get pods
NAME                                 READY     STATUS    RESTARTS   AGE
burger-deployment-85448f5797-8wktg   1/1       Running   0          36s
foodstore-7bc59c848b-7lk5d           1/1       Running   0          11s
foodstore-7bc59c848b-8nczc           1/1       Running   0          11s


$> kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
burgerep-svc   ClusterIP   10.107.127.86   <none>        9096/TCP   45s
pizzaep-svc    ClusterIP   10.96.214.133   <none>        9099/TCP   45s

$> kubectl get ingress
NAME               HOSTS        ADDRESS   PORTS     AGE
burgerep-ingress   burger.com             80, 443   1m
pizzaep-ingress    pizza.com              80        43s

```

7. Access the hello world service with curl command:

- **Using ingress:**
Add /etc/hosts entry to match hostname. 
_(127.0.0.1 is only applicable to docker for mac users. Other users should map the hostname with correct ip address 
from `kubectl get ingress` command.)_

```bash
$> curl http://pizza.com/pizzastore/pizza/menu
Pizza menu

$> curl https://burger.com/menu -k
Burger menu
```

8. Undeploy sample:
```bash
$> kubectl delete -f target/kubernetes/pizza
$> kubectl delete -f target/kubernetes/burger
$> docker rmi pizza burger

```
