# DEPLOYING AN HIBRID INFRASTRUCTURE FOR RESEARCHERS IN AWS

## Table of Contents
1. [INTRODUCTION](#introduction)	
2. [CREATION OF KEY PAIRS](#creation-of-key-pairs)	
3. [CREATION OF A VPC](#creation-of-a-vpc)	
4. [SUBNETS](#subnets)

## 1) INTRODUCTION
Figure 1 illustrates a hybrid infrastructure design proposed for deployment of this practical activity. This infrastructure is hosted on Amazon Web Services (AWS) and is represented visually to provide a clear understanding of its components and their relationships.

![Figure 1. Architecture diagram of the proposed activity](figure1.png)

At the core of this infrastructure is a Virtual Private Cloud, which serves as a virtual network environment for hosting various resources. The VPC is divided into three distinct subnets, each serving specific purposes:

1. **Production Subnet:** This subnet is designated for hosting production-related resources. Within this subnet, there is an EC2 instance named HOL02-VoilaServer. This instance is configured to be accessible over the internet, implying that it can interact with users or external systems outside of the VPC.
2. **Research Subnet:** The research subnet is allocated for research-related activities and resources. Within this subnet, there exists an EC2 instance named HOL02-Jupyter. Unlike the Production Subnet, access to this instance is restricted and is only possible through a Virtual Private Network (VPN). This security measure ensures that access to research-related resources is controlled and limited to authorized personnel.
3. **DMZ Subnet:** The DMZ Subnet acts as an intermediary between the internal network (Research Subnet) and the external network (the team's access). It hosts an EC2 instance named HOL02-VPN, which serves as a Virtual Private Network gateway. This VPN gateway facilitates secure SSH connections from the team's environment to the Research Subnet. By leveraging the VPN, the team can establish encrypted and authenticated connections to securely access resources within the Research Subnet.
