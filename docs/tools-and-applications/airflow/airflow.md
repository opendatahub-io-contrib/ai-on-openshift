
# Apache Airflow

## What is it?

![logo](img/logo.png){ align=left .noborder width=35%}[**Apache Airflow**](https://airflow.apache.org/){:target="_blank"} is a platform created by the community to programmatically author, schedule and monitor workflows.  
It has become popular because of how easy it is to use and how extendable it is, covering a wide variety of tasks and allowing you to connect your workflows with virtually any technology.  
Since it's a Python framework it has also gathered a lot of interest from the Data Science field.

One important concept used in Airflow is **DAGs** (Directed Acyclical Graphs).  
A DAG is a graph without any cycles. In other words, a node in your graph may never point back to a node higher up in your workflow.  
DAGs are used to model your workflows/pipelines, which essentially means that you are building and executing graphs when working with Airflow.  
You can read more about DAGs here: [https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html)

The key features of Airflow are:

- **Webserver**: It's a user interface where you can see the status of your jobs, as well as inspect, trigger, and debug your DAGs and tasks. It also gives a database interface and lets you read logs from the remote file store.
- **Scheduler**: The Scheduler is a component that monitors and manages all your tasks and DAGs, it checks their status and triggers them in the correct order once their dependencies are complete.
- **Executors**: It handles running your task when they are assigned by the scheduler. It can either run the tasks inside the scheduler or push task execution out to workers. Airflow supports a variety of different executors which you can choose between.
- **Metadata database**: The metadata database is used by the executor, webserver, and scheduler to store state.

![graph](img/graph.png)

## Installing Apache Airflow on OpenShift

Airflow can be run as a pip package, through docker, or a Helm chart.  
The official Helm chart can be found here: [https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html#using-official-airflow-helm-chart](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html#using-official-airflow-helm-chart) 

Here is a modified version of the Helm chart which can be installed on OpenShift 4.12: [https://github.com/eformat/openshift-airflow](https://github.com/eformat/openshift-airflow)