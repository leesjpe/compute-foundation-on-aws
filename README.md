# XC Foundation on AWS 🏗️

Prerequisites, foundational resources, and shared infrastructure guides for AWS Accelerated Computing (XC). This repository supports both **Neuron** and **GPU** workloads.

## 🎯 Objectives
* **Capacity Management:** Guides on **EC2 Capacity Blocks (ODCR)** for machine learning.
* **Network & Security:** VPC design for EFA, Security Groups, and IAM roles.
* **Quotas:** Managing and requesting Service Quotas for high-performance instances.
* **Image Builder:** creating Golden AMIs for XC workloads.

## 📂 Contents
* `/capacity-blocks`: How to reserve, purchase, and use EC2 Capacity Blocks.
* `/vpc-patterns`: CloudFormation/Terraform templates for HPC networking.
* `/iam-policies`: Least privilege policies for XC users.

## 🔗 Related Repositories
* [XC Neuron Silicon](https://github.com/your-username/xc-neuron-silicon-aws)
* [XC GPU on AWS](https://github.com/your-username/xc-gpu-on-aws)
* [ParallelCluster on AWS](https://github.com/your-username/parallel-cluster-on-aws)
