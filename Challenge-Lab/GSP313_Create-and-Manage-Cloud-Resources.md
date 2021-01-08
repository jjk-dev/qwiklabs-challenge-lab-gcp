# Getting Started: Create and Manage Cloud Resources: [Challenge Lab](https://www.qwiklabs.com/focuses/10258?parent=catalog)

## Challenge scenario
You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:
- Create an instance
- Create a 3-node Kubernetes cluster and run a simple service
- Create an HTTP(s) load balancer in front of two web servers

Some standards you should follow:
- Create all resources in the default region or zone, unless otherwise directed.
- Naming normally uses the format team-resource; for example, an instance could be named **nucleus-webserver1.**
- Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. This is the guidance the monitoring team is willing to share: unless directed, use **f1-micro** for small Linux VMs, and use **n1-standard-1** for Windows or other applications, such as Kubernetes nodes.

### Task 1: Create a project jumphost instance
You will use this instance to perform maintenance for the project.

Requirements:
- Name the instance **nucleus-jumphost.**
- Use an **f1-micro** machine type.
- Use the default image type (Debian Linux).

Create VM instance.
```
gcloud compute instances create nucleus-jumphost \
  --machine-type f1-micro \
  --zone us-east1-b \
  --machine-family debian-9 \
  --image-project debain-cloud
```

### Task 2: Create a Kubernetes service cluster
The team is building an application that will use a service running on Kubernetes. You need to:
- Create a cluster (in the us-east1-b zone) to host the service.
- Use the Docker container hello-app `(gcr.io/google-samples/hello-app:2.0)` as a place holder; the team will replace the container with their own work later.
- Expose the app on port 8080.

Set a default compute zone and region.
```
gcloud config set compute/zone us-east1-b
gcloud config set compute/region us-east1
```

Create GKE cluster and get authentication credentials for the cluster to interact with it.
```
gcloud container clusters create nucleus-cluster
gcloud container clusters get-credentials nucleus-cluster
```

Deploy an application to the cluser.
```
kubectl create deployment nucleus-server --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment nucleus-server --type=LoadBalancer --port 8080
kubectl get service
```

To view the application from your web browser, open a new tab and enter the following address:
```
http://[EXTERNAL-IP]:8080
```

### Task 3: Set up an HTTP load balancer
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of **2 nginx web servers.** Use the following code to configure the web servers; the team will replace this with their own configuration later.

Run the configuration of the web servers.

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

Create multiple instance template.
```
gcloud compute instance-templates create nginx-template \
  --metadata-from-file startup-script=startup.sh
```

Create a target pool.
```
gcloud compute target-pools create nginx-pool
```

Create a managed instance group.
```
gcloud compute instance-groups managed create nginx-group \
  --base-instance-name nginx \
  --size 2 \
  --template nginx-template \
  --target-pool nginx-pool
  
gcloud compute instances list
```

Create a firewall rule to allow traffic.
```
gcloud compute firewall-rules create www-firewall --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
  --region us-east1 \
  --ports=80 \
  --target-pool nginx-pool
  
gcloud compute forwarding-rules list
```

Create a health check.
```
gcloud compute http-health-checks create basic-check

gcloud compute instance-groups managed \
  set-named-ports nginx-group \
  --named-ports http:80
```

Create a backend service, and attach the managed instance group.
```
gcloud compute backend-services create nginx-backend \
  --protocol HTTP --http-health-checks basic-check \
  --global
  
gcloud compute backend-services add-backend nginx-backend \
  --instance-group nginx-group \
  --instance-group-zone us-east1-b \
  --global

```

Create a URL map, and target the HTTP proxy to route requests to your URL map.
```
gcloud compute url-maps create web-map \
  --default-service nginx-backend
  
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map
```

Create a forwarding rule.
```
gcloud compute forwarding-rules create http-rule \
  --global \
  --target-http-proxy http-lb-proxy \
  --port 80
  
gcloud compute forwarding-rules list
```


## Congratulations!
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.png) ![Badge_cl Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud-cl.png)

### Finish your Quest
This self-paced lab is part of the [Create and Manage Cloud Resources](https://google.qwiklabs.com/quests/120) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in a Quest and get immediate completion credit for taking this lab. [See other available Qwiklabs Quests.](https://google.qwiklabs.com/catalog)

This skill badge quest is part of Google’s [Associate Cloud Engineer](https://cloud.google.com/certification/cloud-engineer) and [Professional Cloud Architect](https://cloud.google.com/certification/cloud-architect) learning paths. Continue your learning journey by enrolling in the [Perform Foundational Infrastructure Tasks in Google Cloud](https://google.qwiklabs.com/quests/118) skill badge quest.

