

Overview
Visual Inspection AI Cosmetic Inspection inspects products to detect and recognize defects such as dents, scratches, cracks, deformations, foreign materials, etc. on any kind of surface such as those shown in the following image.

Composite image of six defects

In this lab, you will deploy a pre-prepared Cosmetic Inspection anomaly detection solution artifact and use it to analyze sample images.

Model training can take a long time, so this lab is paired with Creating a Cosmetic Anomaly Detection Model using Visual Inspection AI. You are provided with the trained Cosmetic Inspection anomaly detection solution artifact that was created using the same dataset used in that lab.

Objectives
In this lab you will learn how to complete the following tasks:

Deploy a trained Cosmetic Inspection anomaly detection solution artifact.
Perform batch prediction using an Cosmetic Inspection anomaly detection solution artifact.
Setup and requirements
Before you click the Start Lab button
Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click Start Lab, shows how long Google Cloud resources will be made available to you.

This hands-on lab lets you do the lab activities yourself in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials that you use to sign in and access Google Cloud for the duration of the lab.

To complete this lab, you need:

Access to a standard internet browser (Chrome browser recommended).
Note: Use an Incognito or private browser window to run this lab. This prevents any conflicts between your personal account and the Student account, which may cause extra charges incurred to your personal account.
Time to complete the lab---remember, once you start, you cannot pause a lab.
Note: If you already have your own personal Google Cloud account or project, do not use it for this lab to avoid extra charges to your account.
How to start your lab and sign in to the Google Cloud console
Click the Start Lab button. If you need to pay for the lab, a pop-up opens for you to select your payment method. On the left is the Lab Details panel with the following:

The Open Google Cloud console button
Time remaining
The temporary credentials that you must use for this lab
Other information, if needed, to step through this lab
Click Open Google Cloud console (or right-click and select Open Link in Incognito Window if you are running the Chrome browser).

The lab spins up resources, and then opens another tab that shows the Sign in page.

Tip: Arrange the tabs in separate windows, side-by-side.

Note: If you see the Choose an account dialog, click Use Another Account.
If necessary, copy the Username below and paste it into the Sign in dialog.

student-00-13a66bd7698c@qwiklabs.net
Copied!
You can also find the Username in the Lab Details panel.

Click Next.

Copy the Password below and paste it into the Welcome dialog.

ZiBMqrpXqlm5
Copied!
You can also find the Password in the Lab Details panel.

Click Next.

Important: You must use the credentials the lab provides you. Do not use your Google Cloud account credentials.
Note: Using your own Google Cloud account for this lab may incur extra charges.
Click through the subsequent pages:

Accept the terms and conditions.
Do not add recovery options or two-factor authentication (because this is a temporary account).
Do not sign up for free trials.
After a few moments, the Google Cloud console opens in this tab.

Note: To view a menu with a list of Google Cloud products and services, click the Navigation menu at the top-left. Navigation menu icon
Activate Cloud Shell
Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

Click Activate Cloud Shell Activate Cloud Shell icon at the top of the Google Cloud console.
When you are connected, you are already authenticated, and the project is set to your Project_ID, qwiklabs-gcp-02-dbf6642b4b34. The output contains a line that declares the Project_ID for this session:

Your Cloud Platform project in this session is set to qwiklabs-gcp-02-dbf6642b4b34
gcloud is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

(Optional) You can list the active account name with this command:
gcloud auth list
Copied!
Click Authorize.
Output:

ACTIVE: *
ACCOUNT: student-00-13a66bd7698c@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
(Optional) You can list the project ID with this command:
gcloud config list project
Copied!
Output:

[core]
project = qwiklabs-gcp-02-dbf6642b4b34
Note: For full documentation of gcloud, in Google Cloud, refer to the gcloud CLI overview guide.
Task 1. Deploy the exported Cosmetic Inspection anomaly detection solution artifact
A solution container from Creating a Cosmetic Anomaly Detection Model using Visual Inspection AI is stored in the Container Registry (gcr.io) location. You can deploy the exported solution container in your own environment, whether that is a Google Cloud VM or in an on-premise compute unit.

Run and test a CPU-based solution artifact locally
In this task, you will deploy an exported assembly inspection solution artifact by running it as a container using Docker in Google Cloud VM, that is created for you. The process shown here uses docker commands to pull, or start the Docker compatible solution artifact container.

The exported solution artifact container uses port 8601 for grpc traffic, port 8602 for http traffic, and port 8603 for Prometheus metric traffic. You can map these ports to locally available ports in the VM environment when starting the container with Docker using the command line switches -v 9000:8602 or -v 3006:8603.

You need to map a local port to port 8602 for sending http requests, and another local port to port 8603 if you want to see the metrics logs locally.

Your first step is to connect to the Google Cloud VM. In Cloud Shell, run the following command to connect to the VM:
gcloud compute ssh lab-vm --zone us-east1-c
Copied!
Type Y to continue.

For any prompts, press Enter to continue.

Define an environment variable to store the name of the Visual Inspection solution artifact Docker image that is stored in Container Registry.

In the terminal, define a variable named DOCKER_TAG using the full image name of the docker container:
export DOCKER_TAG=gcr.io/ql-shared-resources-test/defect_solution@sha256:776fd8c65304ac017f5b9a986a1b8189695b7abbff6aa0e4ef693c46c7122f4c
Copied!
This is the Container Registry image ID of a Visual Inspection Cosmetic Inspection anomaly detection solution artifact that was trained using the Visual Inspection cosmetic defect demo dataset. This solution artifact will identify cosmetic defects in images.

Define variables for ports used by the solution artifact container:
export VISERVING_CPU_DOCKER_WITH_MODEL=${DOCKER_TAG}
export HTTP_PORT=8602
export LOCAL_METRIC_PORT=8603
Copied!
Pull the docker image from the Google Source Repository using the following command:
docker pull ${VISERVING_CPU_DOCKER_WITH_MODEL}
Copied!
Start the solution artifact locally using Docker in the VM:
docker run -v /secrets:/secrets --rm -d --name "test_cpu" \
--network="host" \
-p ${HTTP_PORT}:8602 \
-p ${LOCAL_METRIC_PORT}:8603 \
-t ${VISERVING_CPU_DOCKER_WITH_MODEL} \
--metric_project_id="${PROJECT_ID}" \
--use_default_credentials=false \
--service_account_credentials_json=/secrets/assembly-usage-reporter.json
Copied!
The reported usage metrics uploaded from the container include:

num_request_processed: the total number of processed requests.
prediction_latency: the prediction latency in one request.
average_prediction_latency: the average prediction latency.
Confirm that the container is running:
docker container ls
Copied!
You should see a container listed that has the image name for your solution artifact.

Once you have started the solution artifact container with the above Docker run command, you can send requests to the running container using the python script below.

Copy the file prediction_script.py to run predictions by calling Visual Inspection AI rest APIs via the solution artifact container:
gsutil cp gs://cloud-training/gsp895/prediction_script.py .
Copied!
The code in this file is displayed below for your reference.

from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import base64 import json import time import re

from absl import app from absl import flags import numpy as np import requests

flags.DEFINE_string('hostname', 'http://localhost', 'The hostname for serving.') flags.DEFINE_string('input_image_file', None, 'The input image file name.') flags.DEFINE_string('output_result_file', None, 'The prediction output file name.') flags.DEFINE_integer('port', None, 'The port of rest api.') flags.DEFINE_integer('num_of_requests', 1, 'The number of requests to send.')

FLAGS = flags.FLAGS

def create_request_body(input_image_file): """Creates the request body to perform api calls.

    Args:
    input_image_file: String, the input image file name.

    Returns:
    A json format string of the request body. The format is like below:
        {"image_bytes":}
    """

    with open(input_image_file, 'rb') as image_file:
        encoded_string = base64.b64encode(image_file.read()).decode('utf-8')

    request_body = {'image_bytes': str(encoded_string)}

    return json.dumps(request_body)

def predict(hostname, input_image_file, port): """Predict results on the input image using services at the given port.

    Args:
    hostname: String, the host name for the serving.
    input_image_file: String, the input image file name.
    port: Integer, the port that runs the rest api service.

    Returns:
    The predicted results in json format.
    """
    url = hostname + ':' + str(port) + '/v1beta1/visualInspection:predict'
    request_body = create_request_body(input_image_file)
    response = requests.post(url, data=request_body)
    return response.json()

def compute_latency_percentile(hostname, input_image_file, port, num_of_requests): """Computes latency percentiles of server's prediction endpoint.

    Args:
    hostname: String, the host name for the serving.
    input_image_file: String, the input image file name.
    port: Integer, the port that runs the rest api service.
    num_of_requests: The number of requests to send.

    Returns:
    The dictionary of latency percentiles of 75%, 90%, 95%, 99%.
    """
    latency_list = []

    for _ in range(num_of_requests):
        response = predict(hostname, input_image_file, port)
        latency_in_ms = float(response['predictionLatency'][:-1])
        latency_list.append(latency_in_ms)

    latency_percentile = {}
    percentiles = [75, 90, 95, 99]
    for percentile in percentiles:
        latency_percentile[percentile] = np.percentile(latency_list, percentile)

    return latency_percentile

def main(\_): if FLAGS.num_of_requests > 1: latency_percentile = compute_latency_percentile(FLAGS.hostname, FLAGS.input_image_file, FLAGS.port, FLAGS.num_of_requests) print(latency_percentile) with open(FLAGS.output_result_file, 'w+') as latency_result: latency_result.write(json.dumps(latency_percentile)) else: start = time.time() results = predict(FLAGS.hostname, FLAGS.input_image_file, FLAGS.port) end = time.time() print('Processed image {} in {}s.'.format(FLAGS.input_image_file, end - start)) print(json.dumps(results, indent=2)) with open(FLAGS.output_result_file, 'w+') as prediction_result: prediction_result.write(json.dumps(results, indent=2))

if **name** == '**main**': flags.mark_flag_as_required('input_image_file') flags.mark_flag_as_required('port') flags.mark_flag_as_required('output_result_file') app.run(main)
Click Check my progress to verify the objective.
Step successfully completed.
Deploy the exported Cosmetic Inspection anomaly detection solution artifact

Step successfully completed.

Task 2. Serve the exported assembly inspection solution artifact
Identifying a defective product
In the VM terminal, run the commands below to copy training images to your Cloud Storage bucket, followed by copying a defective sample image to the VM:
export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp897/cosmetic-test-data/*.png \
gs://${PROJECT_ID}/cosmetic-test-data/
gsutil cp gs://${PROJECT_ID}/cosmetic-test-data/IMG_07703.png .
Copied!
Next, find a defective product from the cloud storage bucket. In the cloud console, go to Navigation menu > Cloud Storage > Buckets.

In the Cloud Storage, navigate to the bucket named qwiklabs-gcp-02-dbf6642b4b34 > cosmetic-test-data.

Now find and click on the image named IMG_07703.png from the list of the images. You might have to scroll through the list to find this image.

Sample defect image

This is an image of a defective product that has a scratch on the surface.

In the terminal, run the following commands to install the python envrionment and packages:
sudo apt install python3 -y
sudo apt install python3-pip -y
sudo apt install python3.11-venv -y 
python3 -m venv create myvenv
source myvenv/bin/activate
pip install absl-py  
pip install numpy 
pip install requests
Copied!
In the terminal, run the following command to send the selected image as a request to the solution artifact container:
python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --output_result_file=def_prediction_result.json
Copied!
This will print out the JSON result data returned by your Visual Inspection AI model. You can inspect the annotation sets and annotations to see that this returns the same data structures that are returned when batch predictions are performed using the UI.

However this sample image is one of the images with a cosmetic defect. If you look at the annotation results, you can see that Visual Inspection AI has identified a scratch in this particular image.

The script also stores the prediction result in a file named def_prediction_result.json that is created and saved in the HOME directory of the VM. This file is passed to the script using the flag --output_result_file.

...
  "predictionResult": {
    "annotationsGroups": [
      {
...     "annotations": [
          {
            "name": "localAnnotations/1",
            "annotationSetId": "2435347886779662336",
            "mask": {
...
              "annotationSpecColors": [
                {
                  "annotationSpecId": "3513930310720946176",
                  "color": {
                    "red": 0.717647076,
                    "green": 0.615686297,
                    "blue": 0.356862754
                  },
                  "annotationSpecDisplayName": "scratch"
                }
              ]
...
  "predictionLatency": "1.299129679s"
Run the following command to send multiple requests to the running container:
python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --num_of_requests=10 --output_result_file=def_latency_result.json
Copied!
The script calls the solution artifact 10 times and reports the distribution of the response latencies that are returned in the response each time the solution artifact processes an image. The script also stores the latency result in a file named def_latency_result.json that is created and saved in the HOME directory of the VM. This file is passed to the script using the flag --output_result_file.

Click Check my progress to verify the objective.
Step successfully completed.
Identify defective product

Step successfully completed.

Identifying a non-defective product
In the VM terminal, run the commands below to copy a non-defective sample image to the VM:
export PROJECT_ID=$(gcloud config get-value core/project)
gsutil cp gs://${PROJECT_ID}/cosmetic-test-data/IMG_0769.png .
Copied!
Next, find a non-defective product from the cloud storage bucket. If you are not in Cloud Storage browser, in cloud console, go to Navigation menu (Navigation menu icon) > Cloud Storage > Buckets. In the Cloud Storage Browser, navigate to the bucket named qwiklabs-gcp-02-dbf6642b4b34 > cosmetic-test-data.

Now find and click on the image named IMG_0769.png from the list of the images. You might have to scroll through the list to find this image.

Sample image of non-defective product

This is a non-defective product that does not have any defects.

In the terminal, run the following command to send the selected image as a request to the solution artifact container:
python3 ./prediction_script.py --input_image_file=./IMG_0769.png  --port=8602 --output_result_file=non_def_prediction_result.json
Copied!
This will print out the JSON result data returned by your Visual Inspection AI model. You can inspect the annotation sets and annotations to see that this returns the same general data structures that were returned in the previous prediction.

However this sample image does not have any cosmetic defects present. If you look at the annotation results, you can see that Visual Inspection AI has not identified any region with either of the two defect types this model has been trained to identify, scratches or dents.

The script also stores the prediction result in a file named non_def_prediction_result.json that is created and saved in the HOME directory of the VM. This file is passed to the script using the flag --output_result_file.

{
  "predictionResult": {
    "annotationsGroups": [
      {
...
        "annotations": [
          {
            "name": "localAnnotations/1",
            "annotationSetId": "2435347886779662336",
            "mask": {
...
              "annotationSpecColors": [
                {
                  "annotationSpecId": "4043103266936979456",
                  "color": {},
                  "annotationSpecDisplayName": "none"
                }
              ]
...
  "predictionLatency": "0.414128490s"
}
Run the following command in the VM terminal to send multiple requests to the running container:
python3 ./prediction_script.py --input_image_file=./IMG_0769.png  --port=8602 --num_of_requests=10 --output_result_file=non_def_latency_result.json
Copied!
The script calls the solution artifact 10 times and reports the distribution of the response latencies that are returned in the response each time the solution artifact processes an image.

The script also stores the latency result in a file named non_def_latency_result.json that is created and saved in the HOME directory of the VM. This file is passed to the script using the flag --output_result_file.

Click Check my progress to verify the objective.
Step successfully completed.
Identify non-defective product

Step successfully completed.

Congratulations!
Congratulations! In this lab, you deployed a pre-prepared Cosmetic Inspection anomaly detection solution artifact and used it to analyze sample images. You also learned how to perform batch prediction using an Cosmetic Inspection anomaly detection solution artifact.
