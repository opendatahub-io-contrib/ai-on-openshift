# Credit Card Fraud Detection Demo using MLFlow and RHODS
[GitHub Source](https://github.com/red-hat-data-services/credit-fraud-detection-demo)

## Pre-requisites:
- Have [RHODS](/getting_started_rhods) running in a cluster
- Have [MLFlow](/tools_and_applications_mlflow) running in a cluster

## Demo Description & Architecture
The goal of this demo is to demonstrate how RHODS and MLFlow can be used together to build an end-to-end MLOps platform where we can: 

- Build and train models in RHODS
- Track and store those models with MLFlow
- Deploy a model application in OpenShift that predicts with a specific model from MLFlow

The architecture looks like this:
![Diagram](img/Diagram.PNG)

For S3 we use ODF (OpenShift Data Foundation) according to the [MLFlow guide](/tools_and_applications_mlflow), but it can be replaced with another storage.  
The monitoring component uses Prometheus and Grafana.

The model we will build is a Credit Card Fraud Detection model, which predicts if a credit card usage is fraudulent or not depending on a few parameters such as: distance from home and last transaction, purchase price compared to median, if it's from a retailer that already has been purchased from before, if the PIN number is used and if it's an online order or not.


## Demo

### 1.1: MLFlow Route through the visual interface
Start by finding your route to MLFlow. You will need it to send any data to MLFlow.

- Go to the OpenShift Console as a Developer
- Select your mlflow project 
- Press Topology 
- Press the mlflow-server circle 
    - While you are at it, you can also press the little "Open URL" button in the top right corner of the circle to open up the MLFlow UI in a new tab - we will need it later.
- Go to the Resources tab 
- Press mlflow-server under Services 
- Look at the Hostname and mlflow-server Port.  
NOTE: This route and port only work internally in the cluster.
![Find mlflow-server-service](img/mlflow-server-service.png)
![Find the hostname and port](img/hostname-and-port.png)

### 1.2: MLFlow Route through oc
Alternatively, you can use the OC command to get the route through: `oc get route mlflow | grep mlflow`

### 2: Create a RHODS workbench
Open up RHODS and create a new Data Science project (see image), this is where we will build and train our model. This will also create a namespace in OpenShift which is where we will be running our application after the model is done.  
I'm calling my project 'Credit Fraud', feel free to call yours something different but be aware that some things further down in the demo may change.
![Create Data Science Project](img/Create_data_science_project.png)

After the project has been created, create a workbench where we can run Jupyter.  
There are a few important settings here that we need to set:

- **Name:** I simply call it "Credit Fraud Model", but feel free to call it something else.
- **Notebook Image:** Standard Data Science
- **Deployment Size:** Small
- **Environment Variable:** Add a new one that's a Config Map -> Key/value and enter `MLFLOW_ROUTE` as the key and `http://<route-to-mlflow>:<port>` as the value.  
This is the same route and port that we found in [step one](#11-mlflow-route-through-visual-interface), in my case it is `http://mlflow-server.mlflow.svc.cluster.local:8080`.
- **Cluster Storage:** Create new persistent storage - I call it "Credit Fraud Storage" and set the size to 20GB.

Press Create Workbench and wait for it to start - status should say "Running" and you should be able to press the Open link
![Workbench Settings](img/Workbench_Settings.png)
![Workbench](img/Workbench.png)

Open the workbench and login if needed.

### 3: Train the model
When inside the workbench (Jupyter), pull down this repo from GitHub: [https://github.com/red-hat-data-services/credit-fraud-detection-demo](https://github.com/red-hat-data-services/credit-fraud-detection-demo).   
It contains:

- A notebook (model.ipynb) inside the `model` folder with a Deep Neural Network model we will train.
- Data for that model.
- An application (model_application.py) inside the `application` folder that will fetch the trained model from MLFlow and run a prediction on it whenever it gets any user input.
![Jupyter](img/Jupyter.png)

Take a look inside the folders and scripts if you are interested.  
Some things to note is how we set up tracking and logging with MLFlow in `model.ipynb` to make sure that the experiment and the model is captured. You can also read about that in the [MLFlow guide](/tools_and_applications_mlflow#adding-mlflow-to-training-code).

When you are done, open up the `model.ipynb` notebook and run all the cells. If everything is set up correctly it will train the model and push both the experiment and the model to MLFlow.  
The experiment is a record with metrics of how the run went, while the model is the actual tensorflow model which we later will use for inference.  
You may see some warnings in the last cell related to MLFlow, as long as you see a final progressbar for the model being pushed to MLFlow you are fine:
![Trained model](img/Trained_model.png)

### 4: View the model in MLFlow
Let's take a look at how it looks inside MLFlow now that we have trained the model.  
If you opened the MLFlow UI in a new tab in step 1.1, then just swap over to that tab, otherwise follow these steps:

- Go to the OpenShift Console
- Make sure you are in Developer view in the left menu
- Go to Topology in the left menu
- At the top left, change your project to MLFlow (or whatever you called it when installing the MLFlow operator in pre-requisites)
- Press the "Open URL" icon in the top right of the MLFlow circle in the topology map

![Open MLFlow UI](img/Open_MLFlow_UI.png)

When inside the MLFlow interface you should see your new experiment in the left menu. Click that to see all the runs, should be a single run from the model we just trained.  
You can now click on the row in the Created column to get more information about the experiment and how to run the model.

![MLFlow view](img/MLFlow_view.png)

### 5: Deploy the model
To deploy the model, go to the OpenShift Console and make sure you are in **Developer** view and have selected the **credit-fraud** project.  
Then press "+Add" in the left menu and select Import from Git.  
![Import from Git](img/Import_from_Git.png)
In the "Git Repo URL" enter: [https://github.com/red-hat-data-services/credit-fraud-detection-demo](https://github.com/red-hat-data-services/credit-fraud-detection-demo) (this is the same repository we pulled into RHODS earlier).  
Then press "Show advanced Git options" and set "Context dir" to "/application".  
Press Create to start deploying the application.
![Import from Git Settings](img/Import_from_Git_settings.png)
You should now see two objects in your topology map, one for the Workbench we created earlier and one for the application we just added.  
When the circle of your deployment turns dark blue it means that it has finished deploying.  
If you want more details on how the deployment is going, you can press the circle and look at Resources in the right menu that opens up. There you can see how the build is going and what's happening to the pod. The application will be ready when the build is complete and the pod is "Running".  
When the application has been deployed you can press the "Open URL" button to open up the interface in a new tab. 
![Application deployed](img/Application_deployed.png)
Congratulations, you now have an application running your AI model!  
If you looked inside the application code earlier, you also know that we specifically pull version 1 of the model called "DNN Fraud Detection" from MLFlow. This makes sense since we only ran the model once, but is easy to change if any other version or model should go into production.  
We are also utilizing a program called "Gradio" to create the interface, it's a super lightweight way to get a nice-looking interface running.

Try entering a few values and see if it predicts it as a credit fraud or not. You can select one of the examples at the bottom of the application page.
![Gradio](img/Gradio.PNG)