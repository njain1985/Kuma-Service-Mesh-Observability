MICROSERVICES OBSERVABILITY With SERVICES MESH and NEW RELIC ONE (NR1)
======================================================================

![Solution](https://user-images.githubusercontent.com/25683435/96712545-0958f800-13eb-11eb-8795-d6d46656d3ad.png)

In this Nerd Day session, we will provide you the fundamentals of Kuma Service Mesh, NR1, K8s and Open Source Metrics, Tracing and Logging

### Why should you care?
1. Fragmentation of Traces, Metrics and Logs
2. Reduce the MTTXs for degradations/exceptions with your microservices 
3. Minimize hops and Dev/Ops resource fatigue optimization (1 O11y console vs 7 GUIs)


## INTRODUCTIONS
-------------------------------------------

In this directory, you will find the necessary files to get Kuma Service Mesh up and running in a K8s cluster (Minikube used here)

When running on Kubernetes, Kuma will store all of its state and configuration on the underlying Kubernetes API Server, therefore requiring no dependency to store the data.

### Table of Contents
* Kubernetes Basic Commands (Cheat sheet)
* Kuma Service Mesh (Deployment How-tos)
* New Relic One (Deployment How-tos)
* Deploy Kuma Marketplace full-stack app (Vue.js Browser Service, NodeJS App service, PostgreSQL and Redis)
   Note that the Kuma Marketplace has been pre-instrumented with New Relic FSO (APM and Browser)
   Docker hub public image used: [monitorjain/kuma-demo-frontend:v3](https://hub.docker.com/r/monitorjain/kuma-demo-frontend) and [monitorjain/kuma-demo-backend:latest](https://hub.docker.com/repository/docker/monitorjain/kuma-demo-backend)
5. Deploy New Relic [Helm Chart](one.newrelic.com) for K8s (deploys prometheus, logs, traces, metadata injector, daemonset, native events collector etc)

### Setup Environment
-------------------------------------------
* Optional: Install kubectl - command line tool to run commands against a K8s cluster.
* You can leverage this CLI to deploy apps, inspect and manage cluster resources, and view logs. 
* For a complete list you can refer this [resource](https://kubernetes.io/docs/reference/kubectl/overview/)
* MacOs: brew install kubectl | [Other OS](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Verify install: kubectl version --client

### Bonus: K8s basic commands
-------------------------------------------
* To deploy a resource (service/pod/deployment/replicaset) on K8s: kubectl apply -f . 
* To check cluster version: kubectl cluster-info
* To check nodes: kubectl get nodes
* To get a certain pod: kubectl get pod pod1
* To get a certain pod: kubectl get pod pod1 --all-namespaces
* To get all resources on a specific namespace: kubectl get all -n default
* To check logs: kubectl logs pod/pod_name -n kuma-demo
* To check specific logs: kubectl logs POD [-c CONTAINER] [--follow] [flags]
* To check pod status: kubectl get pods --watch 


## 1.0 Setup Minikube K8s Cluster
-------------------------------------------

## 1.1 [Install Minikube](https://minikube.sigs.k8s.io/docs/start/)
-------------------------------------------
* MacOS: brew install minikube
* [Other OS](https://minikube.sigs.k8s.io/docs/start/) 

## 1.2 Start your Minikube K8s Cluster
-------------------------------------------
	minikube start --vm-driver=hyperkit -p kuma-demo --cpus=3 (min) --memory=8192 
- Wait until you see this output:
	Done! kubectl is now configured to use "kuma-demo"
* Note: You may also leverage [EKS](https://console.aws.amazon.com/eks/home?region=us-east-1#), [GKE](https://console.cloud.google.com/) or [AKS](https://portal.azure.com/?quickstart=true#home). 

## 1.3 Deploy the Marketplace full stack app (Vue.js Browser, NodeJS App, Redis and PostgreSQL)
-------------------------------------------

![App Architecture](https://user-images.githubusercontent.com/25683435/96713912-1ecf2180-13ed-11eb-8fa7-15889043466a.jpg)

- Run the following command to deploy the marketplace application via bit.ly/demokuma, which points to the [all-in-one YAML](https://github.com/njain1985/Kuma-Service-Mesh-Observability/blob/main/full_stack_app_with_NR_FSO/kuma-aio.yaml) file provided in this directory:

        $ kubectl apply -f kuma-aio.yaml
    

- And then check the pods are up and running by getting all pods in the kuma-demo namespace:

    	$ kubectl get pods -n kuma-demo
    	NAME                                   READY   STATUS    RESTARTS   AGE
    	kuma-demo-app-69c9fd4bd-4lkl7          1/1     Running   0          40s
    	kuma-demo-backend-v0-d7cb6b576-nrl67   1/1     Running   0          40s
    	postgres-master-65df766577-qqqwr       1/1     Running   0          40s
    	redis-master-78ff699f7-rsdfk           1/1     Running   0          40s


- If you are on a cloud K8s, an external IP and Port will be auto-generally (Frontend Service is LoadBalancer Type), if you're on a Minikube, simply port-forward your frontend service:

    	kubectl port-forward service/frontend -n kuma-demo 8080
    	Forwarding from 127.0.0.1:8080 -> 8080
    	Forwarding from [::1]:8080 -> 8080  

- This marketplace application is currently running WITHOUT Kuma. So all traffic is going directly between the services, and not routed through any dataplanes. In the next step, we will download Kuma and quickly deploy the mesh alongside an existing application.



## 2.0 Setup KUMA Service Mesh
-------------------------------------------

## 2.1 [Install Kuma](https://kuma.io/install)
-------------------------------------------
* Run the following curl script to automatically detect the OS and download kuma

        curl -L https://kuma.io/installer.sh | sh -

* On K8s, KUBERNETES, Run the following command:

        cd kuma-0.7.1/bin && ls
        $ kumactl install control-plane | kubectl apply -f -
        $ kubectl get pods -n kuma-system (validate that kuma control plane is deployed and running)
        $ kubectl delete pods --all -n kuma-demo (delete existing pods for sidecar injector to kick-in)
        $ kubectl get pods -n kuma-demo -w (this time you'll observe multi-container pods - viola, envoy is ready!)
        $ kubectl port-forward service/frontend -n kuma-demo 8080 (port forward again - only for Minikube)

* When running on K8s, there is no external dependency since its written in Go and has no external dependencies (leverages underlying API server to store its config - universal and straight forward to deploy)

* Now we are ready to deploy some dataplane (envoy proxy side cars: fret not the annotations are included in my kuma-aio.yaml)

* NEXT STEPS: You can now explore the Kuma GUI on port 5681!
        kubectl port-forward service/kuma-control-plane -n kuma-system 5681 (to access KUMA GUI: http://localhost:5681/gui)

* Next, configure kumactl to point to the address where the HTTP API server sits:
        $ ./kumactl config control-planes add --name=minikube --address=http://localhost:5681

* Inspect the dataplanes by leveraging
        $ ./kumactl inspect dataplanes (via CLI)

        http://localhost:5681/gui (visually)

        https://github.com/kumahq/kuma-gui - the GUI is open source



## INTEGRATIONS
-------------------------------------------

## KONG GATEWAY BONUS (OPTIONAL)
* Command to install Kong API G/W
        $ kubectl apply -f https://bit.ly/demokumakong

        $ kubectl get pods -n kuma-demo (validation step)

        export PROXY_IP=$(minikube service -p kuma-demo -n kuma-demo kong-proxy --url | head -1) (To point mkt place requests at Kong)

        echo $PROXY_IP (e.g. http://192.168.64.49:31553)

        Add an Ingress Rule
        $ cat <<EOF | kubectl apply -f - 
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: marketplace
          namespace: kuma-demo
        spec:
          rules:
          - http:
              paths:
              - path: /
                backend:
                  serviceName: frontend
                  servicePort: 8080
        EOF