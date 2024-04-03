# DEPLOYING AN HIBRID INFRASTRUCTURE FOR RESEARCHERS IN AWS

## Table of Contents

[1) INTRODUCTION](#introduction)	
[2) CREATION OF KEY PAIRS](#creation-of-key-pairs)	
[3) CREATION OF A VPC](#creation-of-a-vpc)	
[4) SUBNETS](#subnets)
[5) INTERNET GATEWAY](#internet-gateway)
[6) ROUTE TABLES](#route-tables)
[7) EC2 INSTANCES](#ec2-instances)
[8) S3 BUCKET](#s3-bucket)
[9) OPENVPN](#openvpn)
[10) NGINX CONFIGURATION](#nginx-configuration)
[11) JUPYTER NOTEBOOK](#jupyter-notebook)
[12) AUTOMATIC SYNCHRONIZATION WITH CRON](#automatic-synchronization-with-cron)



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

##### Figure 2. Key pairs created.

After generating the public and private keys using the "ed25519" algorithm, the next step is to incorporate the public key into AWS. Importing the public key into AWS allows for secure authentication and authorization mechanisms within your AWS environment. By associating the public key with your AWS resources, you establish a trusted identity for accessing those resources. This is particularly important for tasks such as SSH authentication, where the public key is used to verify the identity of users attempting to access AWS instances securely.

<img width="207" alt="Picture 3" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/b7931384-5be3-4b98-8ed2-74e1d7c5fe4d">

##### Figure 3. Public key pair imported in AWS.

## 3) CREATION OF A VPC

The creation of a VPC is the foundational step in setting up a network environment, as it serves as an isolated section in AWS where you can deploy your resources in a virtual network defined by the user. In our case, we assigned the following name to our VPC, which has been HOL02-VPC. Additionally, we have also defined the CIDR, which defines the range of IP addresses that can be used within the VPC. In our case, we have defined it as 10.0.0.0/16. It consists of an IP address and a suffix indicating the number of bits allocated for the network portion of the address. 

The steps followed to create the VPC with these specifications, user needs to navigate to the AWS Management Console, select the VPC service, and then follow these steps:
1. Go to the VPC dashboard in the AWS Management Console.
2. Click on the "Create VPC" button.
3. Enter the name "HOL02-VPC" in the appropriate field.
4. Specify the CIDR block as "10.0.0.0/16" in the CIDR block field.
5. Click on the "Create" button to create the VPC.

Once created, the VPC provides a virtual networking environment where you can launch AWS resources such as EC2 instances, among others, while maintaining control over the network configuration and security settings.

<img width="296" alt="Picture 4" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/52e4140f-ff46-444e-b003-c1c2c692e772">

##### Figure 4. Steps to create a VPC.

## 4) SUBNETS

Following the establishment of our VPC, the next step involves the segmentation of this overarching network into smaller, more manageable units known as subnets. Each subnet serves as a distinct segment of the VPC's IP address space, contributing significantly to the organization and fortification of our network infrastructure. In this endeavor, we meticulously craft three subnets within our VPC:

1. **HOL02-DMZ Subnet:** This subnet is meticulously structured with the CIDR block "10.0.1.0/24". It assumes a pivotal role in our network architecture by providing an intermediary layer of security between our internal network and external entities. Acting as a controlled gateway, it enables selective exposure of certain services to external access while maintaining a secure level of isolation from the broader internet landscape.

2. **HOL02-Production Subnet:** Named HOL02-Production, this subnet is designated with the CIDR block "10.0.2.0/24". It serves as an exclusive enclave for hosting production-related resources. Within this subnet, we deploy critical instances and services vital for our operational infrastructure, ensuring their protection within a secure environment.

3. **HOL02-Research Subnet:** Tailored for research-related endeavors, HOL02-Research is identified by the CIDR block "10.0.3.0/24". This subnet is purposefully crafted to accommodate research activities and resources. By housing these resources within a dedicated subnet, we enforce customized security measures and access controls, thereby safeguarding sensitive research data.

To actualize the creation of these subnets:

- Navigate to the VPC service in the AWS Management Console.
- Select Subnets.
- Click on Create subnet.
- Fill in the form with the following settings:
  - Name: HOL02-DMZ Subnet (or HOL02-Production Subnet, HOL02-Research Subnet for their respective creations).
  - VPC: Select the VPC created (HOL02-VPC).
  - Availability Zone
  - IPv4 CIDR block: Enter the respective CIDR block for each subnet (10.0.1.0/24 for DMZ, 10.0.2.0/24 for Production, 10.0.3.0/24 for Research).

Through these meticulously executed steps, we ensure the creation of a robust network infrastructure, fortified by carefully crafted subnets, each serving a distinct purpose in safeguarding our data and facilitating efficient operations within the AWS environment.

<img width="255" alt="Picture 5" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/3e17991a-12b9-4885-ac44-5731343f57e1">
##### Figure 5. HOL02-DMZ subnet creation.
<img width="255" alt="Picture 6" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/43a13050-aee9-486f-a2b9-edcf7fdb6f10">
##### Figure 6. HOL02-Production subnet creation.
<img width="255" alt="Picture 7" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/cc9d95e0-9dc5-4526-b9eb-b44e7bcf75aa">
##### Figure 7. HOL02-Research subnet creation.


## 5) INTERNET GATEWAY

As we move forward in setting up our network, an important part we add is the Internet Gateway. This is like a bridge that helps our EC2 instances connect to the internet. By setting up the Internet Gateway, we're making sure that our instances can easily communicate with external services and access online resources.
<img width="241" alt="Picture 8" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/2a53b99b-8a9e-448b-bca5-844fde9cb9f1">
##### Figure 8. Creation of internet gateway.

We connect the Internet Gateway (HOL02-IGW) to the VPC that we established earlier. By attaching the Internet Gateway to our VPC, we enable our VPC to serve as a gateway for outbound and inbound internet traffic. This means that any data leaving our VPC destined for the internet, or incoming from the internet into our VPC, can now flow through the Internet Gateway seamlessly.
<img width="335" alt="Picture 9" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/84b87527-c380-4906-9423-e6fe957f4350">
##### Figure 9. Internet gateway attached to the VPC.

## 6) ROUTE TABLES

To enable communication from our subnets to the internet, we configure routing tables. These tables dictate how traffic from each subnet is directed, including traffic destined for the internet.

We create three routing tables, each associated with its respective subnet:

1. HOL02-DMZ-RT: Associated with the HOL02-DMZ subnet.
2. HOL02-Production-RT: Associated with the HOL02-Production subnet.
3. HOL02-Research-RT: Associated with the HOL02-Research subnet.

These routing tables ensure efficient traffic management within each subnet and enable connectivity to external destinations, including the internet.

<img width="342" alt="Picture 10" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/5f96fc52-86c1-425f-92a7-fb6ae0e575c6">
##### Figure 10. DMZ routing table creation

To complete the setup, we must associate each routing table with its corresponding subnet. We navigate to subnet associations and select the appropriate subnet for each routing table to establish the connection.

<img width="476" alt="Picture 11" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/a9de6fea-d1a8-493c-b367-a229b4b1f3e7">
##### Figure 11. Subnet associated to the corresponding routing table.

We repeat the process for the routing table for production:
<img width="296" alt="Picture 12" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/50858216-2473-4d29-8d98-1d10b679b198">
##### Figure 12. Production routing table creation

<img width="452" alt="Picture 13" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/a41b0e81-7ec5-42f3-898f-e362c1b5af88">
##### Figure 13. Subnet associated to the corresponding routing table.

<img width="487" alt="Picture 14" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/58a5b6e2-aa30-4610-806d-378c45809078">
##### Figure 14. Production routing table associated to the internet.

<img width="357" alt="Picture 15" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/f8cb9ed6-0989-495c-800a-48fcd6330fa5">
##### Figure 15. Research routing table created.

<img width="452" alt="Picture 16" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/23a4a266-4b02-45c1-addf-5cd7178929e6">
##### Figure 16. Resource map obtained
