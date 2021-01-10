# Perform Foundational Data, ML, and AI Tasks in Google Cloud: [Challenge Lab](https://www.qwiklabs.com/focuses/11044?parent=catalog)

## Challenge scenario
As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.

- Create a simple Dataproc job
- Create a simple DataFlow job
- Create a simple Dataprep job
- Perform one of three Google machine learning backed API tasks


### Task 1: Run a simple Dataflow job
You have used Dataflow in the question to load data into BigQuery from Pub/Sub, now use the Dataflow batch template **Text Files on Cloud Storage to BigQuery** under "Process Data in Bulk (batch)" to transfer data from a Cloud Storage bucket (`gs://cloud-training/gsp323/lab.csv`). The following table has the values you need to correctly configure the Dataflow job.

You will need to make sure you have:
- Created a BigQuery dataset called **lab**
- Replace **YOUR_PROJECT** with the lab project name
- Created a Cloud Storage Bucket called **YOUR_PROJECT**

|Field|Value|
|---|---|
|JavaScript UDF path in Cloud Storage|`gs://cloud-training/gsp323/lab.js`|
|JSON path|`gs://cloud-training/gsp323/lab.schema`|
|JavaScript UDF name|`transform`|
|BigQuery output table|`YOUR_PROJECT:lab.customers`|
|Cloud Storage input path|`gs://cloud-training/gsp323/lab.csv`|
|Temporary BigQuery directory|`gs://YOUR_PROJECT/bigquery_temp`|
|Temporary location|`gs://YOUR_PROJECT/temp`|

```
bq mk lab

gsuitl cp gs://cloud-training/gsp323/lab.schema .
cat lab.schema
```
```
gsuitl cp gs://cloud-training/gsp323/lab.csv .
cat lab.csv
```

### Task 2: Run a simple Dataproc job
You have used Dataproc in the quest, now you must run another example Spark job using Dataproc.

Before you run the job, log into one of the cluster nodes and copy the /data.txt file into hdfs (use the command `hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt`).

Run a Dataproc job using the values below.

|Field|Value|
|---|---|
|Region|`us-central1`|
|Job type|`Spark`|
|Main class or jar|`org.apache.spark.examples.SparkPageRank`|
|Jar files|`file:///usr/lib/spark/examples/jars/spark-examples.jar`|
|Arguments|`/data.txt`|

```

```

### Task 3: Run a simple Dataprep job
You have used Dataprep to import data files and transformed them to gain views of the data. Use Dataprep to import one CSV file (described below) that holds data of lab executions.

`gs://cloud-training/gsp323/runs.csv` structure:
|runid|userid|labid|lab_title|start|end|time|score|state|
|---|---|---|---|---|---|---|---|---|
|5556|545|122|Lab 122|2020-04-09 16:18:19|2020-04-09 17:10:11|3112|61.25|SUCCESS|
|5557|116|165|Lab 165|2020-04-09 16:44:45|2020-04-09 18:13:58|5353|60.5|SUCCESS|
|5558|969|31|Lab 31|2020-04-09 17:59:01|2020-04-09 18:02:09|188|0|FAILURE|

Perform the following transforms to ensure the data is in the right state:
- Remove all rows with the state of **"FAILURE"**
- Remove all rows with 0 or 0.0 as a score (Use the regex pattern **/(^0$|^0\.0$)/**)
- Label columns with the names above

```

```

### Task 4: Perform one of three Google machine learning backed API tasks
For the options below **YOUR_PROJECT** must be replaced with your lab project name.

You have the option to complete one of the tasks below:

Use Google Cloud Speech API to analyze the audio file `gs://cloud-training/gsp323/task4.flac`. Once you have analyzed the file you can upload the resulting analysis to `gs://YOUR_PROJECT-marking/task4-gcs.result`.

Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text you can upload the resulting analysis to `gs://YOUR_PROJECT-marking/task4-cnl.result`.

Use Google Video Intelligence and detect all text on the video `gs://spls/gsp154/video/train.mp4`. Once you have completed the processing of the video, pipe the output into a file and upload to `gs://YOUR_PROJECT-marking/task4-gvi.result`.

```

```

## Congratulations!
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.png) ![Badge_cl Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud-cl.png)

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Perform Foundational Data, ML, and AI Tasks in Google Cloud](https://google.qwiklabs.com/quests/117) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. Enroll in this Quest and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests](https://google.qwiklabs.com/catalog).

This skill badge quest is part of Google Cloud’s [Data Analyst](https://cloud.google.com/training/data-ml#data-analyst-learning-path) and [Data Engineer](https://cloud.google.com/training/data-ml#data-engineer-learning-path) learning paths. Continue your learning journey by enrolling in the [Engineer Data with Google Cloud](https://google.qwiklabs.com/quests/132) quest.
