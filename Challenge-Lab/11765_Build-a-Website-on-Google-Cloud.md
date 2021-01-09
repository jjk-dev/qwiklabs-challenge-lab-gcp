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

You will have to run Cloud Build (in that monolith folder) to build it, then push it up to GCR.

Name your artifact as follows:
```
GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
Image name: fancytest
Image version: 1.0.0
```

Make sure that you submit a build named **fancytest** with a version of **1.0.0**.

```

```

### Task 2: Create a kubernetes cluster and deploy the application

```

```

### Task 3: Create a containerized version of your Microservices
```

```

### Task 4: Deploy the new microservices
```

```

### Task 5: Configure the Frontend microservice
```

```

### Task 6: Create a containerized version of the Frontend microservice
```

```

### Task 7: Deploy the Frontend microservice
```

```

## Congratulations!
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.png) ![Badge_cl Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud-cl.png)

### Finish your Quest


