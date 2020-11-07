# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab
- [Qwiklabs Link](https://www.qwiklabs.com/focuses/10379?parent=catalog)

## Challenge scenario
You are now asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called memories. You have been asked to assist the memories team with initial configuration for their application development environment; you receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

Some standards you should follow:
- Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications such as Kubernetes nodes.


### Task 1: Create a bucket
You need to create a bucket for the storage of the photographs. Please remove '[ ]'.
```
gsutil mb gs://[YOUR-BUCKET-NAME]/
```

### Task 2: Create a Pub/Sub topic
Create a Pub/Sub topic for the Cloud Function to send messages.
```
gcloud pubsub topics create [YOUR-TOPIC-NAME]
```

### Task 3: Create the Cloud Function
Create a Cloud Function that executes every time an object is created in the bucket you created in task 1. The function is written in Node.js 10. Make sure you set the **Function to execute** to **thumbnail**.

In line 15 of index.js replace the text **REPLACE_WITH_YOUR_TOPIC** with the topic you created in task 2.

```
nano index.js
```

**index.js:**
```
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const gcs = require("@google-cloud/storage")();
const PubSub = require("@google-cloud/pubsub");
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "[REPLACE_WITH_YOUR_TOPIC]";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });

          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
Exit nano (Ctrl+x) and save (Y) the file.

```
nano package.json
```

**package.json:**
```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/storage": "1.5.1",
    "@google-cloud/pubsub": "^0.18.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

```
gcloud functions deploy thumbnail \
--stage-bucket [YOUR-BUCKET-NAME] \
--trigger-resource [YOUR-BUCKET-NAME] \
--trigger-event google.storage.object.finalize \
--runtime nodejs10
```
You must upload one JPG or PNG image into the bucket, we will verify the thumbnail was created. Use any JPG or PNG image, or use [this image](https://storage.googleapis.com/cloud-training/gsp315/map.jpg); download the image to your machine and then upload that file to your bucket. You will see a thumbnail image appear shortly afterwards.

### Task 4: Remove the previous cloud engineer
You will see that there are two users, one is your account (with the role of Owner) and the other is the previous cloud engineer (with the role of Viewer). We like to keep our security tight, so please remove the previous cloud engineer’s access to the project.

Navigate to the IAM & Admin console, select **Navigation menu > IAM & Admin > IAM.**
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-IAM-1.png)
![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-IAM-2.png)


## Congratulations!

![Badge Image](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.png)

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Baseline: Infrastructure](https://google.qwiklabs.com/quests/33) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. Enroll in a Quest and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests.](https://google.qwiklabs.com/catalog)
