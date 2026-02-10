# Lab 3 Report - Lian Welch

## Task 1
1. gcloud auth list
- Checking to confirm authentication.
- ![1.1](images/#.png)

2. gcloud config list project
- Confirming that the active project is lianproject-9.
- ![1.2](images/#.png)

3. gcloud config set compute/zone us-east1-b
- Set the defualt compute zone to us-east1-b.
- gcloud config list compute/zone to verify.
- ![1.3](images/#.png)

## Task 2
1. cd ~
- naviagte to working directory.
2. git clone https://github.com/googlecodelabs/monolith-to-microservices.git
- Clone the public Google Codelabs repo, this contains the orders and products microservices.
3. cd ~/monolith-to-microservices
- Navigate to the monolith-to-microservices directory.
- ![2](images/#.png)

## Task 3
1. gcloud services enable \
  cloudresourcemanager.googleapis.com \
  container.googleapis.com
- Enables the required Google Kubernetes Engine (GKE) API.
- ![3.1](images/#.png)

2. gcloud services enable containerregistry.googleapis.com
- Enables the container registry API.
- ![3.2](images/#.png)

3. gcloud services list --enabled | grep containerregistry.googleapis.com
- Check that the container registry API is enabled.
- ![3.3](images/#.png)

4. gcloud container clusters create gke-cluster \
  --num-nodes=3 \
  --machine-type=e2-medium \
  --disk-type=pd-balanced \
  --disk-size=30 \
  --zone us-east1-b
- Create a GKE cluster named gke-cluster with 3 worker nodes.
- ![3.4](images/#.png)

5. gcloud container clusters list
- Lists all clusters in the project, which is just gke-cluster
- ![3.5](images/#.png)

6. gcloud container clusters get-credentials gke-cluster \
  --zone us-east1-b
- Retrieves credentials so that kubectl to enable secure communcation with the cluster by retrieving the cluster endpoint, download authentication credentials, and updates local kubeconfig file.
- ![3.6](images/#.png)

7. kubectl get nodes
- Verfiy that the cluster is connected and accessible, which is shown by three worker nodes in the Ready state.
- ![3.7](images/#.png)

## Task 4
1. gcloud artifacts repositories list --location=us-east1
- Check the name of the repo, lianproject-9, location us-east1.
- ![4.1](images/#.png)

2. cd ~
gcloud artifacts repositories list --location=us-east4
- I used us-east4 in lab 1, where my-repository can be found.
- ![4.2](images/#.png)

3. gcloud artifacts repositories describe my-repository --location=us-east4
- Displayed the repository to conrim the output and region.
- registryUri: us-east4-docker.pkg.dev/lianproject-9/my-repository
- ![4.3](images/#.png)

4. cd ~
- Navigate to home directory
- ![4.4](images/#.png)

5. echo 'export ART_REG=$(gcloud artifacts repositories describe my-repository --location=us-east4 --format="value(registryUri)")' >> ~/.bashrc
- Automaticaaly set in every Cloud session.
- ![4.5](images/#.png)

6. source ~/.bashrc
- Reloads the shell
- ![4.6](images/#.png)

7. echo $ART_REG
- Verify the environmental variable ART_REG
- See screenshot below

8. env | grep ART_REG
- Verify the environmental variable ART_REG
- ![4.8](images/#.png)

## Task 5
1. cd ~/monolith-to-microservices/microservices/src/products pwd ls
- Naviagte to products directory
- Report the current directory path
- List the contents
- ![5.1](images/#.png)

2. gcloud builds submit --tag "$ART_REG/products:v1" .
- Build and push the Products image by using the current directory as the build context for; uploading the source code and DockerFile, automaic Build the Docker image, and push the image to artifact Registry.
- gcloud projects add-iam-policy-binding lianproject-9 --member="user:lianrothe@gmail.com" --role="roles/cloudbuild.builds.editor"
- The command above was needed first in order to enable the IAM.
-  ![5.2.1](images/#.png)
-  ![5.2.2](images/#.png)

3. cd ~/monolith-to-microservices/microservices/src/orders pwd ls
- Navigate to orders directory
- ![5.3](images/#.png)

4. gcloud builds submit --tag "$ART_REG/orders:v1" .
- Build and push the orders image
- ![5.4](images/#.png)

5. gcloud artifacts docker images list $ART_REG --include-tags
- Verify that the images exist in the Artifact Registry and view the tags
- ![5.5](images/#.png)

## Task 6
1. cd ~ mkdir -p gke-microservices-manifests cd gke-microservices-manifests pwd
- Starting from home directory, create a folder to hold the manifests, and navigate to it.
- ![6.1](images/#.png)

2. cat > products-deployment.yaml - with file contents
- Creates the file products-deployment.yaml with the properties seen below
- ![6.2](images/#.png)

3. cat products-deployment.yaml
- Confirm that the file was created with the correct deployment, 3 Pod replicas, products assigned as labels, and the port.
- ![6.3](images/#.png)

4. cat > products-service.yaml - with file contents
- Create the Service manifest.
- ![6.4](images/#.png)

5. cat products-service.yaml
- Verfiy that the file was created as a Service, named product-service, with ClusterIp for the IP address, and the ports match (8082).
- ![6.5](images/#.png)

6. cd ~/gke-microservices-manifests kubectl apply -f products-deployment.yaml kubectl apply -f products-service.yaml
- Navigate to the gke-microservices-manifests directory.
- Apply the Deployment
- Apply the Service
- ![6.6](images/#.png)

7. kubectl get deployment products
- Verify that the product microservice Deployment was created, as seen with READY showing 3/3
- ![6.7](images/#.png)

8. kubectl get rs -l app=products
- Verify the ReplicaSet created by the Deployment, which can be seen with one ReplicaSet with the products Deployment.
- ![6.8](images/#.png)

9. kubectl get pods -l app=products \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP,NODE:.spec.nodeNameame
- Verify Pods created by the ReplicaSet, with 3 Pods running for products, each pod has its own Pod IP, and all Pods share app=products
- ![6.9](images/#.png)

10. kubectl delete pod products-696697b867-hcnrx
- Deletes one of the Pods and the Kubernetes automaitcally restores the wanted replica count.
- ![6.10](images/#.png)

11. kubectl get pods -l app=products \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP,NODE:.spec.nodeNameame -w
- A second terminal shows the Pods in real time.
- ![6.11](images/#.png)

12. kubectl get pods -l app=products \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP,NODE:.spec.nodeNameame
- This shows the three running Pods again, including the new one with a different IP.
- This shows that the ReplicaSet automatically replaced the deleted Pod.
- ![6.12](images/#.png)

13. kubectl get pods -l app=products \
  -o custom-columns=NAME:.metadata.name,LABELS:.metadata.labels,IP:.status.podIP
- This confirms the labels on the products Pods
- ![6.13](images/#.png)

14. kubectl describe svc products-service | grep Selector
- Displays the Service selector for product-service, which defines which Pods recieve from products-service.
- ![6.14](images/#.png)

15. kubectl describe deployment products | grep Selector
- Shows the Deployment selector, which defines which Podws belong to this Depolyment's ReplicaSet.
- ![6.15](images/#.png)

16. kubectl run dns-client --image=busybox:1.36 --restart=Never -- sleep infinity
- Create the DNS client pod
- ![6.16](images/#.png)

17. kubectl get pod dns-client
- Verify that it is running
- ![6.17](images/#.png)

18. kubectl run http-client --image=curlimages/curl:8.5.0 --restart=Never -- sleep infinity
- Create the HTTP client pod
- ![6.18](images/#.png)

19. kubectl get pod http-client
- Verify that it is running.
- ![6.19](images/#.png)

20. kubectl exec -it dns-client -- nslookup products-service
- Resolve the service name from the DNS client pod
- ![6.20](images/#.png)

21. kubectl exec -it dns-client -- nslookup products-service.default.svc.cluster.local
- Resolve the full in-cluster FQDN by confirming that kubedns is automatically publishing a DNS entry for the products-service Service.
- ![6.21](images/#.png)

22. kubectl exec -it http-client -- curl -s http://products-service/api/products
- Displays the complete list of products.
- ![6.22](images/#.png)

23. kubectl exec -it http-client -- curl -s http://products-service/api/products/0PUK6V6EV0
- This retrieves the vintge record player using its product ID
- ![6.23](images/#.png)

## Task 7
1. cd ~/gke-microservices-manifests
- Start in the manifests directory
- ![7.1](images/#.png)

2. cat > orders.yaml - with file information
- Create orders.yaml
- ![7.2](images/#.png)

3. cat orders.yaml
- Verify that the orders.yaml manifest is correct for the orders microservice with port 80 and targetPort 8081.
- ![7.3](images/#.png)

4. cd ~/gke-microservices-manifests
- Navigate to gke-microservices-manifests
- ![7.4](images/#.png)

5. kubectl apply -f orders.yaml
- Deploy Deployment and Service in a single file.
- ![7.5](images/#.png)

6. kubectl get deployment orders
- Check that the Deployment is created and ready
- ![7.6](images/#.png)

7. kubectl get pods -l app=orders \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP,NODE:.spec.nodeNameame
- Check that three orders Pods are running
- ![7.7](images/#.png)

8. kubectl get svc orders-service
- Check that the orders service exists and has a ClusterIP Service named orders-service.
- ![7.8](images/#.png)

9. kubectl exec -it dns-client -- nslookup orders-service
- Verify in-cluster DNS for orders-service from the in-cluster DNS test Pod.
- Name:   orders-service.default.svc.cluster.local
- ![7.9](images/#.png)

10. kubectl exec -it http-client -- curl -s http://orders-service/api/orders | head
- Verify the HTTP access to the orders microservice by retrieving the list of orders.
- ![7.10](images/#.png)

11. kubectl exec -it http-client -- curl -s http://orders-service/api/orders/ORD-000001-MICROSERVICE
- Retrieves a single order showing that the Deployment was successfull, the Service correctly selected the Pods, The CoreDNS resolved the Service name from inside the cluster, the Service name can be used instead of Pod IPs, and the port mapping in Service is correct.
- ![7.11](images/#.png)

12. kubectl get pods -l app=orders \
  -o custom-columns=NAME:.metadata.name,IP:.status.podIP && \
kubectl get svc orders-service \
  -o custom-columns=SERVICE:.metadata.name,CLUSTERIP:.spec.clusterIP && \
kubectl get endpointslice -l kubernetes.io/service-name=orders-service \
  -o custom-columns=NAME:.metadata.name,ENDPOINTS:.endpoints[*].addresses
- The orders Pods, Service, and EndpointSlice relate to each other when the Pod Ips match the addresses in the EndpointSlice block and the Service ClusterIP remains constant.
- ![7.12](images/#.png)

## Reflection - to do
1 Explain the relationship between Deployment, ReplicaSet, and Pods.

Specifically:

What does the Deployment control?
What role does the ReplicaSet play?
Why are Pods considered ephemeral, and how did you observe this in the lab when you deleted a Pod?

2. How does a Kubernetes Service (ClusterIP) enable stable access to a microservice?

In your answer, describe:

Why clients use the Service name (e.g., products-service or orders-service) instead of Pod IPs
How labels and selectors connect the Service to the correct Pods
What would break if Services did not exist and clients had to use Pod IPs directly

3. What is an EndpointSlice and why is it important?

Explain in your own words:

What information the EndpointSlice contains
How it relates to the Service and the Pods
What you observed when comparing Pod IPs, Service ClusterIP, and EndpointSlice addresses in Step 7.5

## Diagram - to do
- see lab assign for specifics

## Clean-up
![cleanup](images/#.png)
