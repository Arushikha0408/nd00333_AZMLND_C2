# Operationalizing Machine Learning

## Project overview
In this project, we need to configure a cloud-based machine learning production model using Azure, deploy it and consume it. The aim of the project was to learn about Operationalizing Machine Learning model by deploying a machine learning model to an endpoint which is a URL that can be easily accessed by any end-user. I will also create, publish, and consume a pipeline. In the end, I will fully demonstrate all of my work in this README file.

I will be working with the Bank Marketing dataset, it is related with direct marketing campaigns (phone calls) of a Portuguese banking institution. The classification goal is to predict if the client will subscribe a term deposit (y). It consists of 20 input variables (columns) and 32,950 rows with 3,692 positive classes and 29,258 negative classes. I will use AutoML to train a model, then deploy the model as a REST endpoint, and test that it's working.

The bank marketing dataset used in this project can be found in the link below: https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv

## Architectural Diagram
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/architecture.PNG)

The diagram above shows the workflow of operationalizing machine learning starting from the creation of an experiment using Automated ML, deployment of the best performing model after the completion of the experiment, enabling Application Insights and retrieving logs, consuming the deployed model using Swagger and lastly, consuming the deployed model endpoints by using the endpoint.py script provided to interact with the trained model.

## Key Steps
**Step-1: Upload the bank marketing datasets to the azure machine learning studio.**

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/dataset1.PNG)

**Step-2: Create an experiment using Automated ML, configure a compute cluster with vm_size of 'STANDARD_DS12_V2, and use that cluster to run the experiment.**

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/automl_run1.PNG)

**Step-3: Get the best performing model to be VotingEnsemble with an AUC_weighted score of 0.94550.**

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/votingensemble1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/votingensemble2.PNG)

**Step-4: Besides the VotingEnsemble model, we have other models that were generated during the iteration procees.**

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/otheralgo.PNG)

**Step-5: Deploy the Best Model(VotingEnsemble).**

Go to the Automated ML section and find the recent experiment with a completed status. Click on it. Go to the "Model" tab and select the votingensemble model from the list and click it. Above it, a triangle button (or Play button) will show with the "Deploy" word. Click on it. Then fill out the form with a meaningful name and description. For Compute Type use Azure Container Instance (ACI) and Enable Authentication. Do not change anything in the Advanced section. Then deploy. Deployment takes a few seconds. After a successful deployment, a green checkmark will appear on the "Run" tab and the "Deploy status" will show as succeed.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/deploy_details1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/deploy_details2.PNG)

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/deploy_status.PNG)

**Step-6: Enable Application Insights.**

Download the config.json file from the top left menu in the Azure portal. Put this file in the same directory of other files needed for this project. Find the previously deployed model to verify its name. It is needed in the SDK to select it for enabling logging. In this example, exercise-deployment-1 is the name of the service. This information is available from the Endpoints section. To enable application insights, we'll add: service.update(enable_app_insights=True) to the logs.py file to enable logging.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs2.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs3.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs4.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs5.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/logs6.PNG)

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/deploy_authentication_made_true.PNG)

**Step-7: Swagger documentation**

Ensure that Docker is installed on your computer. Azure provides a Swagger JSON file for deployed models, so head to the Endpoints section, and find your deployed model there. Click on the name of the model, and details will open that contains a Swagger URI section. Download the file locally to your computer and put it in the same directory with serve.py and swagger.sh. Serve.py will start a Python server on port 8000. This script needs to be right next to the downloaded swagger.json file. NOTE: this will not work if swagger.json is not on the same directory. Since I didn't have permissions for port 80 on my computer, I updated the script to a higher number of 9000. I made use of localhost on port 9000 to display the Swagger page while ensuring that the updated port is used when trying to reach the swagger instance by localhost, for example localhost:9000/swagger.json

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/swagger1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/swagger2.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/swagger3.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/swagger4.PNG)

**Step-8: Modify the scoring uri and key to match the deployed service generated and execute the endpoint.py file with python. You can then benchmark the endpoint using Apache benchmark to run against the HTTP API using authentication keys to retrieve performance.**

In Azure ML Studio, head over to the "Endpoints" section and find a previously deployed model. The compute type should be ACI (Azure Container Instance). In the "Consume" tab, of the endpoint, a "Basic consumption info" will show the endpoint URL and the authentication types. Take note of the URL and the "Primary Key" authentication type. Using the provided endpoint.py replace the scoring_uri and key to match the REST endpoint and primary key respectively. The script issues a POST request to the deployed model and gets a JSON response that gets printed to the terminal. A data.json file will appear after you run endpoint.py.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/benchmark1.PNG)

Make sure you have the Apache Benchmark command-line tool installed and available in your path. Run the endpoint.py. Just like before, it is important to use the right URI and Key to communicate with the deployed endpoint. A data.json should be present. This is required for the next step where the JSON file is used to HTTP POST to the endpoint. In the provided started code, there is a benchmark.sh script with a call to ab.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/benchmark2.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/benchmark3.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/benchmark4.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/benchmark5.PNG)


**Step-9: Create a Pipeline.**

Open up the Jupyter Notebook and make sure you replace all of the URIs, Keys, and experiment names to match your own. Anywhere noted, ensure that the right components are replaced as shown in the next screenshot. Run the Jupyter Notebook all the way up until the Examine Results section. In the end, the pipeline should be available in Azure ML Studio in the Pipelines section. Clicking on the Pipeline should take you to the experiment that demonstrates the graph using the Bankmarketing Dataset and the Auto ML Model. You can choose to keep running through the cells in the Examine Results section to retrieve metrics and the best model.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/pipeline1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/pipeline1.1.PNG)


**Step-10: Publish a pipeline either in the ML studio or with the Python SDK.**

ML Studio: In Azure ML Studio, under the Pipelines section, you will get to a list of all the pipelines available. Click on a Run ID that has a status of Completed. Click on the Publish button so that the overlay menu shows up, and fill it with something descriptive. You can either re-use an endpoint, or create a new one.

Python SDK: Create experiment and pipeline_run. The experiment and run_id of that experiment are crucial. Update the experiment name, project_folder and the run id. Once the pipeline_run object is created, you can publish the pipeline.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/published_pipeline1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/published_pipeline2.PNG)

A screenshot of the Jupyter Notebook is included in the submission showing the “Use RunDetails Widget” with the step runs
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/RunDetails1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/RunDetails2.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/RunDetails3.PNG)

**Step-11: Consume a pipeline.**

Once the pipeline is published, you can authenticate. Next, the published pipeline will be used to retrieve the endpoint. This endpoint is the URI that the SDK will use to communicate with it over HTTP requests. Once the Jupyter Notebook completes all of its steps, the Pipeline will be triggered and available in Azure ML Studio.

![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/pipeline2.1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/bakmarketing_pipeline1.PNG)
![alt text](https://github.com/Arushikha0408/nd00333_AZMLND_C2/blob/master/starter_files/bakmarketing_pipeline.PNG)

## Screen Recording
Link to the screen recording is - https://drive.google.com/file/d/14xA6woAFR6t0prLCYmHZ3hadkEDm5MBb/view?usp=sharing

## Ideas for Improvement 
If you more larger dataset and introduce deep learning model into picture, than we would have got better response. Also, we can use ParallelRunSteps that can help in creating an Azure ML Pipeline step to process large amount of data asynchronously and in parallel. It is a resilient and highly available solution. ParallelRunStep is flexibly designed for a variety of workloads. It’s not just for batch inference, but also other workloads which necessitate parallel processing, for example, training many models concurrently, or processing large amount of data.
