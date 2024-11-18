NOTES:

I'm using docker instead with command:
minikube start --driver=docker

For more info on pods:
kubectl get pods -o wide

VSCode has YAML Red Hat extension that is useful for development

In VSCode settings.json I added:
{
    "yaml.customTags": [
        
    ],
    "yaml.schemas": {
        "kubernetes": "*.yaml"
    }
}

Then had to restart VSCode

Can run with create/apply to create pods:
  * kubectl create -f nginx.yaml

Can generate yaml file with:
  * kubectl run redis --image=redis --dry-run=client -o yaml > redis.yaml
  * can create pod without --dry-run...

kubectl edit (to edit pod?)

https://www.udemy.com/course/learn-kubernetes/learn/lecture/9723234#notes

Replication controllers:

 * kubectl create -f replication-controller.yaml
 * kubectl get replicationcontroller

Replicat set (better):
  * kubectl create -f replica-set.yaml
  * kubectl get replicaset

Scaling
* After updating replicas in file
  * kubectl replace -f replica-set.yaml
* kubectl scale --replicas=6 -f replica-set.yaml
  * kubectl scale --replicas=8 replicaset myapp-replicaset

* kubectl edit replicaset myapp-replicaset
* kubectl scale replicaset myapp-replicaset --replicas=2


* kubectl explain replicaset
* kubectl explain pod

DEPLOYMENTS (rolling updates)

  * wraps around replica sets

  * kubectl create -f deployment-definition.yaml
    * automatically creates replicaset
  * kubectl get deployments

TO SEE ALL AT ONCE
kubectl get all

* kubectl create -f deployments/deployment.yaml
* kubectl create deployment --help

Rollout command
* kubectl rollout status deployment/myapp-deployment
* kubectl rollout history deployment/myapp-deployment

Deployment Strategies
* Recreate (not default, destroys all at once)
* Rolling Update (default)

* Different options for changing a deployment
  * kubectl apply -f deployment-definition.yaml
  * kubectl set image deployment/myapp-deployment \ nginx-container=nginx:1.9.1
    * 
  * kubectl describe deployment
  * when updating a deployment a new replica set is made
  * UNDOING A CHANGE
    * kubectl rollout undo deployment/myapp-deployment

kubectl create -f deployments/deployment.yaml --record
  * Tells kubernetes to records the cause of change

kubectl edit deployment myapp-deployment --record
kubectl set image deployment myapp-deployment nginx=nginx:1.18-perl
kubectl rollout undo deployment/myapp-deployment

Networking

* IP address is assigned to a POD
  * It gets this IP address via an internal network
* Kubernetes expects up to set up networking
  * All containers/PODs can communicate to one another without NAT
  * All nodes can communicate with all containers and vice-versa without NAT

Services

* Enable communication between various components withing and outside the application
* Listens to port and can forward requests to pods
* Types:
  * NodePort (makes internal pod accessable in port on node)
    * Has a port on the service, target port on pod, and node has it's own port (valid ports 30,000-32,767)
  * ClusterIP (virtual IP inside the cluster, to enable service communication)
  * LoadBalancer (distributes load)
* kubectl create -f service-definition.yaml
  * kubectl get services
* Can route traffic to multiple pods and does so randomly
  * If pods on different nodes, kubernetes can make a service that maps across all nodes
* kubectl get svc
* minikube service myapp-service --url

Microservices Application

```
docker run -d --name=redis redis
docker run -d --name=db postgres=9.4
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
docker run -d --name=result -p 5001:80 --link db:db result-app
docker run -d --name=worker --link db:db --link redis:redis worker
```
* Note: `--link` is deprecated and may be removed from docker, this is because docker swarm and networking involves better ways of achieving this

* Steps to do this in k8s
  * Deploy PODs
  * Create Services (ClusterIP)
    * redis
    * db
  * Create Services (NodePort)
    * voting-app
    * result-app
  * Note worker does not need to be exposed
* Setup k8s 
```
kubectl create -f voting-app/voting-app-pod.yaml
kubectl create -f voting-app/voting-app-service.yaml
kubectl create -f voting-app/redis-pod.yaml
kubectl create -f voting-app/redis-service.yaml
kubectl create -f voting-app/postgres-pod.yaml
kubectl create -f voting-app/postgres-service.yaml
kubectl create -f voting-app/worker-app-pod.yaml
kubectl create -f voting-app/result-app-pod.yaml
kubectl create -f voting-app/result-app-service.yaml
```
* View pods/services
```
kubectl get pods,svc
kubectl get all
kubectl get pods --watch
```
* Get URL then access in local browser (separate terminals)
```
minikube service voting-service --url
minikube service result-service --url
```

Above works, but deploying all the pods is cumbersome and would be better to use deployments instead (which includes replica-sets)

* Delete everything and we'll start over with deployments
```
kubectl create -f voting-app/voting-app-deploy.yaml
kubectl create -f voting-app/voting-app-service.yaml
kubectl create -f voting-app/redis-deploy.yaml
kubectl create -f voting-app/redis-service.yaml
kubectl create -f voting-app/postgres-deploy.yaml
kubectl create -f voting-app/postgres-service.yaml
kubectl create -f voting-app/worker-app-deploy.yaml
kubectl create -f voting-app/result-app-deploy.yaml
kubectl create -f voting-app/result-app-service.yaml
```

* View with
```
kubectl get deployments,svc
```

* Get URL then access in local browser (separate terminals)
```
minikube service voting-service --url
minikube service result-service --url
```

* Scale up
```
kubectl scale deployment voting-app-deploy --replicas=3
```

KUBERENTES ON CLOUD
  * Self Hosted / Turnkey Solutions
    * You provision/configure VMs
    * You use scripts to deploy cluster
    * You maintain VMs yourself
    * eg: Kubernetes on AWS using kops or KubeOne
  * Hosted Solutions / Managed Solutions
    * Kubernetes-As-A-Services
    * Provider provisions VMs
    * Provider installs Kubernetes
    * Provider maintains VMs
    * eg: Google Container Engine (GKE) / AKS/ EKS

GOOGLE KUBERNETES ENGINGE (GKE)
  * changed voting-app-service/result-app-service to type LoadBalancer because deploying on cloud env
  * Delete when done

AMAZON ELASTIC KUBERNETES SERVICE (EKS)
  * Need AWS Account
  * IAM Role for Node Group
  * EKS Cluster Role
  * VPC / EC2 Key Pair to SSH to worker nodes
  * Can take up to 10 minutes to make
  * Compute section has option for node Group
  * Create kubectl commands once aws cli is set up and kube config is updated
  * Delete when done

AZURE KUBERNETES SERVICE (AKS)
  * Need Azure account
  * Look up AKS and add cluster
  * Can take up to 10 minutes
  * Clone GH repo into virtual terminal in Azure and create deployments/services
  * Delete when done

Multi Node cluster with Kubeadm
* Requires setting up a POD Network and setting Master node
