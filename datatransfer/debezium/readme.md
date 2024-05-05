
Strimzi: Apahttps://github.com/strimzi/strimzi-kafka-operator/blob/main/helm-charts/helm3/strimzi-kafka-operator/README.md

Apche Kafka on Kubernetes

Strimzi provides a way to run an Apache Kafka® cluster on Kubernetes or OpenShift in various deployment configurations. See our website for more details about the project.

!!! IMPORTANT !!! Upgrading to Strimzi 0.32 and newer directly from Strimzi 0.22 and earlier is no longer possible. Please follow the documentation for more details.

!!! IMPORTANT !!! From Strimzi 0.40 on, we support only Kubernetes 1.23 and newer. Kubernetes versions 1.21 and 1.22 are no longer supported.

Introduction

This chart bootstraps the Strimzi Cluster Operator Deployment, Cluster Roles, Cluster Role Bindings, Service Accounts, and Custom Resource Definitions for running Apache Kafka on Kubernetes cluster using the Helm package manager.

Supported Features

Manages the Kafka Cluster - Deploys and manages all of the components of this complex application, including dependencies like Apache ZooKeeper® that are traditionally hard to administer.
Includes Kafka Connect - Allows for configuration of common data sources and sinks to move data into and out of the Kafka cluster.
Topic Management - Creates and manages Kafka Topics within the cluster.
User Management - Creates and manages Kafka Users within the cluster.
Connector Management - Creates and manages Kafka Connect connectors.
Includes Kafka Mirror Maker 1 and 2 - Allows for mirroring data between different Apache Kafka® clusters.
Includes HTTP Kafka Bridge - Allows clients to send and receive messages through an Apache Kafka® cluster via the HTTP protocol.
Includes Cruise Control - Automates the process of balancing partitions across an Apache Kafka® cluster.
Prometheus monitoring - Built-in support for monitoring using Prometheus.
Grafana Dashboards - Built-in support for loading Grafana® dashboards via the grafana_sidecar
Upgrading your Clusters

To upgrade the Strimzi operator, you can use the helm upgrade command. The helm upgrade command does not upgrade the Custom Resource Definitions. Install the new CRDs manually after upgrading the Cluster Operator. You can access the CRDs from our GitHub release page or find them in the crd subdirectory inside the Helm Chart.

The Strimzi Operator understands how to run and upgrade between a set of Kafka versions. When specifying a new version in your config, check to make sure you aren't using any features that may have been removed. See the upgrade guide for more information.

Documentation

Documentation to all releases can be found on our website.

Getting help

If you encounter any issues while using Strimzi, you can get help using:

Strimzi mailing list on CNCF
Strimzi Slack channel on CNCF workspace
GitHub Discussions
License

Strimzi is licensed under the Apache License, Version 2.0.