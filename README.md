# Qwiklabs - Perform Foundational Infrastructure Tasks in Google Cloud Challenge Lab

Step by Step guide to solve this challenge. Can refer my YouTube video https://youtu.be/eJxeGaTBdH0

#Task 1  
Create a bucket   

Navigation menu -> Storage ->Create Bucket        
Name your bucket -> Your GCD Project-ID                     
Location Type ->Region                         
Location -> us-east1                   
Rest all value to default             
Create        

#Task 2
Create a Pub/Sub topic     

Navigation menu -> Pub/Sub ->Create Topic               
Topic ID ->Jooli                   
Encryption -> default             
Create              

#Task 3             
Create a Cloud Function.             

Navigation menu -> Cloud Functions -> Create function                  

Name -> CFJooli

Trigger -> Cloud Storage

Event Type -> Finalize/Create

Bucket -> Select bucket you created

Entry point -> thumbnail

In line 15 of index.js replace the text REPLACE_WITH_YOUR_TOPIC ID with the Topic ID you created in task 2.

index.js                 
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
  const topicName = "Jooli";                  
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
                      
package.json                      
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
             
Create             

use this image                      
https://storage.googleapis.com/cloud-training/gsp315/map.jpg;                      
download the image to your machine and then upload that file to your bucket.                         

To upload image                        
Navigation menu ->Storage -> Click on the bucket you created                     
Upload image downloaded                 
Refresh Bucket -> you will notice thumbnail in the list       

#Task 4                   
Remove the previous cloud engineer’s access from the memories project.              

Navigation menu ->IAM & Admin
Find Username 2 in the list -> Click on three dots on right -> Click on Trashbin ->Save
