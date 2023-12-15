# lab-Implement-DevOps-in-Google-Cloud

# Task 1. Create Environment Variables

export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION=us-central1
gcloud config set compute/region $REGION
export CLUSTER_NAME=hello-cluster

# ENABLE APIs and ADD SERVICE ACCOUNT FOR K8s

gcloud services enable container.googleapis.com \
    cloudbuild.googleapis.com \
    sourcerepo.googleapis.com
    
export PROJECT_ID=$(gcloud config get-value project)
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$(gcloud projects describe $PROJECT_ID \
--format="value(projectNumber)")@cloudbuild.gserviceaccount.com --role="roles/container.developer"

# SETUP GIT CONFIG IN CLOUDSHELL
git config --global user.email $EMAIL
git config --global user.name student

# Create Artifact Registry Docker repository named my-repository 
gcloud artifacts repositories create my-repository \
  --repository-format=docker \
  --location=$REGION \
  --description= "This is my App"

# CREATE GKE CLUSTER VIA THE CONSOLE

Setting Value
Zone us-central1a
Release channel Regular
Cluster version 1.25.4-gke.2100 or newer
Cluster autoscaler Enabled
Number of nodes 3
Minimum nodes 2
Maximum nodes 6

# create two namespaces prod & dev
kubectl create namespace prod	
kubectl create namespace dev

# Create Cloud Source Repositories
gcloud source repos create sample-app
gcloud source repos clone sample-app --project=$PROJECT_ID
cd ~
gsutil cp -r gs://spls/gsp330/sample-app/* sample-app

# Make your first commitand push the changes to the master branch

cd sample-app
 
git init
 
git add .
git commit -m "first commit file"
git push -u origin master

Create a branch named dev
git checkout -b dev
git push -u origin dev

# Task 3. Create the Cloud Build Triggers

Go to the Google Cloud Console.
 
2. Navigate to the Cloud Build Triggers page.
 
3. Click on the Create Trigger button. Fill in the details for the trigger:
Name the trigger: sample-app-prod-deploy
For event select: Push to a branch
Choose the source repository sample-app
Select the branch ^master$
Choose Cloud Build Configuration File as the Build Configuration option and enter the cloudbuild.yaml file location in the repository.
Save the trigger by clicking the Create button.
 
Create the sample-app-dev-deploy trigger
1. Repeat the above steps for the dev branch:
Name the trigger: sample-app-dev-deploy
For event select: Push to a branch
Choose the source repository sample-app
Select the branch ^dev$
Choose Cloud Build Configuration File as the Build Configuration option and enter the cloudbuild-dev.yaml file location in the repository.
Save the trigger by clicking the Create button.

# Task 4. Deploy the first version of the application

In this section, you will build the first version of the production application and the development application.
Build the first development deployment
1. In Cloud Shell, inspect the cloudbuild-dev.yaml file to see the steps in the build process. Fill in the <version> on lines 10 and 15 with v1.0.
 
2. Navigate to the dev/deployment.yaml file and fill in the <todo> on line 17 with the correct container image name.
Simply copy the container image (from line 10 or 15) in the cloudbuild-dev.yaml file and replace the $PROJECT_ID variable with your Project ID. It should look like: us-central1-docker.pkg.dev/qwiklabs-gcp-01-0eb5ce3edc80/my-repository/hello-cloudbuild-dev:v1.0
 
3. Make a commit with your changes on the dev branch and push changes to trigger the sample-app-dev-deploy build job.
 
cd ~/sample-app
git checkout dev
git commit -a -m "update dev cloudbuild and deployment scripts"
git push -u origin dev
 
4. Verify your build executed successfully, and verify the development-deployment application was deployed onto the dev namespace of the cluster.
 
5. Expose the development-deployment deployment to a LoadBalancer service named dev-deployment-service on port 8080, and set the target port of the container to the one specified in the Dockerfile.
 
kubectl expose deployment development-deployment -n dev --name=dev-deployment-service --type=LoadBalancer --port 8080 --target-port 8080
 
6. Navigate to the Load Balancer IP of the service and add the /blue entry point at the end of the URL to verify the application is up and running. It should resemble something like the following: http://34.135.97.199:8080/blue.
 
Build the first production deployment
1. In Cloud Shell, inspect the cloudbuild.yaml file to see the steps in the build process. Fill in the <version> on lines 10 and 15 with v1.0.
 
2. Navigate to the prod/deployment.yaml file and fill in the <todo> on line 17 with the correct container image name.
Simply copy the container image (from line 10 or 15) in the cloudbuild.yaml file and replace the $PROJECT_ID variable with your Project ID. It should look like: us-central1-docker.pkg.dev/<project-id>/my-repository/hello-cloudbuild:v1.0
 
3. Make a commit with your changes on the master branch and push changes to trigger the sample-app-prod-deploy build job.
 
cd ~/sample-app
 
git checkout master
 
git commit -a -m "update cloudbuild and deployment scripts"
 
git push -u origin master
 
 
4. Verify your build executed successfully, and verify the production-deployment application was deployed onto the prod namespace of the cluster.
 
5. Expose the production-deployment deployment to a LoadBalancer service named prod-deployment-service on port 8080, and set the target port of the container to the one specified in the Dockerfile.
 
kubectl expose deployment production-deployment -n prod --name=prod-deployment-service --type=LoadBalancer --port 8080 --target-port 8080
 
6. Navigate to the Load Balancer IP of the service and add the /blue entry point at the end of the URL to verify the application is up and running. It should resemble something like the following: http://34.135.245.19:8080/blue.
 
# Task 5. Deploy the second versions of the application
Build the second development deployment
 
1. Switch back to the dev branch.
 
git checkout dev
 
2. In the main.go file, update the main() function to the following:
 
func main() {
http.HandleFunc("/blue", blueHandler)
http.HandleFunc("/red", redHandler)
http.ListenAndServe(":8080", nil)
}
 
# 3. Add the following function inside of the main.go file:
func redHandler(w http.ResponseWriter, r *http.Request) {
img := image.NewRGBA(image.Rect(0, 0, 100, 100))
draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{255, 0, 0, 255}}, image.ZP, draw.Src)
w.Header().Set("Content-Type", "image/png")
png.Encode(w, img)
}
 
4. Inspect the cloudbuild-dev.yaml file to see the steps in the build process. Update the version of the Docker image to v2.0.
 
5. Navigate to the dev/deployment.yaml file and update the container image name to the new version (v2.0).
 
6. Make a commit with your changes on the dev branch and push changes to trigger the sample-app-dev-deploy build job.
 
git add .
git commit -m "add red feature to app"
git push -u origin dev
 
7. Verify your build executed successfully, and verify the development-deployment application was deployed onto the dev namespace of the cluster and is using the v2.0 image.
 
8. Navigate to the Load Balancer IP of the service and add the /red entry point at the end of the URL to verify the application is up and running. It should resemble something like the following: http://34.135.245.19:8080/red.
 
Build the second production deployment
 
1. Switch back to the dev branch.
 
git checkout master
 
2. In the main.go file, update the main() function to the following:
 
func main() {
http.HandleFunc("/blue", blueHandler)
http.HandleFunc("/red", redHandler)
http.ListenAndServe(":8080", nil)
}
 
# 3. Add the following function inside of the main.go file:
 
func redHandler(w http.ResponseWriter, r *http.Request) {
img := image.NewRGBA(image.Rect(0, 0, 100, 100))
draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{255, 0, 0, 255}}, image.ZP, draw.Src)
w.Header().Set("Content-Type", "image/png")
png.Encode(w, img)
}
  
4. Inspect the cloudbuild.yaml file to see the steps in the build process. Update the version of the Docker image to v2.0.
 
5. Navigate to the prod/deployment.yaml file and update the container image name to the new version (v2.0).
 
6. Make a commit with your changes on the master branch and push changes to trigger the sample-app-prod-deploy build job.
 
git commit -a -m "add red feature to app"
 
git push -u origin master
 
 
7. Verify your build executed successfully, and verify the production-deployment application was deployed onto the prod namespace of the cluster and is using the v2.0 image.
 
8. Navigate to the Load Balancer IP of the service and add the /red entry point at the end of the URL to verify the application is up and running. It should resemble something like the following: http://34.135.245.19:8080/red.
# Task 6. Roll back the production deployment
 
1. Roll back the production-deployment to use the v1.0 version of the application.
In the Cloud console, go to Cloud Build > Dashboard.
Click on View all link under Build History for the sample-app-prod-deploy trigger.
Click on the second most recent build available (or the v1.0 build that executed successfully)
Click Rebuild.
Other method to roll back using kubectl tool:
Run the following command to roll back the production deployment
kubectl -n prod rollout undo deployment/production-deployment
Run the following command to check version status
kubectl -n prod get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'







