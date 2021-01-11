# Deploy and Manage Cloud Environments with Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/10417?parent=catalog)

## Challenge scenario
You have started a new role as a Cloud Architect for Jooli Inc. You are expected to help design and manage the infrastructure at Jooli. Common tasks revolve around designing environments for the various projects within the Jooli Inc. family but also include provisioning resources for projects.

You have been asked to assist the kraken team complete setting up their product development environment. The previous Cloud Architect working with the kraken team was unfortunately too curious about if krakens were real or not, and has gone missing after venturing out into the open sea last weekend in search of such a beast.

The kraken team are building a next generation tool and they will host the application on Kubernetes. The project source code is stored in Cloud Source Repositories, with Spinnaker building and deploying any changes into the build Kubernetes environment.
- Create the Production Environment
- Setup the Admin instance
- Verify the Spinnaker deployment

Some standards you should follow:
- Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications such as Kubernetes nodes.

### Task 1: Create the Production Environment
The previous Cloud Architect had written the **Deployment Manager** configuration to build the network for kraken's production environment. You can find the DM configuration on your **jumphost** in `/work/dm`. Create the network using the Deployment Manager configuration (`/work/dm/prod-network.yaml and /work/dm/prod-network.jinja`). Make sure you review the configuration before deploying it.

You will also need to create the Kubernetes environment. The application is already created and in the Container Repository.
- Create a two (2) node cluster called **kraken-prod** in the **kraken-prod-vpc** (remember to use **--num-nodes** to create 2 nodes only).
- Use **kubectl** with the files in **/work/k8s** to create the frontend and backend deployments and services (which will expose the frontend service via a load balancer).

Go to **Compute Engine > VM Instnaces.** And login to **kraken-jumphost** via **SSH**.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Deploy-and-Manage-Cloud-Environments-with-Google-Cloud-1.png)

Run below code on SSH.

```
cd /work/dm
sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud config set compute/zone us-east1-b

gcloud deployment-manager deployments create pod-network \
  --config=prod-network.yaml

gcloud container clusters create kraken-prod \
  --num-nodes 2 \
  --network kraken-prod-vpc \
  --subnetwork kraken-prod-subnet

gcloud container get-credentials kraken-prod

cd /work/k8s
kubectl create -f deployment-prod-frontend.yaml
kubectl create -f deployment-prod-backend.yaml
kubectl get pods

kubectl create -f service-prod-frontend.yaml
kubectl create -f service-prod-backend.yaml
kubectl get services
```

### Task 2: Setup the Admin instance
You need to set up an admin machine for the team to use. Once you create the **kraken-prod-vpc**, you will need to add an instance called `kraken-admin`, a network interface in **kraken-mgmt-subnet** and another in **kraken-prod-subnet**.

```
gcloud compute instances create kraken-admin \
  --network-interface="subnet=kraken-mgmt-subnet" \
  --network-interface="subnet=kraken-prod-subnet"
```

You need to monitor **kraken-admin** and if **CPU utilization** is over **50%** for more than a minute you need to send an email to yourself, as admin of the system.
1. In the left navigation, **Monitoring** > **Alerting**
2. CREATE POLICY > ADD CONDITION
3. Untitled Condition: VM Instance
4. In Target section, click **VM instance** on Resource type > **CPU utilization** on Metric
5. On Filter field, click **instance_name** > **kraken-admin** to Value > Click Apply
6. In Configuration section, put **50** on Threshold section > Click ADD
7. NEXT > Click Notification Channels > MANAGE NOTIFICAITON CHANNELS
8. On Email, click ADD NEW > Email Address: Username, Display Name: student > SAVE > Back to previous page
9. Refresh > Click Notification Channels > Check student > OK > NEXT
10. Alert name: kraken-admin > SAVE

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Deploy-and-Manage-Cloud-Environments-with-Google-Cloud-2.png)

### Task 3: Verify the Spinnaker deployment
The previous architect set up Spinnaker in **kraken-build-vpc**. Please connect to the Spinnaker console and verify that the built resources are working.

To access the Spinnaker console use Cloud Shell and **kubectl** to **port forward** the **spin-deck** pod from port **9000 to 8080** and then use Cloud Shell's **web preview.**

You must test that a change to the source code will result in the automated deployment of the new build. You should pull the **sample-app** repository to make the changes. Make sure you push a **new, updated, tag.**

Use CloudShell to run below.

```
gcloud config set compute/zone us-east1-b
gcloud container clusters get-credentials spinnaker-tutorial

export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
  -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &

gcloud source repos clone sample-app
cd sample-app
git config --global user.email "$(gcloud config get-value core/account)"
git config --global user.name "student"

git commit -a -m "change"
git tag v1.0.1
git push --tags
```

Click **Web Preview** at the top of the CloudShell and select **Preview on port 8080**.
Then on the top navigation, click **Application > sample > PIPELINES > Continue** on pipleline with human icon.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Deploy-and-Manage-Cloud-Environments-with-Google-Cloud-3.png)

## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Deploy-and-Manage-Cloud-Environments-with-Google-Cloud.png" height="150" /> <img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Deploy-and-Manage-Cloud-Environments-with-Google-Cloud-cl.png" height="150" /> 

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Cloud Architecture](https://google.qwiklabs.com/quests/24) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in a Quest and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests](https://google.qwiklabs.com/catalog).
