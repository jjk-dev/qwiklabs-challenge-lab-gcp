# Build a Website on Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/11765?parent=catalog)

## Challenge scenario
Your task is to take the company's existing monolithic e-commerce website and break it into a series of logically separated microservices. The existing monolith code is sitting in a GitHub repo, and you will be expected to containerize this app and then refactor it.

You have been asked to take the lead on this, after the last team suffered from monolith-related burnout and left for greener pastures (literally, they are running a lavender farm now). You will be tasked with pulling down the source code, building a container from it (one of the farmers left you a Dockerfile), and then pushing it out to GKE.

You receive the following request to complete the following tasks:
- Building and refactoring a monolithic web app into microservices
- Deploying microservices in GKE
- Exposing the Services deployed on GKE

Some standards you should follow:
- Create your cluster in **us-central-1.**
- Naming is normally team-resource, e.g. an instance could be named **fancystore-orderservice1.**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination.
- Use the **n1-standard-1** machine type unless directed otherwise.

### Task 1: Download the monolith code and build your container
You'll need to clone your team's git repo. There's a **setup.sh** script in the root directory of the project that you'll need to run to get your monolith container built up.

After running the **setup.sh** script, there will be a few different projects that can be built and pushed.

Push the monolith build up to the Google Container Registry. There's a Dockerfile located in the **~/monotlith-to-microservices/monolith** folder which you can use to build the application container.
```
git clone https://github.com/googlecodelabs/monolith-to-microservices.git

cd ~/monotlith-to-microservices
./setup.sh

cd ~/monotlith-to-microservices/monolith
npm start
```

You will have to run Cloud Build (in that monolith folder) to build it, then push it up to GCR. Name your artifact as follows:
```
GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
Image name: fancytest
Image version: 1.0.0
```

Make sure that you submit a build named **fancytest** with a version of **1.0.0**.
```
cd ~/monolith-to-microservices/monolith
gcloud services enable cloudbuild.googleapis.com
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancytest:1.0.0 .
```

### Task 2: Create a kubernetes cluster and deploy the application
Now that you have the image created and sitting in the container registry, it's time to create a cluster to deploy it to.

You've been told to deploy all of your resources in the **us-central1-a zone**, so first you'll need to create a GKE cluster for it. Start with a 3 node cluster to begin with.

Create your cluster as follows:
```
Cluster name: fancy-cluster
Region: us-central1-a
Node count: 3
```

Make sure your cluster is named **"fancy-cluster"**, and is in the running state in **us-central1-a**.
```
gcloud config set compute/zone us-central1-a
gcloud services enable container.googleapis.com
gcloud container clusters create fancy-cluster --num-nodes 3
gcloud compute instances list
```

Now that you've built up an image, and have a cluster up and running, it's time to deploy your application.

You'll need to deploy the image that you've built onto your cluster. This will get your application up and running, but it can't be accessed until you expose it to the outside world. Your team has told you that the application runs on port 8080, but you will need to expose this on a more consumer-friendly port 80.

Create and expose your deployment as follows:
```
Cluster name: fancy-cluster
Container name: fancytest
Container version: 1.0.0
Application port: 8080
Externally accessible port: 80
```

Make note of the IP address that is assigned in the expose deployment operation. You should now be able to visit this IP address from your browser!

Make sure your deployment is named **"fancytest"**, and that you have exposed the service on **port 80**, and mapped it to **port 8080**.
```
kubectl create deployment fancytest --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancytest:1.0.0
kubectl expose deployment fancytest --type=LoadBalancer --port 80 --target-port 8080
```

### Task 3: Create a containerized version of your Microservices
There are 3 services that need to be broken out into their own containers. Since you are moving all of the services into containers, you need to track the following information for each service:
- The root folder of the service (where you will build the container)
- The repository you will upload the container to
- The name & version of the container artifact

Below is the set of services which need to be containerized. Navigate to the source roots mentioned below, and upload the artifacts which are created to the Google Container Registry with the metadata indicated.

|Field|Value|
|---|---|
|**Orders Microservice**|Service root folder: `~/monolith-to-microservices/microservices/src/orders`, GCR Repo: `gcr.io/${GOOGLE_CLOUD_PROJECT}`, Image name: orders, Image version: 1.0.0|
|**Products Microservice**|Service root folder: `~/monolith-to-microservices/microservices/src/products`, GCR Repo: `gcr.io/${GOOGLE_CLOUD_PROJECT}`, Image name: products, Image version: 1.0.0|

Once these microservices have been containerized, and their images uploaded to the GCR, you should deploy and expose these services.

Make sure that you submit a build named **"orders"** with a **version of "1.0.0"**, AND a build named **"products"** with a **version of "1.0.0"**.
```
cd ~/monolith-to-microservices/microservices/src/orders
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0 .

cd ~/monolith-to-microservices/microservices/src/products
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0 .
```

### Task 4: Deploy the new microservices
Deploy these new containers following the same process that you followed for the "fancytest" monolith. Note that these services will be listening on different ports, so make note of the port mappings in the table below.

Create and expose your deployments as follows:

|Field|Value|
|---|---|
|**Orders Microservice**|Cluster name: fancy-cluster, Container name: orders, Container version: 1.0.0, Application port: 8081, Externally accessible port: 80|
|**Products Microservice**|Cluster name: fancy-cluster, Container name: products, Container version: 1.0.0, Application port: 8082, Externally accessible port: 80|

Make sure your deployments are named "orders" and "products", and that you see the services exposed on port 80.
```
kubectl create deployment orders --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0
kubectl expose deployment orders --type=LoadBalancer --port 80 --target-port 8081

kubectl create deployment products --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0
kubectl expose deployment products --type=LoadBalancer --port 80 --target-port 8082

kubectl get services
```

You can verify that the deployments were successful and that the services have been exposed by going to the following URLs in your browser:
- `http://ORDERS_EXTERNAL_IP/api/orders`
- `http://PRODUCTS_EXTERNAL_IP/api/products`

You will see each service return a JSON string if the deployments were successful. Make sure your deployments are named **"orders"** and **"products"**, and that you see the services exposed on **port 80**.

### Task 5: Configure the Frontend microservice
Now that you have extracted both the Orders and Products microservice, you need to configure the Frontend service to point to them, and get it deployed.
 
Use the **nano** editor to replace the local URL with the IP address of the new Products microservices:
```
cd ~/monolith-to-microservices/react-app
nano .env
```

Replace the **REACT_APP_PRODUCTS_URL** to the new format while replacing with your Orders and Product microservice IP addresses so it matches below:
```
REACT_APP_ORDERS_URL=http://ORDERS_IP_ADDRESS/api/orders
REACT_APP_PRODUCTS_URL=http://PRODUCTS_IP_ADDRESS/api/products
```

Press `CTRL+O`, press `ENTER`, then `CTRL+X` to save the file in the nano editor.

Now rebuild the frontend app before containerizing it:
```
npm run build
```

### Task 6: Create a containerized version of the Frontend microservice
With the Orders and Products microservices now containerized and deployed, and the Frontend service configured to point to them, the final step is to containerize and deploy the Frontend.

Use Cloud Build to package up the contents of the Frontend service and push it up to the Google Container Registry.
```
Service root folder:  ~/monolith-to-microservices/microservices/src/frontend
GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
Image name: frontend
Image version: 1.0.0
```

This process may take a few minutes, so be patient. Make sure that you submit a build named **"frontend"** with a **version of "1.0.0"**.
```
cd ~/monolith-to-microservices/microservices/src/frontend
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0 .
```

### Task 7: Deploy the Frontend microservice
Deploy this container following the same process that you followed for the **"Orders"** and **"Products"** microservices.

Create and expose your deployment as follows:
```
Cluster name: fancy-cluster
Container name: frontend
Container version: 1.0.0
Application port: 8080
Externally accessible port: 80
```

You can verify that the deployment was successful and that the microservices have been properly exposed by hitting the following the IP address of the frontend service in your browser: You will see the Fancy Store homepage, with links to the Products and Orders pages powered by your new microservices.
```
kubectl create deployment frontend --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0
kubectl expose deployment frontend --type=LoadBalancer --port 80 --target-port 8080
```

## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Build-a-Website-on-Google-Cloud.png" height="150" />
