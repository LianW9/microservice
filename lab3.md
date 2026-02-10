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
-  ![5.21](images/#.png)
-  ![5.22](images/#.png)

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
- Confirm that the file was created witht he correct deployment, 3 Pod replicas, products assigned as labels, and the port.
- ![6.3](images/#.png)

4. 

## Reflection
## Diagram
- see lab assign for specifics
## Cleanup
