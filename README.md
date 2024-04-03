# DEPLOYING AN HIBRID INFRASTRUCTURE FOR RESEARCHERS IN AWS

## Table of Contents
1. [INTRODUCTION](#introduction)	
2. [CREATION OF KEY PAIRS](#creation-of-key-pairs)	
3. [CREATION OF A VPC](#creation-of-a-vpc)	
4. [SUBNETS](#subnets)


## 1) INTRODUCTION
Figure 1 illustrates a hybrid infrastructure design proposed for deployment of this practical activity. This infrastructure is hosted on Amazon Web Services (AWS) and is represented visually to provide a clear understanding of its components and their relationships.

![Picture 1](https://github.com/adriapladevallblancafort/HOL02/assets/155838997/2d6824fb-804d-4625-a320-cd51209aee54)

##### Figure 1. Architecture diagram of the proposed activity

At the core of this infrastructure is a Virtual Private Cloud, which serves as a virtual network environment for hosting various resources. The VPC is divided into three distinct subnets, each serving specific purposes:

1. **Production Subnet**: This subnet is designated for hosting production-related resources. Within this subnet, there is an EC2 instance named HOL02-VoilaServer. This instance is configured to be accessible over the internet, implying that it can interact with users or external systems outside of the VPC.

2. **Research Subnet**: The research subnet is allocated for research-related activities and resources. Within this subnet, there exists an EC2 instance named HOL02-Jupyter. Unlike the Production Subnet, access to this instance is restricted and is only possible through a Virtual Private Network (VPN). This security measure ensures that access to research-related resources is controlled and limited to authorized personnel.

3. **DMZ Subnet**: The DMZ Subnet acts as an intermediary between the internal network (Research Subnet) and the external network (the team's access). It hosts an EC2 instance named HOL02-VPN, which serves as a Virtual Private Network gateway. This VPN gateway facilitates secure SSH connections from the team's environment to the Research Subnet. By leveraging the VPN, the team can establish encrypted and authenticated connections to securely access resources within the Research Subnet.

   ## 2) CREATION OF KEY PAIRS

To facilitate secure communication between the clients (referred to as "us") and the server, which in this scenario is Amazon, the implementation of a robust protocol is essential. The SSH, Secure Shell protocol is chosen for its reliability and encryption capabilities. To establish this secure connection, a pair of cryptographic keys is utilized. The public key is stored on the server, while the corresponding private key resides on the client's computer. The initial step in this setup involves generating an SSH key pair. This is accomplished using the command `ssh-keygen -t ed25519`, and the result is the following one:

<img width="460" alt="Picture 2" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/09c630fa-39bd-4c88-81d9-8322bb365a55">

### Figure 2. Key pairs created.

After generating the public and private keys using the "ed25519" algorithm, the next step is to incorporate the public key into AWS. Importing the public key into AWS allows for secure authentication and authorization mechanisms within your AWS environment. By associating the public key with your AWS resources, you establish a trusted identity for accessing those resources. This is particularly important for tasks such as SSH authentication, where the public key is used to verify the identity of users attempting to access AWS instances securely.

<img width="207" alt="Picture 3" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/b7931384-5be3-4b98-8ed2-74e1d7c5fe4d">

### Figure 3. Public key pair imported in AWS.


