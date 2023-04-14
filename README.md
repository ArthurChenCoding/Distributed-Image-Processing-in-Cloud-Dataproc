[![CI](https://github.com/nogibjj/mlops-template/actions/workflows/cicd.yml/badge.svg?branch=GPU)](https://github.com/nogibjj/mlops-template/actions/workflows/cicd.yml)
[![Codespaces Prebuilds](https://github.com/nogibjj/mlops-template/actions/workflows/codespaces/create_codespaces_prebuilds/badge.svg?branch=GPU)](https://github.com/nogibjj/mlops-template/actions/workflows/codespaces/create_codespaces_prebuilds)

## Distributed Image Processing in Cloud Dataproc
This project uses Apache Spark on Cloud Dataproc to distribute a computationally intensive image processing task onto a cluster of machines

## step 1: create a devlopment machine in compute engine on google cloud with the following configuration
```
Name: devhost

Series: N1

Machine Type: 2 vCPUs (n1-standard-2 instance)

Identity and API Access: Allow full access to all Cloud APIs.
```

## step 2: install software
```
sudo apt-get install -y dirmngr unzip
sudo apt-get update
sudo apt-get install -y apt-transport-https
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
sudo apt-get update
sudo apt-get install -y bc scala sbt
sbt assembly
```
## step 3: Task 3. Create a Cloud Storage bucket and collect images
```
GCP_PROJECT=$(gcloud config get-value core/project)
MYBUCKET="${USER//google}-image-${RANDOM}"
echo MYBUCKET=${MYBUCKET}
gsutil mb gs://${MYBUCKET}
curl https://www.publicdomainpictures.net/pictures/20000/velka/family-of-three-871290963799xUk.jpg | gsutil cp - gs://${MYBUCKET}/imgs/family-of-three.jpg
curl https://www.publicdomainpictures.net/pictures/10000/velka/african-woman-331287912508yqXc.jpg | gsutil cp - gs://${MYBUCKET}/imgs/african-woman.jpg
curl https://www.publicdomainpictures.net/pictures/10000/velka/296-1246658839vCW7.jpg | gsutil cp - gs://${MYBUCKET}/imgs/classroom.jpg
gsutil ls -R gs://${MYBUCKET}
```
## Task 4. Create a Cloud Dataproc cluster
```
MYCLUSTER="${USER/_/-}-qwiklab"
echo MYCLUSTER=${MYCLUSTER}
gcloud config set dataproc/region us-west1
gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2 --initialization-actions=gs://spls/gsp010/install-libgtk.sh --image-version=2.0  
```
## Task 5. Submit job to Cloud Dataproc
```
curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml
cd ~/cloud-dataproc/codelabs/opencv-haarcascade
gcloud dataproc jobs submit spark \
--cluster ${MYCLUSTER} \
--jar target/scala-2.12/feature_detector-assembly-1.0.jar -- \
gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
gs://${MYBUCKET}/imgs/ \
gs://${MYBUCKET}/out/
```

### please watch demo video for more detail
