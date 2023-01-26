# Telecom Customer Churn using Airflow and Red Hat OpenShift Data Science

!!!info
    The full source and instructions for this demo are available in **[this repo](https://github.com/red-hat-data-services/telecom-customer-churn-airflow){:target="_blank"}**

## Demo description

The goal of this demo is to demonstrate how RHODS and Airflow can be used together to build an easy-to-manage pipeline.  
To do that, we will show how to deploy an airflow pipeline, both using a DAG script and with Elyra.  
In the end, you will have a pipeline that:

- Pulls data from S3
- Trains two different models
- Evaluates which model is best
- Pushes that model to S3

The models we build are used to predict customer churn for a Telecom company using structured data. The data contains fields such as: If they are a senior citizen, if they are a partner, their tenure, etc.

## Deploying the demo

### Pre-requisites

- Have [Airflow](/tools-and-applications/airflow/airflow/) running in a cluster and point Airflow to this GitHub: [https://github.com/red-hat-data-services/telecom-customer-churn-airflow](https://github.com/red-hat-data-services/telecom-customer-churn-airflow)
- Have [Red Hat OpenShift Data Science](/getting-started/openshift-data-science/) (RHODS) running in a cluster. Make sure you have admin access in RHODS
!!! note
    Note: You can use [Open Data Hub](/getting-started/opendatahub/) instead of RHODS, but some instructions and screenshots may not apply

### 1: Add Elyra as a Custom Notebook Image

Start by opening up RHODS by clicking on the 9 square symbol in the top menu and choosing "Red Hat OpenShift Data Science".

![Open RHODS](img/Open_RHODS.png)

Then go to Settings -> Notebook Images and press "Import new image".  
If you can't see Settings then you are lacking sufficient access, ask your admin to add this image instead.

![Import Notebook Image](img/Import_Notebook_Image.png)

Under Repository enter: `quay.io/thoth-station/s2i-lab-elyra:v0.2.1` and then name it to something like `Elyra`.

### 2: Create a RHODS workbench

A workbench in RHODS lets us spin up and down notebooks as needed and bundle them under Projects, which is a great way to get easy access to compute resources and keep track of your work.  
Start by creating a new Data Science project (see image). I'm calling my project 'Telecom Customer Churn', feel free to call yours something different but be aware that some things further down in the demo may change.

![Create Data Science Project](img/Create_data_science_project.png)

After the project has been created, create a workbench where we can run Jupyter.
There are a few important settings here that we need to set:

- **Name:** Customer Churn
- **Notebook Image:** Elyra
- **Deployment Size:** Small

![Workbench Settings](img/Workbench_Settings.png)

Press Create Workbench and wait for it to start - status should say "Running" and you should be able to press the Open link.

![Workbench](img/Workbench.png)

Open the workbench and login if needed.

### 3: Load a Git repository

When inside the workbench (Jupyter), we are going to clone a GitHub repository that contains everything we need to build our DAG.  
You can clone the GitHub repository by pressing the GitHub button in the left side menu (see image), then select "Clone a Repository" and enter this GitHub URL: [https://github.com/red-hat-data-services/credit-fraud-detection-demo](https://github.com/red-hat-data-services/credit-fraud-detection-demo){:target="_blank"}

![Jupyter](img/Jupyter.png)

### 4: Set up an S3 bucket with the data

### 5.1: Create the DAG with Elyra

First we need to add Elyra as a notebook image. We do that by adding a custom notebook image through the RHODS interface:


The notebook image address is: quay.io/thoth-station/s2i-lab-elyra:v0.2.1

### 5.2: Use a DAG file

### 6: Execute the DAG and see the results
