# Serverless Firebase Development: [Challenge Lab](https://www.qwiklabs.com/focuses/14677?parent=catalog)

## Challenge scenario
In this lab you will create a frontend solution using a Rest API and Firestore database. Cloud Firestore is a NoSQL document database that is part of the Firebase platform where you can store, sync, and query data for your mobile and web apps at scale. Lab content is based on resolving a real world scenario through the use of Google Cloud serverless infrastructure.

![Screenshot](https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/img/Serverless-Firebase-Development-1.png)

### Task 1: Create a Firestore database
Create a Firestore Database in Google Cloud. The high level architecture diagram below summarizes the general architecture.

|**Field**|**Value**|
|Cloud Firestore|Native Mode|
|Location|Nam5 (United States)|

Provision the environment:
```
gcloud config set run/region us-central
gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')
git clone https://github.com/rosera/pet-theory.git
```
Go to Firestore in the left navigation. Select **Naive Mode**.

![Screenshot](https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/img/Serverless-Firebase-Development-2.png)

Set **Location: nam5** then click **CREATE DATABASE** button.

![Screenshot](https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/img/Serverless-Firebase-Development-3.png)


### Task 2: Populate the Database
Populate the database using test data.

Use the sample code from `pet-theory/lab06/firebase-import-csv/solution`. And to import CSV, use the node `index.js`.
```
cd ~/pet-theory/lab06/firebase-import-csv/solution
npm install
node index.js netflix_titles_original.csv
```

### Task 3: Create a REST API
Create an example REST API.
- Access `pet-theory/lab06/firebase-rest-api/solution-01`
- Build and Deploy the code to Google Container Registry
- Deploy the image as a Cloud Run Service
- `curl -X GET $SERVICE_URL` should respond with:
```{"status":"Netflix Dataset! Make a query."}```

|**Field**|**Value**|
|Container Registry Image|rest-api:0.1|
|Cloud Run Service|netflix-dataset-service|
|Permission|--allow-unauthenticated|

```
cd ~/pet-theory/lab06/firebase-rest-api/solution-01
gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1
gcloud run deploy netflix-dataset-service --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 --allow-unauthenticated
```

When deploying is completed, copy netflix-dataset-service's service url.
```
SERVICE_URL=[CHANGE TO YOUR SERVICE URL]
curl -X GET $SERVICE_URL
```

### Task 4: Firestore API access
Deploy an updated revision of the code to access the Firestore DB.
- Access `pet-theory/lab06/firebase-rest-api/solution-02`
- Build the updated application
- Use Cloud Build to tag and deploy image revision to Container Registry
- Deploy the new image as Cloud Run service
- `curl -X GET $SERVICE_URL/2019` should respond with json dataset

|**Field**|**Value**|
|Container Registry Image|rest-api:0.2|
|Cloud Run Service|netflix-dataset-service|
|Permission|--allow-unauthenticated|

```
cd ~/pet-theory/lab06/firebase-rest-api/solution-02
gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2
gcloud run deploy netflix-dataset-service --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 --allow-unauthenticated
curl -X GET $SERVICE_URL/2019
```

### Task 5: Deploy the Staging Frontend
Deploy the Staging Frontend.
- Access `pet-theory/lab06/firebase-frontend`
- Build the frontend staging application
- Use Cloud Build to tag and deploy image revision to Container Registry
- Deploy the new image as Cloud Run service
- Frontend access to Rest API and Firestore Database

|**Field**|**Value**|
|REST_API_SERVICE|REST API SERVICE URL|
|Container Registry Image|frontend-staging:0.1|
|Cloud Run Service|frontend-staging-service|

```
cd ~/pet-theory/lab06/firebase-frontend
gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1
gcloud run deploy frontend-staging-service --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 --allow-unauthenticated
```

Access the Frontend Service URL.

![Screenshot](https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/img/Serverless-Firebase-Development-4.png)

### Task 6: Deploy the Production Frontend
Update the Staging Frontend to use the Firestore database.
- Access `pet-theory/lab06/firebase-frontend/public`
- Update the frontend application i.e. `app.js` to use the REST API
- Don't forget to append the year to the SERVICE_URL
- Use Cloud Build to tag and deploy image revision to Container Registry
- Deploy the new image as Cloud Run service
- Frontend access to Rest API and Firestore Database

```
cd ~/pet-theory/lab06/firebase-frontend/public
nano app.js
```

In line 4 **const REST_API_SERVICE**, erase **//** and modify to your netflix-dataset-service url like below. Save the following code with `Ctrl + x` > `Y` > `Enter`.
```
const REST_API_SERVICE = "https://netflix-dataset-service-js3gnnsjfq-uc.a.run.app/2020" 
```

Then run build and deploy as a production environment in Cloud shell.
```
cd ~/pet-theory/lab06/firebase-frontend
gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1
gcloud run deploy frontend-production-service --image=gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1 --allow-unauthenticated
```

## Congratulations!
<img src="https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/img/Serverless-Firebase-Development.png" height="150" />

### Finish your Quest
This self-paced lab is part of the Serverless Firebase Development skill badge quest. Completing this skill badge quest earns you the badge above, to recognize your achievement. Share your badge on your resume and social platforms, and announce your accomplishment using #GoogleCloudBadge.

This skill badge is part of Google Cloud’s Cloud Developer learning path. Continue your learning journey by searching the catalog for 20+ other skill badge quests in which you can enroll.
