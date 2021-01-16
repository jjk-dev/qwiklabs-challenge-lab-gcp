# Perform Foundational Data, ML, and AI Tasks in Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/11044?parent=catalog)

## Challenge scenario
As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.

- Create a simple Dataproc job
- Create a simple DataFlow job
- Create a simple Dataprep job
- Perform one of three Google machine learning backed API tasks


### Task 1: Run a simple Dataflow job
You have used Dataflow in the question to load data into BigQuery from Pub/Sub, now use the Dataflow batch template **Text Files on Cloud Storage to BigQuery** under "Process Data in Bulk (batch)" to transfer data from a Cloud Storage bucket (`gs://cloud-training/gsp323/lab.csv`). The following table has the values you need to correctly configure the Dataflow job.

In the Cloud Shell, create BigQuery dataset called **lab** and download the schema file.
```
bq mk lab

gsutil cp gs://cloud-training/gsp323/lab.schema .
cat lab.schema
```

Copy below schema witch is the result from `cat lab.schema`.
```
[
  {"type":"STRING","name":"guid"},
  {"type":"BOOLEAN","name":"isActive"},
  {"type":"STRING","name":"firstname"},
  {"type":"STRING","name":"surname"},
  {"type":"STRING","name":"company"},
  {"type":"STRING","name":"email"},
  {"type":"STRING","name":"phone"},
  {"type":"STRING","name":"address"},
  {"type":"STRING","name":"about"},
  {"type":"TIMESTAMP","name":"registered"},
  {"type":"FLOAT","name":"latitude"},
  {"type":"FLOAT","name":"longitude"}
]
```

Go to **Navigation Menu > BigQuery**. Select **Project ID > lab** then click **CREATE TABLE**.
1. In Source field, select **Google Cloud Storege** > put `gs://cloud-training/gsp323/lab.csv` on GCS bucket.
2. Table name: `customers`
3. Schema: Copy from `lab.schema`
4. Click **Create Table**.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud-1.png)

Create a Cloud Storage Bucket called **YOUR_PROJECT** and change it to your **Project ID**.
```
gsutil mb gs://YOUR_PROJECT/
```

Go to **Navigation Menu > Dataflow**.
1. Jobs > CREATE JOB FROM TEMPLATE
2. Job name: `lab-flow`
3. In Dataflow template section, select **Text Files on Cloud Storage to BigQuery** under **Process Data in Bulk (batch)**.
4. Enter the values on the parameters with the below table. Must replace **YOUR_PROJECT** with the Project ID.

|Field|Value|
|---|---|
|JavaScript UDF path in Cloud Storage|`gs://cloud-training/gsp323/lab.js`|
|JSON path|`gs://cloud-training/gsp323/lab.schema`|
|JavaScript UDF name|`transform`|
|BigQuery output table|`YOUR_PROJECT:lab.customers`|
|Cloud Storage input path|`gs://cloud-training/gsp323/lab.csv`|
|Temporary BigQuery directory|`gs://YOUR_PROJECT/bigquery_temp`|
|Temporary location|`gs://YOUR_PROJECT/temp`|

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud-2.png)

5. Click **RUN JOB**.

### Task 2: Run a simple Dataproc job
You have used Dataproc in the quest, now you must run another example Spark job using Dataproc.

Create Dataproc cluster named `lab-cluster`.
```
gcloud config set dataproc/region us-central1
gcloud dataproc clusters create lab-cluster
```

Before you run the job, log into one of the cluster nodes and copy the /data.txt file into hdfs.
1. Navigation Menu > Dataproc
2. Click the name of cluster > VM INSTNACES tab
3. Click **SSH** and run `hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt` in the SSH window.

To run a Dataproc job using the values below, run the following command in Cloud Shell.

|Field|Value|
|---|---|
|Region|`us-central1`|
|Job type|`Spark`|
|Main class or jar|`org.apache.spark.examples.SparkPageRank`|
|Jar files|`file:///usr/lib/spark/examples/jars/spark-examples.jar`|
|Arguments|`/data.txt`|

```
gcloud dataproc jobs submit spark --cluster lab-cluster \
  --class org.apache.spark.examples.SparkPageRank \
  --jars file:///usr/lib/spark/examples/jars/spark-examples.jar \
  -- /data.txt
```

### Task 3: Run a simple Dataprep job
You have used Dataprep to import data files and transformed them to gain views of the data. Use Dataprep to import one CSV file (described below) that holds data of lab executions.
1. Navigation Menu > Dataprep
2. Accpet all agreements.
3. Enter the Cloud Dataprep home page > In the left panel, select **GCS** > Click **Pencil icon** to edit folder path.
4. Copy `gs://cloud-training/gsp323/runs.csv` in the textbox > Click **GO** > Click **Continue**

Perform the following transforms to ensure the data is in the right state.

Remove all rows with the state of **"FAILURE"**
1. Scroll right to the end and select **column10**.
2. In the Details panel, do Right Click **FAILURE** on **Unique Values section**
3. Select **Delete rows with selected values**

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud-3.png)

Remove all rows with 0 or 0.0 as a score (Use the regex pattern **/(^0$|^0\.0$)/**).
1. Select **column9**.
2. Click three dot at the top > Choose **Filter rows > On column value > Contains**

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud-4.png)

3. On Pattern to match field, put **/(^0$|^0\.0$)/** > Select **Delete matching rows** > Click **Add**

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud-5.png)

Rename the columns with the names:
**runid / userid / labid / lab_title / start / end / time / score / state**

`gs://cloud-training/gsp323/runs.csv` structure:
|runid|userid|labid|lab_title|start|end|time|score|state|
|---|---|---|---|---|---|---|---|---|
|5556|545|122|Lab 122|2020-04-09 16:18:19|2020-04-09 17:10:11|3112|61.25|SUCCESS|
|5557|116|165|Lab 165|2020-04-09 16:44:45|2020-04-09 18:13:58|5353|60.5|SUCCESS|
|5558|969|31|Lab 31|2020-04-09 17:59:01|2020-04-09 18:02:09|188|0|FAILURE|

Click **Run > Run**.

### Task 4: Perform one of three Google machine learning backed API tasks
For the options below **YOUR_PROJECT** must be replaced with your lab project name.

To use AI API, you have to create API Key.
```
export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)

gcloud iam service-accounts create my-lab
  
gcloud iam service-accounts keys create ~/key.json \
  --iam-account my-lab@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com

gcloud auth activate-service-account --key-file key.json
```

You have the option to complete one of the tasks below:

#### Google Cloud Speech API
Use Google Cloud Speech API to analyze the audio file `gs://cloud-training/gsp323/task4.flac`. Once you have analyzed the file you can upload the resulting analysis to `gs://YOUR_PROJECT-marking/task4-gcs.result`.

Create a JSON file called `request-gcs.json`.
```
nano request-gcs.json
```

Save the following code with `Ctrl+x` > `Y` > `Enter`.
```
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-training/gsp323/task4.flac"
  }
}
```

Submit the request to Google Cloud Speech API and store the response to file and Cloud Storage.
```
curl -s -X POST -H "Content-Type: application/json" --data-binary @request-gcs.json \
"https://speech.googleapis.com/v1/speech:recognize?key=$(gcloud auth print-access-token)" > result-gcs.json

cat result-gcs.json

gsutil cp result-gcs.json gs://$GOOGLE_CLOUD_PROJECT-marking/task4-gcs.result
```

#### Cloud Natural Language API
Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text you can upload the resulting analysis to `gs://YOUR_PROJECT-marking/task4-cnl.result`.

```
gcloud ml language analyze-entities --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > result-cnl.json

cat result-cnl.json

gcloud auth login
```

Click the provided link and copy the verification code. Then put following code.
```
gsutil cp result-cnl.json gs://$GOOGLE_CLOUD_PROJECT-marking/task4-cnl.result
```

#### Google Video Intelligence
Use Google Video Intelligence and detect all text on the video `gs://spls/gsp154/video/train.mp4`. Once you have completed the processing of the video, pipe the output into a file and upload to `gs://YOUR_PROJECT-marking/task4-gvi.result`.

Create a JSON file called `request-gvi.json`.
```
nano request-gvi.json
```

Save the following code with `Ctrl+x` > `Y` > `Enter`.
```
{
   "inputUri":"gs://spls/gsp154/video/train.mp4",
   "features": [
       "LABEL_DETECTION"
   ]
}
```

Submit the request to Google Cloud Speech API and store the response to file and Cloud Storage.
```
curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/videos:annotate' \
    -d @requset-gvi.json > result-gvi.json

cat result-gvi.json

gsutil cp result-gvi.json gs://$GOOGLE_CLOUD_PROJECT-marking/task4-gvi.result
```

## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Perform-Foundational-Data-ML-and-AI-Tasks-in-Google-Cloud.png" height="150" />

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Perform Foundational Data, ML, and AI Tasks in Google Cloud](https://google.qwiklabs.com/quests/117) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. Enroll in this Quest and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests](https://google.qwiklabs.com/catalog).

This skill badge quest is part of Google Cloud’s [Data Analyst](https://cloud.google.com/training/data-ml#data-analyst-learning-path) and [Data Engineer](https://cloud.google.com/training/data-ml#data-engineer-learning-path) learning paths. Continue your learning journey by enrolling in the [Engineer Data with Google Cloud](https://google.qwiklabs.com/quests/132) quest.
