MICROSERVICES OBSERVABILITY With SERVICES MESH and NEW RELIC ONE (NR1)
======================================================================

![Solution](https://user-images.githubusercontent.com/25683435/96712545-0958f800-13eb-11eb-8795-d6d46656d3ad.png)

In this Nerd Day session, we will provide you the fundamentals of Kuma Service Mesh, NR1, K8s and Open Source Metrics, Tracing and Logging

![Kuma Mesh Architecture](https://user-images.githubusercontent.com/25683435/96800971-5623e980-1452-11eb-900a-3307997d1ca1.jpeg)

### Why should you care?
1. Fragmentation of Traces, Metrics and Logs
2. Reduce the MTTXs for degradations/exceptions with your microservices 
3. Minimize hops and Dev/Ops resource fatigue optimization (1 O11y console vs 7 GUIs)
4. Have a monitor for your monitoring Infrastructure (Airbus A380 metaphor)

Let's see a demo
[NR1 Service Mesh Demo](loom video to be added)
- Limit special orders
- Protection spamming
- Fault Injection for emulation (Chaos Engineering)
- Circuit Breaking for elegant failing and re-routing


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
* Deploy New Relic [Helm Chart](one.newrelic.com) for K8s (deploys prometheus, logs, traces, metadata injector, daemonset, native events collector etc)

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
- Caution verify the compute power on your laptop before allocating 3 CPUs and 8 Gb memory

- <ins>Wait until you see this output:</ins>
	Done! kubectl is now configured to use "kuma-demo"
* Note: You may also leverage [EKS](https://console.aws.amazon.com/eks/home?region=us-east-1#), [GKE](https://console.cloud.google.com/) or [AKS](https://portal.azure.com/?quickstart=true#home). 

## 1.3 Deploy the Marketplace full stack app (Vue.js Browser, NodeJS App, Redis and PostgreSQL)
-------------------------------------------

Before continuing with the deployment, let's build a docker image with your New Relic Browser Agent baked-in.

Click on this [link](https://github.com/njain1985/Kuma-Service-Mesh-Observability/blob/main/app/) to continue

# Continue 

Welcome back, now, let's add your unique New Relic License Key to the K8s deployment YAML below.

![App Architecture](https://user-images.githubusercontent.com/25683435/96713912-1ecf2180-13ed-11eb-8fa7-15889043466a.jpg)

- Run the following command to deploy the marketplace application via bit.ly/demokuma, which points to the [all-in-one YAML](https://github.com/njain1985/Kuma-Service-Mesh-Observability/blob/main/full_stack_app_with_NR_FSO/kuma-aio.yaml) file provided in this directory:

- [NR1 settings page](https://one.newrelic.com/launcher/account-settings-launcher.account-settings-launcher)

        First open the kuma-aio.yaml or vi kuma-aio.yaml and add the new relic license key

        </ins>You may grab your license key from this and replace in the yaml file:</ins>
          name: NEW_RELIC_LICENSE_KEY
          value: "<INSERT_LICENSE_KEY>"

        Repeat the license key input step 4 times over

        Next, replace the frontend docker app image name with the image you created in the sub-tutorial (NR browser instrumented docker frontend Vue.js app image) section

        <ins>This will look like:</ins>
          cd full_stack_app_with_NR_FSO/
          vi kuma-aio.yaml
          replace image: monitorjain/kuma-demo-frontend:v3 with image: <hub_user>/<your image name>:<version>

        Next, lets deploy the full stack app on the Minikube or cloud K8s cluster with the following command:
          $ kubectl apply -f kuma-aio.yaml
    

- And then verify that the pods are up and running by executing the following command:

    	$ kubectl get pods -n kuma-demo

        <ins>EXPECTED OUTPUT:</ins>
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

        added Control Plane "minikube"
        
        switched active Control Plane to "minikube"

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

        Now, hit the PROXY_IP URL (i.e. the Ingress Kong API G/W address), the marketplace app should be available there. 


## 3.0 NEW RELIC CENTRALIZED O11Y SETUP
-------------------------------------------
* Click on K8s instrumentation option via User menu ![Setup K8s 0](https://user-images.githubusercontent.com/25683435/96803352-d13bce80-1457-11eb-9587-615acee29dfd.png)

* Fill in the required details and config attributes ![Setup K8s 1](https://user-images.githubusercontent.com/25683435/96803364-d6991900-1457-11eb-8873-09c2063347b2.png)

* Next, copy helm chart deploy commands to clipboard ![Setup K8s 2](https://user-images.githubusercontent.com/25683435/96803366-d862dc80-1457-11eb-8c7f-e61b219669a0.png)
* In case, if you don't have helm installed on your workstation, brew install helm


* Last, validate that the data is received ![Setup K8s 3](https://user-images.githubusercontent.com/25683435/96803368-db5dcd00-1457-11eb-8746-5c872f34ba99.png)
* Note: Your traffic permissions may block data off - skim through your Service Mesh config and settings via the GUI

## 4.0 NEW RELIC ACTIVATION STEPS
-------------------------------------------
* [Create Workload](https://one.newrelic.com/launcher/workloads.home) | [Doc](https://docs.newrelic.com/docs/new-relic-one/use-new-relic-one/workloads/use-workloads#:~:text=one.newrelic.com%20%3E%20Apps,API%20to%20create%20a%20workload.)

* [Explore K8s explorer](https://one.newrelic.com/launcher/k8s-cluster-explorer-nerdlet.cluster-explorer-launcher?) | [Doc](https://docs.newrelic.com/docs/integrations/kubernetes-integration/understand-use-data/kubernetes-cluster-explorer)

* [Explore Entities](https://one.newrelic.com/launcher/nr1-core.explorer) | [Doc](https://docs.newrelic.com/docs/new-relic-one/use-new-relic-one/ui-data/entity-explorer-view-performance-across-apps-services-hosts)

## 5.0 NEW RELIC ADD-ONS
-------------------------------------------
* Ready to use alerts and dashboard templates for Terraform

* Ready to use alert & dashboard JSONs

* Walk through video 

* Please raise an issue if you have recommendations for improvements

* Kuma Mesh - Logging, Tracing and Metrics collection setup

* Advanced Kuma deployment - Global and multi-zone mode

* Advance Kuma specific monitoring templates and alerting strategy

* For leveraging OSS natively on a Mesh - https://github.com/kumahq/kuma-demo/tree/master/kubernetes#prometheus-and-grafana


## OTHER CRITICAL USE-CASES
-------------------------------------------
* Get Meshes

        $ ./kumactl get meshes

* Activate MTLS

        $ cat <<EOF | kubectl apply -f - 
        apiVersion: kuma.io/v1alpha1
        kind: Mesh
        metadata:
        name: default
        spec:
        mtls:
            enabledBackend: ca-1
            backends:
            - name: ca-1
            type: builtin
        metrics:
            enabledBackend: prometheus-1
            backends:
            - name: prometheus-1
            type: prometheus
        EOF

* Validation step

        $ ./kumactl get meshes
        NAME      mTLS           METRICS                   LOGGING   TRACING   AGE
        default   builtin/ca-1   prometheus/prometheus-1   off       off       24m

* Traffic permission

        $ cat <<EOF | kubectl apply -f - 
        apiVersion: kuma.io/v1alpha1
        kind: TrafficPermission
        mesh: default
        metadata:
        namespace: kuma-demo
        name: everything
        spec:
        sources:
        - match:
            kuma.io/service: '*'
        destinations:
        - match:
            kuma.io/service: '*'
        EOF

* Delete free flowing traffic permission (Stop fake spamming reviews from being submitted into Redis)

        $ kubectl delete trafficpermission -n kuma-demo --all

* Scale Replicas - backend-v1 and backend-v2 were deployed with 0 replicas so let's scale them up to one replica to see how traffic routing works:

        $ kubectl scale deployment kuma-demo-backend-v1 -n kuma-demo --replicas=1


        $ kubectl scale deployment kuma-demo-backend-v2 -n kuma-demo --replicas=1