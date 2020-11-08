# Set up and Configure a Cloud Environment in Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/10603?parent=catalog)

## Challenge scenario
As a cloud engineer in Jooli Inc. and recently trained with Google Cloud and Kubernetes you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

You need to complete the following tasks:
- Create a development VPC with three subnets manually
- Create a production VPC with three subnets using a provided Deployment Manager configuration
- Create a bastion that is connected to both VPCs
- Create a development Cloud SQL Instance and connect and prepare the WordPress environment
- Create a Kubernetes cluster in the development VPC for WordPress
- Prepare the Kubernetes cluster for the WordPress environment
- Create a WordPress deployment using the supplied configuration
- Enable monitoring of the cluster via stackdriver
- Provide access for an additional engineer

Some standards you should follow:
- Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications such as Kubernetes nodes.


### Task 1: Create development VPC manually
Create a VPC called **griffin-dev-vpc** with the following subnets only:
- griffin-dev-wp
  - IP address block: 192.168.16.0/20
- griffin-dev-mgmt
  - IP address block: 192.168.32.0/20
```

```

### Task 2: Create production VPC using Deployment Manager
Use Cloud Shell and copy all files from **gs://cloud-training/gsp321/dm.**

Check the Deployment Manager configuration and make any adjustments you need, then use the template to create the production VPC with the 2 subnets. 

Please remove '[ ]'.
```

```

### Task 3: Create bastion host
Create a bastion host with two network interfaces, one connected to **griffin-dev-mgmt** and the other connected to **griffin-prod-mgmt.** Make sure you can SSH to the host.

```

```

### Task 4: Create and configure Cloud SQL Instance
Create a **MySQL Cloud SQL Instance** called **griffin-dev-db** in us-east1. Connect to the instance and run the following SQL commands to prepare the **WordPress** environment:

```
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```

These SQL statements create the worpdress database and create a user with access to the wordpress database.

### Task 5: Create Kubernetes cluster
Create a 2 node cluster (n1-standard-4) called **griffin-dev**, in the **griffin-dev-wp** subnet, and in zone **us-east1-b.**
```

```

### Task 6: Prepare the Kubernetes cluster
Use Cloud Shell and copy all files from **gs://cloud-training/gsp321/wp-k8s.**

The **WordPress** server needs to access the MySQL database using the **username and password** you created in task 4. You do this by setting the values as secrets. **WordPress** also needs to store its working files outside the container, so you need to create a volume.

Add the following secrets and volume to the cluster using **wp-env.yaml.** Make sure you configure the username to **wp_user** and password to **stormwind_rules** before creating the configuration.

You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container. Use the command below to create the key, and then add the key to the Kubernetes environment.
```
gcloud iam service-accounts keys create key.json \
--iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```
```
kubectl create secret generic cloudsql-instance-credentials \
--from-file key.json
```

### Task 7: Create a WordPress deployment
Now you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using **wp-deployment.yaml.** Before you create the deployment you need to edit **wp-deployment.yaml** and replace **YOUR_SQL_INSTANCE** with griffin-dev-db's **Instance connection name.** Get the **Instance connection name** from your Cloud SQL instance.

After you create your WordPress deployment, create the service with **wp-service.yaml.**

Once the Load Balancer is created, you can visit the site and ensure you see the **WordPress** site installer. At this point the dev team will take over and complete the install and you move on to the next task.```

```
```

### Task 8: Enable monitoring
Create an uptime check for your WordPress development site.

```
```

### Task 9: Provide access for an additional engineer
You have an additional engineer starting and you want to ensure they have access to the project, so please go ahead and grant them the editor role to the project.

The second user account for the lab represents the additional engineer.

```
```


## Congratulations!
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.png) ![Badge_cl Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud-cl.png)

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Cloud Engineering](https://google.qwiklabs.com/quests/66) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. Enroll in this Quest and get immediate completion credit if you've taken this lab. 
 [See other available Qwiklabs Quests.](https://google.qwiklabs.com/catalog)

