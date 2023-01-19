
# Airflow

## What is it?
[**Airflow**](https://airflow.apache.org/) is a platform created by the community to programmatically author, schedule and monitor workflows.  
It has become popular because of the wide variety of tasks it can use for building pipelines, thanks to how extendable it is. Since it's a Python framework it has also gathered a lot of interest in the Data Science field.

The key features of airflow are:

- **Webserver**: It's a user interface where you can see the status of your jobs, as well as inspect, trigger, and debug your DAGs and tasks. It also gives a database interface and lets you read logs from the remote file store.
- **Scheduler**: The Scheduler is a component that monitors and manages all your tasks and DAGs, it checks their status and triggers them in the correct order once their dependencies are complete.
- **Executors**: It handles running your task when they are assigned by the scheduler. It can either run the tasks inside the scheduler or push task execution out to workers. Airflow supports a variety of different executors which you can choose between.
- **Metadata database**: The metadata database is used by the executor, webserver, and scheduler to store state.

![graph](img/graph.png)

## Installing Airflow on OpenShift
It can be run as a pip package, through docker, or a Helm chart.  
The official Helm chart can be found here: https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html#using-official-airflow-helm-chart 