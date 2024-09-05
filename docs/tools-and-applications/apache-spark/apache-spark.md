# Apache Spark

## What is it?

![Spark](img/spark-logo.png){ align=left .noborder}**[Apache Spark](https://spark.apache.org/){:target="_blank"}** is an open-source, **distributed computing system** used for big data processing. It can process **large amounts of data** quickly and efficiently, and handle both **batch and streaming** data. Spark uses the in-memory computing concept, which allows it to process data much faster than traditional disk-based systems.

Spark supports a wide range of programming languages including **Java, Python, and Scala**. It provides a number of high-level **libraries and APIs**, such as Spark SQL, Spark Streaming, and MLlib, that make it easy for developers to perform **complex data processing tasks**. Spark SQL allows for querying structured data using SQL and the DataFrame API, Spark Streaming allows for processing real-time data streams, and MLlib is a machine learning library for building and deploying machine learning models. Spark also supports graph processing and graph computation through GraphX and GraphFrames.

## Working with Spark on OpenShift

Spark can be fully containerized. Therefore a standalone Spark cluster can of course be installed on OpenShift. However, it sorts of breaks the cloud-native approach brought by Kubernetes of ephemeral workloads. There are in fact many ways to work with Spark on OpenShift, either with Spark-on-Kubernetes operator, or directly through PySpark or spark-submit commands.

In this **[Spark on OpenShift](https://github.com/opendatahub-io-contrib/spark-on-openshift){:target="_blank"}** repository, you will find all the instructions to work with Spark on OpenShift.

It includes:

- pre-built UBI-based Spark images including the drivers to work with S3 storage,
- instructions and examples to build your own images (to include your own libraries for example),
- instructions to deploy the Spark history server to gather your processing logs,
- instructions to deploy the Spark on Kubernetes operator,
- Prometheus and Grafana configuration to monitor your data processing and operator in real time,
- instructions to work without the operator, from a Notebook or a Terminal, inside or outside the OpenShift Cluster,
- various examples to test your installation and the different methods.
