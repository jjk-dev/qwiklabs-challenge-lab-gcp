# Deploy to Kubernetes in Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/10457?parent=catalog)

## Challenge scenario
You are expected to create container images, store the images in a repository, and configure a Jenkins CI/CD pipeline to automate the build for the product. Your know that Kurt, your supervisor, will ask you to complete these tasks:

- Create a Docker image and store the Dockerfile.
- Test the created Docker image.
- Push the Docker image into the Container Repository.
- Use the image to create and expose a deployment in Kubernetes
- Update the image and push a change to the deployment.
- Create a pipeline in Jenkins to deploy a new version of your image when the source code changes.

Some standards you should follow:
- Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named **kraken-webserver1**.

Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use **n1-standard-1**.

### Task 1: Create a Docker image and store the Dockerfile
Open Cloud Shell and run below code. This command will install marking scripts you can use to help check your progress.
```
source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking.sh)
```

Use Cloud Shell to clone the **valkyrie-app** source code repository. Please replace **YOUR_PROJECT_ID** with your Project ID.
```
export PROJECT=YOUR_PROJECT_ID
gcloud source repos clone valkyrie-app --project=$PROJECT
```

The app source code is in **valkyrie-app/source**. Create **valkyrie-app/Dockerfile** and add the configuration below.
```
cd valkyrie-app
cat > Dockerfile << EOF
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
EOF
```

Use **valkyrie-app/Dockerfile** to create a Docker image called **valkyrie-app** with the tag **v0.0.1**
```
docker build -t valkyrie-app:v0.0.1 .
```

Once you have created the Docker image, and before clicking Check my progress, run step1.sh to perform the local check of your work.
```
cd ~/marking
./step1.sh
```

### Task 2: Test the created Docker image
Launch a container using the image **valkyrie-app:v0.0.1**. You need to map the host’s port 8080 to port 8080 on the container. Add & to the end of the command to cause the container to run in the background.
```
docker run -p 8080:8080 --name valkyrie-app valkyrie-app:v0.0.1 &
```

When your container is running you will see the page by **Web Preview**.
![screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Deploy-to-Kubernetes-in-Google-Cloud.png-1.png)

Once you have your container running, and before clicking Check my progress, run step2.sh to perform the local check of your work.
```
cd ~/marking
./step2.sh
```

### Task 3: Push the Docker image in the Container Repository
Push the Docker image **valkyrie-app:v0.0.1** into the Container Registry. Make sure you re-tag the container to **gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.1**.
```
docker tag valkyrie-app:v0.0.1 gcr.io/$PROJECT/valkyrie-app:v0.0.1
docker images
docker push gcr.io/$PROJECT/valkyrie-app:v0.0.1
```

### Task 4: Create and expose a deployment in Kubernetes
Kurt created the **deployment.yaml** and **service.yaml** to deploy your new container image to a Kubernetes cluster (called valkyrie-dev). The two files are in **valkyrie-app/k8s**. Remember you need to get the Kubernetes credentials before you deploy the image onto the Kubernetes cluster.
```
cd k8s
gcloud container clusters get-credentials valkyrie-app --region us-east1
```

With below code, now replace **IMAGE_HERE** with `gcr.io/$PROJECT/valkyrie-app:v0.0.1`.
```
sed -i s#IMAGE_HERE#gcr.io/$PROJECT/valkyrie-app:v0.0.1#g k8s/deployment.yaml
```

Click **ESC button** and enter `:wq!` . Before you create the deployments make sure you check the **deployment.yaml** and **service.yaml** files.
```
kubectl create -f deployment.yaml
kubectl create -f service.yaml
```

### Task 5: Update the deployment with a new version of valkyrie-app
Before deploying the new code, increase the **replicas from 1 to 3** to ensure you don't cause an outage. Kurt made changes to the source code (he put the changes in a branch called **kurt-dev**). You need to merge **kurt-dev into master** (you should use git merge origin/kurt-dev).
```
cd ..
git merge origin/kurt-dev
kubectl edit deployment valkyrie-dev
```

Build the new code as version v0.0.2 of valkyrie-app, push the updated image to the Container Repository, and then redeploy to the valkyrie-dev cluster. You will know you have the new v0.0.2 version because the titles for the cards will be green.
```
docker build -t gcr.io/$PROJECT/valkyrie-app:v0.0.2 .
docker images
docker push gcr.io/$PROJECT/valkyrie-app:v0.0.2
```

Make your to change image version from **v0.0.1 to v0.0.2**. Enter `:42 + enter` and `i` then click **ESC button** and enter `:wq!`.
```
kubectl edit deployment valkyrie-dev
```

### Task 6: Create a pipeline in Jenkins to deploy your app
This process of building the container and pushing to the container repository can be automated using Jenkins. There is a Jenkins deployment in your **valkyrie-dev** cluster - connect to Jenkins and configure a job to build when you push a change to the source code.

Get the password with the below command.
```
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

Connect to the Jenkins console using the commands below. Make sure you don't have a running container `docker ps` and change **CONTAINER_ID** to your container id.
```
docker ps
docker kill CONTAINER_ID
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```

Click on the Web Preview button in cloud shell, then click “Preview on port 8080” to connect to the Jenkins console. **Username: admin** and **Password: result from admin-password**.

Setup your credentials to use **Google Service Account from metadata.**
1. On left navigation, Manage Jenkins > Manage Credentials
2. Jenkins witch is in Stores section > Global Credentials (unrestricted)
2. On left navigation, Add Credentials > In Kind section, Google Service Account from metadata > Click OK

Create a pipeline job that points to your `*/master` branch on your source code.
1. Jenkins > On left navigation, New Item
2. Projecg name: valkyrie-app
3. Multibranch Pipeline > Click OK > In the Branch Sources section, Add Source > Git
4. Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories. And replace **YOUR_PROJECT_ID** with your Project ID. `https://source.developers.google.com/p/YOUR_PROJECT_ID/r/valkyrie-app`
5. Credentials > Select your credentials
6. Under Scan Multibranch Pipeline Triggers section > Check Periodically if not otherwise run > Set Interval value to 1 minute > Click Save

Make two changes to your files before you commit and build:
- Edit `valkyrie-app/source/html.go` and change the two occurrences of green to orange.
- Edit `valkyrie-app/Jenkinsfile` and change **YOUR_PROJECT** to your actual project id. 
```
sed -i "s/green/oragne/g" source/html.go
sed -i "s/YOUR_PROJECT/$PROJECT/g" Jenkinsfile
```

Use git to:
- Add all the changes then commit those changes to the master branch.
- Push the changes back to the repository.
```
git config --global user.email "you@email.com"
git config --global user.name "student"
git add .
git push origin master
```

When you are ready, manually trigger a build (the initial build will take some time, so just monitor the process). The build will replace the running containers with containers with different tags; you will see orange colored headings.


## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Deploy-to-Kubernetes-in-Google-Cloud.png" height="150" />

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Kubernetes in Google Cloud](https://google.qwiklabs.com/quests/29) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. Enroll in this Quest and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests](https://google.qwiklabs.com/catalog).

This skill badge quest is part of Google Cloud’s [Hybrid and Multi-Cloud Cloud Architect](https://cloud.google.com/training/kubernetes-anthos-hybridcloud) learning path. Continue your learning journey by enrolling in the [Secure Workloads in Google Kubernetes Engine](https://google.qwiklabs.com/quests/142) quest.
