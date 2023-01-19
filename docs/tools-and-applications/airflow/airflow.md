# Airflow

## What is it?
[Airflow](https://airflow.apache.org/) is a platform created by the community to programmatically author, schedule and monitor workflows.  
It has become popular because of the wide variety of tasks it can use for building pipelines, how extendable it is, and has gathered a lot of interest in the Data Science field thanks to being a Python framework.

The key features of airflow are:

- **Scheduler**: Monitors tasks and DAGs, triggers scheduled workflows, and submits tasks to the executor to run. It is built to run as a persistent service in the Airflow production environment.
- **Executors**: These are mechanisms that run task instances; they practically run everything in the scheduler. Executors have a common API and you can swap them based on your installation requirements. You can only have one executor configured per time.
- **Webserver**: A user interface that displays the status of your jobs and allows you to view, trigger, and debug DAGs and tasks. It also helps you to interact with the database, read logs from the remote file store.
- **Metadata database**: The metadata database is used by the executor, webserver, and scheduler to store state.

![graph](img/graph.png)

It can be run as a pip package, through docker, or a HELM chart.

## Installing Airflow on OpenShift
