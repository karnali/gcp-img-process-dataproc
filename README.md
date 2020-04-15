# GCP Distributed Image Processing in Cloud Dataproc  

Spin up a Cloud Dataproc cluster in GCP and run distributed Image Processing in Cloud Dataproc.    

This project will demonstarte how to use Apache Spark on Cloud Dataproc to distribute a   
computationally intensive image processing task onto a cluster of machines. We will perform  
the following steps.  

* Create a managed Cloud Dataproc cluster with Apache Spark pre-installed.  
* Build and run jobs that use external packages that aren't already installed on your cluster.  
* Shut down your cluster.  


#### Create a development machine in Compute Engine

##### In the Google Cloud Console, go to  
Compute Engine > VM Instances > Create

##### Configure VM Instances 
-  Name: devhost  
-  Type: 2 vCPUs (n1-standard-2 instance)
-  Identity and API Access: Allow full access to all Cloud APIs.


#### Install Software:

##### Set up Scala and sbt using SSH window
```
$ sudo apt-get install -y dirmngr
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https
$ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
$ sudo apt-get update
$ sudo apt-get install -y bc scala sbt
```

##### Set up the Feature Detector Files

```
$ sudo apt-get update
$ git clone https://github.com/GoogleCloudPlatform/cloud-dataproc
$ cd cloud-dataproc/codelabs/opencv-haarcascade
```

#### Launch build

```
$ sbt assembly
```

#### Create a GCS bucket and collect images

#### Display project info

```
$ GCP_PROJECT=$(gcloud config get-value core/project)
$ MYBUCKET="${USER//google}-image-${RANDOM}"
$ echo MYBUCKET=${MYBUCKET}
```

#### Create bucket

```
$ gsutil mb gs://${MYBUCKET}
```

#### Download some sample images into your bucket:

```
$ curl https://www.publicdomainpictures.net/pictures/10000/velka/albert-einstein-11282050865o83h.jpg | gsutil cp - gs://${MYBUCKET}/imgs/albert-einstein.jpg

$ curl https://www.publicdomainpictures.net/pictures/170000/velka/historical-indian-american-chief-1462985393fHq.jpg | gsutil cp - gs://${MYBUCKET}/imgs/historical-indian-american-chief.jpg

$ curl https://www.publicdomainpictures.net/pictures/20000/velka/father-looking-at-daughter-871294330076QTz.jpg | gsutil cp - gs://${MYBUCKET}/imgs/father-looking-at-daughter.jpg
```

#### List content of your bucket

```
$ gsutil ls -R gs://${MYBUCKET}
```

#### Create a Cloud Dataproc cluster

```
$ MYCLUSTER="${USER/_/-}-qwiklab"
$ echo MYCLUSTER=${MYCLUSTER}
```
```
$ gcloud config set dataproc/region global
$ gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2 
```

#### Submit your job to Cloud Dataproc

Run the following command in the SSH window to load the face detection configuration file into your bucket:
```
$ curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml
```

#### Submit your job to Cloud Dataproc:
```
$ cd ~/cloud-dataproc/codelabs/opencv-haarcascade
$ gcloud dataproc jobs submit spark --cluster ${MYCLUSTER} \
	--jar target/scala-2.10/feature_detector-assembly-1.0.jar \
	-- gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
	gs://${MYBUCKET}/imgs/ \
	gs://${MYBUCKET}/out/
```

#### Monitor the job

```
Console > Navigation menu > Dataproc > Jobs
```

#### Download images to your computer

``` 
Console > Navigation menu > Storage > student-imagexxxx > Out
```
```
onsole > Navigation menu > Storage > student-imagexxxx > imgs
```

#### Try the API 
```
https://cloud.google.com/vision/#industry-leading-accuracy-for-image-understanding
```

