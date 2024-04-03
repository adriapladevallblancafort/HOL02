# DEPLOYING AN HIBRID INFRASTRUCTURE FOR RESEARCHERS IN AWS

## Table of Contents

## 1) [INTRODUCTION](#introduction)

## 2) [CREATION OF KEY PAIRS](#creation-of-key-pairs)

## 3) [CREATION OF A VPC](#creation-of-a-vpc)

## 4) [SUBNETS](#subnets)

## 5) [INTERNET GATEWAY](#internet-gateway)

## 6) [ROUTE TABLES](#route-tables)

## 7) [EC2 INSTANCES](#ec2-instances)

## 8) [S3 BUCKET](#s3-bucket)

## 9) [OPENVPN](#openvpn)

## 10) [NGINX CONFIGURATION](#nginx-configuration)

## 11) [JUPYTER NOTEBOOK](#jupyter-notebook)

## 12) [AUTOMATIC SYNCHRONIZATION WITH CRON](#automatic-synchronization-with-cron)




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

## 7) EC2 INSTANCES
Now, we proceed with the deployment of instances within our AWS environment. To initiate this process, we navigate to the AWS Management Console, where we will meticulously configure each instance:

1. Naming and Image Selection:
   - We begin by assigning a distinctive name to each instance and selecting the appropriate Amazon Machine Image (AMI) tailored to our requirements.
   - For instance, we designate an EC2 instance named HOL02-VPN, intended for deployment in the HOL02-DMZ subnet. This instance utilizes the Ubuntu 22.04 LTS AMI, ensuring compatibility and reliability for our purposes.
   - Similarly, we designate HOL02-VoilaServer for deployment in the HOL02-Production subnet, utilizing the Amazon Linux 2 AMI.
   - Lastly, HOL02-Jupyter is designated for deployment in the HOL02-Research subnet, leveraging the Amazon Linux 2 AMI.

2. Security Group Configuration:
   - With the network and subnet selections finalized, we proceed to create a security group for each instance, incorporating specific port configurations to enforce access controls.
   - For HOL02-DMZ-SG, utilized by instances within the DMZ subnet, ports 22 (SSH), 443 (HTTPS), 943 (TCP), 945 (TCP), and 1194 (UDP) are opened to facilitate necessary communication.
   - HOL02-Production-SG, associated with instances in the Production subnet, permits inbound traffic on ports 80 (HTTP), 443 (HTTPS), and 22 (SSH) restricted solely from HOL02-DMZ-SG, ensuring a controlled access pathway.
   - Similarly, HOL02-Research-SG, deployed for instances in the Research subnet, allows inbound traffic on ports 22 (SSH) and 8888 (TCP) limited exclusively from HOL02-DMZ-SG, thereby reinforcing security measures tailored to research-related activities.

In this AWS infrastructure setup, security is managed through Security Groups, which act as virtual firewalls controlling inbound and outbound traffic to EC2 instances. The HOL02-DMZ-SG is specifically designated for the DMZ subnet. This subnet typically hosts resources accessible from the internet but separated from the internal network. The security group allows inbound traffic on various ports including 22 (SSH), 443 (HTTPS), 943, 945 (TCP), and 1194 (UDP), ensuring necessary communication channels for services within the DMZ.

For the Production subnet, the HOL02-Production-SG is employed. This security group is tailored for resources hosting public-facing services. It permits inbound traffic on ports 80 (HTTP), 443 (HTTPS), and 22 (SSH). However, SSH access is restricted to incoming connections only from HOL02-DMZ-SG, ensuring that SSH connections originate only from resources within the DMZ subnet, thus enhancing security.

Similarly, the HOL02-Research-SG caters to resources within the Research subnet, possibly containing sensitive data or experimental setups. It allows inbound traffic on ports 22 (SSH) and 8888 (TCP), but access is restricted to connections originating from HOL02-DMZ-SG. This setup ensures that SSH and other services are accessible only from resources within the DMZ subnet, bolstering the overall security posture.

Among the EC2 instances, HOL02-VPN plays a crucial role as a VPN gateway, residing in the DMZ subnet. It facilitates secure communication between different subnets within the VPC. Additionally, it is allocated with an Elastic IP, ensuring consistent external access despite instance restarts or changes in public IP addresses.

On the other hand, HOL02-VoilaServer and HOL02-Jupyter represent instances in the Production and Research subnets, respectively. HOL02-VoilaServer likely hosts services accessible to the public internet, while HOL02-Jupyter may host resources or experiments requiring restricted access, both contributing to the overall functionality and security of the infrastructure.

For HOL02 VPN:

<img width="452" alt="Picture 17" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/e05f6d8b-a3e4-4e0c-bd67-45fae0ae9de5">

##### Figure 17. HOL02 VPN instance creation

<img width="452" alt="Picture 18" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/9d1ba546-3e15-4b1e-ab31-d956756d2871">

##### Figure 18. HOL02 VPN instance creation

<img width="452" alt="Picture 19" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/68c20d95-d1e1-4c5b-91d7-8d46020294fc">

##### Figure 19. HOL02 VPN instance creation

<img width="452" alt="Picture 20" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/814cd1d0-5449-4fd2-8a9e-a0d3ac8e3957">

##### Figure 20. HOL02 VPN instance creation

An Elastic IP address is a static IPv4 address that can be attached to an instance in your AWS environment. It provides a persistent public IP address that remains fixed even if the associated instance is stopped and restarted. Currently, our instance is assigned a public IP address (e.g., 44.204.119.113), but it doesn't have an Elastic IP address assigned to it. This means that if the instance is stopped and started again, the public IP address may change, requiring us to update our configurations to reflect the new address.

To solve this issue, we allocate an Elastic IP address from the AWS Elastic IP pool. Once allocated, we associate this Elastic IP address with the specific resource we want to assign it to. In this case, we choose to associate the Elastic IP address with the instance named HOL02-VPN.

<img width="319" alt="Picture 21" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/5fc0cf09-afce-4703-af4f-179f014c90e7">

##### Figure 21. Elastic IP address allocation

<img width="307" alt="Picture 22" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/eef7c926-998f-49a0-804d-be8259847b9c">

##### Figure 22. Elastic IP address association

For HOL02 Voila server:

<img width="452" alt="Picture 23" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/207b57e2-ac77-4cad-b6e4-61d4cd916396">

##### Figure 23. HOL02 Voila instance creation

<img width="452" alt="Picture 24" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/670e18ab-da0d-4f6a-ba62-d2fbe2f47358">

##### Figure 24. HOL02 Voila instance creation

<img width="452" alt="Picture 25" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/13952f1b-16c8-40e9-84a5-6eae8e5824e3">

##### Figure 25. HOL02 Voila instance creation

For HOL02 Research:

<img width="452" alt="Picture 26" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/7f74f5ec-6b2c-4988-9200-ebe430142b5a">

##### Figure 26. HOL02 research instance creation

<img width="452" alt="Picture 27" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/5e287bcf-26db-4cfe-89cd-cbb4d0c575fb">

##### Figure 27. HOL02 research instance creation

<img width="452" alt="Picture 28" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/fcef7df8-1cc4-4d18-94fb-47b4a40eddc3">

##### Figure 28. HOL02 research instance creation

## 8) S3 BUCKET
The HOL02-Notebooks S3 bucket is designated to store all notebooks used by the research team. It serves as a centralized repository for their computational work, facilitating collaboration, version control, and easy sharing among team members within the AWS infrastructure.

<img width="362" alt="Picture 29" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/3eb8f15d-80f1-4b72-9115-0c3965a33370">

##### Figure 29. Bucket creation.

<img width="452" alt="Picture 30" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/dda555d8-ce2b-4be7-a91f-df6cbd77370d">

##### Figure 30. Bucket created.

## 9) OPENVPN
The following commands are used to install OpenVPN on the EC2 instance named HOL02-VPN:

1. `sudo apt update -y`: This command updates the package lists for available software packages from the repositories defined in the system.
2. `sudo apt install ca-certificates gnupg wget net-tools -y`: This command installs the necessary packages required for the OpenVPN installation. These packages include ca-certificates (certificate authority certificates), gnupg (GNU Privacy Guard for encryption and signing), wget (utility for downloading files from the internet), and net-tools (collection of network configuration and monitoring tools). The "-y" flag again answers "yes" to any prompts during the installation process.
3. `sudo wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repo-public.asc`: This command downloads the OpenVPN Access Server public GPG key from the specified URL and saves it as a trusted key file in the "/etc/apt/trusted.gpg.d/" directory. This key is used to authenticate the OpenVPN packages during installation.
4. `sudo echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repo-public.asc] http://as-repository.openvpn.net/as/debian jammy main" | sudo tee /etc/apt/sources.list.d/openvpn-as-repo.list`: This command adds the OpenVPN Access Server repository to the system's package sources list. It appends a new entry with the repository URL and specifies the GPG key file for package verification.
5. `sudo apt update && sudo apt install openvpn-as -y`: This command updates the package lists again to include the newly added OpenVPN repository. Then, it proceeds to install the OpenVPN Access Server package ("openvpn-as") using the "-y" flag to automatically confirm the installation without user interaction.

Overall, these commands ensure that the necessary dependencies are installed, and that OpenVPN Access Server is set up correctly on the HOL02-VPN EC2 instance.

<img width="452" alt="Picture 31" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/5d404dcd-4f34-44fe-be2c-325d99dc095b">

##### Figure 31. Openvpn installation

<img width="452" alt="Picture 32" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/de7c8e45-74c8-4bd7-bcb6-edd2b6aa9e12">

##### Figure 32. Openvpn installation

<img width="452" alt="Picture 33" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/7440d162-5ff4-45f4-8757-9fc38a68252b">

##### Figure 33. Openvpn installation

<img width="452" alt="Picture 34" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/75ce2988-0d09-450d-a097-d2845fe129f6">

##### Figure 34. Openvpn installation

<img width="452" alt="Picture 35" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/1b2c6eb0-3bc0-459e-bc31-8e7fb7f4c623">

After installing VPN, then we need to configure it:

1. **Accessing OpenVPN Web Interface:** Start by navigating to the public IP address of the HOL02-VPN EC2 instance in your web browser. Log in using the provided username "openvpn" and the password displayed in the terminal during the installation process. The URL to access the OpenVPN web interface should be in the format `https://<elastic-ip>:943/admin`.

2. **Changing Hostname:** Once logged in to the OpenVPN web interface, navigate to the Network Settings section. Here, you will find an option to change the hostname to the elastic IP address of the EC2 instance. After making this change, save the settings and reload the page to apply them.

3. **Configuring VPN Settings:** After reloading the page, log in again if required. Then, navigate to the VPN Settings section. Here, add the CIDR (Classless Inter-Domain Routing) subnets one per line. Additionally, ensure to set the option "Should client Internet traffic be routed through the VPN?" to "No". This configuration prevents routing all client internet traffic through the VPN. After making these changes, save the settings and reload the page to apply them.

4. **Installing OpenVPN Client:** To allow users to connect to the VPN, provide instructions for installing the OpenVPN client on their local machines. Include the URL to the OpenVPN web interface along with the path to download the user-locked profile. Users can follow the provided instructions to install the client and download the necessary configuration files without the need for configuring additional users on the OpenVPN server.

By following these steps, the OpenVPN server is configured and ready for use, allowing authorized users to securely connect to the VPN using the provided client software and configuration profiles.

### Terminal Output:

`Access Server 2.13.1 has been successfully installed in /usr/local/openvpn_as
Configuration log file has been written to /usr/local/openvpn_as/init.log.`

`Access Server Web UIs are available here:
Admin UI: https://10.0.1.209:943/admin
Client UI: https://10.0.1.209:943/
To login please use the "openvpn" account with "O9elW3egjmw3" password.
(password can be changed on Admin UI)`

<img width="452" alt="Picture 35" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/242dd727-8b15-4a26-a813-5935df265931">

##### Figure 35. OPENVPN configuration

<img width="452" alt="Picture 36" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/0b0c7d85-64d3-40dc-a67f-35105a05ba46">

##### Figure 36. Openvpn configuration

<img width="452" alt="Picture 37" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/9536f1b8-0ab3-4c09-8e8a-f3e4361b0d80">

##### Figure 37. Openvpn configuration

<img width="452" alt="Picture 38" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/b7db654a-459e-4c9b-8047-9b666bd9adcf">

##### Figure 38. Openvpn configuration

<img width="452" alt="Picture 39" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/d7c17471-d94e-42f7-a277-de1e8438b8c7">

##### Figure 39. Openvpn installed.

<img width="452" alt="Picture 40" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/d95e61e5-1307-499c-96de-3a8b573a4a5c">

##### Figure 40. Openvpn installed.

### Description of Issues Encountered:

1. **Accessing OpenVPN Web Interface:**
   - I had to ensure that the security group associated with the HOL02-VPN EC2 instance allowed inbound traffic on port 943, the default port for OpenVPN's web interface. This involved adjusting the security group settings to permit access.
   - It was crucial to verify that the public IP address of the EC2 instance was correctly associated with the elastic IP and accessible from my network. This required confirming the IP association and checking network connectivity.
   - Double-checking the username and password provided during the installation process was necessary to ensure they were entered correctly. This involved reviewing the installation logs and ensuring accurate credentials were used.
   - It was important to verify that the elastic IP address was correctly set as the hostname to avoid connectivity issues. This involved confirming the hostname configuration within the OpenVPN settings.

2. **Configuring VPN Settings:**
   - When adding CIDR subnets to the VPN settings, I had to be cautious to avoid misconfigurations that could lead to routing issues. This involved carefully entering the CIDR subnets and reviewing the configuration.
   - Ensuring that the option "Should client Internet traffic be routed through the VPN?" was set correctly was crucial to avoid unintended routing of internet traffic. This required checking and adjusting the VPN settings accordingly.

## 10) NGINX CONFIGURATION

To set up Nginx on the EC2 instance named HOL02-VoilaServer, the following steps are needed: 

1. **Install Nginx:**
   - In our case, for Amazon Linux 2:
     - `sudo amazon-linux-extras install nginx1 -y`: This command installs Nginx on your EC2 instance using the Amazon Linux Extras repository. The `-y` flag automatically confirms the installation without user prompts.
     - `sudo systemctl start nginx`: This command starts the Nginx service immediately.
     - `sudo systemctl enable nginx`: This command enables Nginx to start automatically whenever the system boots up.

2. **Assumption:**
   - For simplification purposes, assume that the default page served by Nginx represents the Voila server.

After installing Nginx on your Amazon Linux 2 instance, we want to access to the Nginx web server to confirm it runs correctly. To do this, you'll need to use the public IP address or domain name of your instance, along with the default port for HTTP traffic, which is port 80.

#### Accessing the Nginx Web Server:
1. **Open a Web Browser:** Launch your preferred web browser on your local machine.
2. **Enter the URL:** In the address bar of your web browser, type the following URL:
   - `http://<public_ip>:80`, with the public IP being your instance IP. 

By following these steps, web browser connects to the Nginx web server running on the Voila instance. If everything is set up correctly, you should see the default Nginx welcome page displayed in your web browser.

<img width="452" alt="Picture 41" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/b9f1f5fb-5b55-4a1f-bb12-6df34a616981">

##### Figure 41. Nginx configuration

<img width="452" alt="Picture 42" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/730a3a55-d447-4f39-bf3f-76adc48ed420">

##### Figure 42. Nginx configuration

<img width="452" alt="Picture 43" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/067c8102-f29d-45c5-a7a6-2239e295b43e">

##### Figure 43. Nginx configuration

<img width="452" alt="Picture 44" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/c3015b5e-0643-40ec-b2ad-7eb13532bbca">

##### Figure 44. Nginx server

## 11) JUPYTER NOTEBOOK

To access to the HOL02-jupyter instance, which is only accessible via VPN, we need to ensure that VPN client is connected. Then, we use the private IP address of the instance to connect to it, instead of the public IP address. So, we use the following command to access to the jupyter instance: 

```bash
ssh -i aws-adria ec2-user@10.0.3.38
```
<img width="348" alt="Picture 45" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/7d8a6a92-8ac7-4261-b80e-ca9ea792503a">

##### Figure 45. HOL02-jupyter instance connection

We need to connect the instance to the internet gateway to update the machine to be ready for the use. We first do sudo yum update -y. Next step is to install python. We create a script name install.sh and we run it.

<img width="452" alt="Picture 46" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/002f0346-dfb2-490d-b766-85d677183ae3">

##### Figure 46. Instance updated.

<img width="452" alt="Picture 47" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/437f2f68-25f6-4d6a-8a78-cc447d3558ce">

##### Figure 47. Python installer script.

So, to set up and run a jupyter notebook on the HOL02-Jupyter instance, we create a python virtual environment named jupyter-env. This environment ensures isolation and facilitates the installation of jupyter notebook with its dependencies. Then the virtual environment is activated to initiate the jupyter notebook server.

`[root@ip-10-0-3-135 ec2-user]# python3 -m venv jupyter-env
[root@ip-10-0-3-135 ec2-user]# source jupyter-env/bin/activate
(jupyter-env) [root@ip-10-0-3-135 ec2-user]#`

During the execution process, a warning message appeared to me on several occasions: "Running as root is not recommended." This warning highlights the potential security risks associated with running services as the root user. To address this concern, several mitigation strategies are proposed. Firstly, creating a dedicated non-root user specifically for running Jupyter Notebook can enhance security by reducing the privileges associated with the service. Alternatively, utilizing the --allow-root flag with the jupyter notebook command bypasses the warning, although it's advised against unless the risks are fully understood.

<img width="452" alt="Picture 48" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/c3903d7c-baae-483e-9851-60047cfb87a3">

##### Figure 48. Jupyter environment created

Now we need to execute a jupyter notebook server within a virtual environment named ‘jupyter-env’. The process begins with the command to initiate the Jupyter Notebook server using the nohup command, which allows the process to continue running even after the terminal is closed. The --ip=0.0.0.0 flag specifies that the server should listen on all available network interfaces, and --notebook-dir=/home/ec2-user/notebooks sets the directory where notebooks are stored. The --no-browser flag indicates that the server should not attempt to open a web browser upon start up, and --allow-root permits running the server as the root user. The output is redirected to a log file located at /home/ec2-user/notebooks/jupyter.log. 

`(jupyter-env) [root@ip-10-0-3-135 notebooks]# nohup jupyter notebook --ip=0.0.0.0 --notebook-dir=/home/ec2-user/notebooks --no-browser --allow-root > /home/ec2-user/notebooks/jupyter.log 2>&1 &
[1] 23009`

If we look at the log display, we can observe the successful start-up of the Jupyter Notebook server, along with the loading of various extensions and components. It confirms the successful linking and loading of extensions such as jupyter_lsp, jupyter_server_terminals, and jupyterlab. Additionally, it provides information about the JupyterLab extension directory and manager.

Further details include the local directory from which notebooks are served (/home/ec2-user/notebooks), the version of the Jupyter Server running, and the URLs at which the server can be accessed. Notably, the URLs include both the internal EC2 instance IP address (http://ip-10-0-3-135.ec2.internal:8888) and localhost (http://127.0.0.1:8888). So, when accessing the jupyter notebook via a web browser, it is important to use the private IP address. This private IP ensures secure communication within the internal network. The format is http:// followed by the private IP address of the EC2 instance and the port number: ‘8888’.

The following output provides two methods for accessing the Jupyter Notebook server. The first method suggests opening a specific file in a web browser. The file path (file:///root/.local/share/jupyter/runtime/jpserver-23009-open.html) indicates the location of a local HTML file generated by the Jupyter server. Opening this file in a browser will likely trigger a redirect or provide a link to access the Jupyter server. Alternatively, the second method provides URLs that can be copied and pasted into a web browser's address bar. These URLs are links to the Jupyter server, each containing a token parameter (token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc). The token serves as a form of authentication to access the server securely.

The first URL (http://ip-10-0-3-135.ec2.internal:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc) points to the Jupyter server using the EC2 instance's internal IP address (ip-10-0-3-135.ec2.internal) and port 8888. This URL can be used to access the server from within the EC2 instance's network.
The second URL (http://127.0.0.1:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc) uses the loopback address (127.0.0.1) to access the Jupyter server from the local machine on which the server is running. 


`(jupyter-env) [root@ip-10-0-3-135 notebooks]# ls
Python-3.9.7  Python-3.9.7.tgz  data.csv  install.sh  jupyter.log  myscript.py  script.ipynb
(jupyter-env) [root@ip-10-0-3-135 notebooks]# cat jupyter.log
nohup: ignoring input
[I 2024-04-01 17:38:31.743 ServerApp] jupyter_lsp | extension was successfully linked.
[I 2024-04-01 17:38:31.749 ServerApp] jupyter_server_terminals | extension was successfully linked.
[I 2024-04-01 17:38:31.756 ServerApp] jupyterlab | extension was successfully linked.
[I 2024-04-01 17:38:31.762 ServerApp] notebook | extension was successfully linked.
[I 2024-04-01 17:38:32.035 ServerApp] notebook_shim | extension was successfully linked.
[I 2024-04-01 17:38:32.058 ServerApp] notebook_shim | extension was successfully loaded.
[I 2024-04-01 17:38:32.062 ServerApp] jupyter_lsp | extension was successfully loaded.
[I 2024-04-01 17:38:32.063 ServerApp] jupyter_server_terminals | extension was successfully loaded.
[I 2024-04-01 17:38:32.066 LabApp] JupyterLab extension loaded from /home/ec2-user/jupyter-env/lib/python3.9/site-packages/jupyterlab
[I 2024-04-01 17:38:32.066 LabApp] JupyterLab application directory is /home/ec2-user/jupyter-env/share/jupyter/lab
[I 2024-04-01 17:38:32.066 LabApp] Extension Manager is 'pypi'.
[I 2024-04-01 17:38:32.081 ServerApp] jupyterlab | extension was successfully loaded.
[I 2024-04-01 17:38:32.085 ServerApp] notebook | extension was successfully loaded.
[I 2024-04-01 17:38:32.085 ServerApp] Serving notebooks from local directory: /home/ec2-user/notebooks
[I 2024-04-01 17:38:32.086 ServerApp] Jupyter Server 2.13.0 is running at:
[I 2024-04-01 17:38:32.086 ServerApp] http://ip-10-0-3-135.ec2.internal:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc
[I 2024-04-01 17:38:32.086 ServerApp]     http://127.0.0.1:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc
[I 2024-04-01 17:38:32.086 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 2024-04-01 17:38:32.090 ServerApp] `
    
    To access the server, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/jpserver-23009-open.html
    Or copy and paste one of these URLs:
        http://ip-10-0-3-135.ec2.internal:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc
        http://127.0.0.1:8888/tree?token=ffbec2ab9377e92a918cf63d946f4bb8999929847a1040bc
        
`[I 2024-04-01 17:38:32.113 ServerApp] Skipped non-installed server(s): bash-language-server, dockerfile-language-server-nodejs, javascript-typescript-langserver, jedi-language-server, julia-language-server, pyright, python-language-server, python-lsp-server, r-languageserver, sql-language-server, texlab, typescript-language-server, unified-language-server, vscode-css-languageserver-bin, vscode-html-languageserver-bin, vscode-json-languageserver-bin, yaml-language-server
(jupyter-env) [root@ip-10-0-3-135 notebooks]#`

<img width="452" alt="Picture 49" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/e8cda550-1e08-413c-ae44-82ad4bdfe64d">

##### Figure 49. Jupyter notebook accessible using the browser

Then, we need to set up AWS access key to connect the jupyter notebook with the bucket. With the aws configure command we set up AWS Access Key ID and Secret Access Key. Additionally, the aws configure set aws_session_token command is employed to configure the AWS session token for temporary security credentials. These credentials are essential for accessing AWS services securely from within the Jupyter Notebook environment.


`(jupyter-env) [root@ip-10-0-3-135 notebooks]# aws configure
AWS Access Key ID [****************2ZW5]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]:`

`(jupyter-env) [root@ip-10-0-3-135 notebooks]# aws configure set aws_session_token FwoGZXIvYXdzEDEaDJNSePrdq6i2KJNu9SLMAV0OxJFubBXF6HYf+p6g22+Jam40HeUI6LF0y8TAE4UCX9FFnBf67XGCJyv8EqUuTCOnacFO43qqG9cH67n4EuPl9evhC20+Mt6lMJjbRink3AChJbInQjHTwE5x/h3WVhDzngs1qGGMvoYVTxE5pwQga+VNUObspEq8uVmoCzC7XC5XKFuzFC2tUWUXykDsP2nWJ946whsobq626xWLX2w+AR10FJEPaXVfR2E4+wIKUKWMIywsTgJFGkcndbOTU2dPd2xcWJE45E8YGSifsquwBjIt6GH8xpeSEcNIw+qsr2vMvOYjd1Hi7Bp60yEPtik8Dv7RkkVqb/24q2yaQLMG`

To configure Jupyter Notebook for direct synchronization with an S3 bucket, the %config magic command is utilized within a Jupyter Notebook cell. This configuration leverages the s3contents extension, enabling seamless integration between Jupyter Notebook and AWS S3 storage. Initially, it's essential to install the s3contents library if not already installed, which can be achieved using the !pip install s3contents command. Following this, the necessary modules are imported, including S3ContentsManager from s3contents.

Once the required modules are imported, the %config command is used to set the NotebookApp.contents_manager_class to 's3contents.S3ContentsManager', indicating that Jupyter Notebook should utilize the S3ContentsManager for managing notebook contents. Additionally, the %config S3ContentsManager.bucket command is employed to specify the name of the S3 bucket to which Jupyter Notebook should synchronize. Users need to replace 'your-bucket-name' with the actual name of their S3 bucket.

By executing these commands within a Jupyter Notebook cell, users configure Jupyter Notebook to synchronize with the specified S3 bucket using the s3contents extension. Consequently, any notebooks created or modified within the Jupyter environment will be stored in the configured S3 bucket, facilitating seamless data management and collaboration workflows.

In the subsequent code snippet, adjustments are made to the configuration to utilize the bucket named hol02-notebooks-adria. The provided Python code demonstrates the creation of a sample Jupyter Notebook, wherein a Data Frame is generated and saved to a CSV file. Subsequently, the CSV file is uploaded to the specified S3 bucket (hol02-notebooks-adria) using the boto3 library for interaction with AWS S3. Upon execution of this code within a Jupyter Notebook cell, the sample Data Frame and data are saved to the designated S3 bucket, enabling users to verify the uploaded file in their S3 bucket.

<img width="422" alt="Picture 50" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/c9977dd6-ba9a-4f4f-bb9f-0b63a1e16f4b">

##### Figure 50. Jupyter synchronization with S3 bucket

 
<img width="419" alt="Picture 51" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/66f249b3-23dc-4fd8-8731-86a18d7254a6">

##### Figure 51. Jupyter synchronization with S3 bucket

 <img width="431" alt="Picture 52" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/688226c7-9552-4d36-8917-d98f40557324">

##### Figure 52. Jupyter synchronization with S3 bucket

<img width="452" alt="Picture 53" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/d9f402a2-baa1-4759-80da-f301f0c6a93f">

##### Figure 53. Jupyter synchronization with S3 bucket

### 12) AUTOMATIC SYNCHRONIZATION WITH CRON

Now, the objective is to synchronize the EC2 instances files with the S3 bucket. This automation allows for seamless data transfer and backup. So, there is the need to create a shell script named `sync_to_s3.sh` using the `nano` text editor. This script likely contains commands to synchronize files between the EC2 instance and an S3 bucket. 

<img width="220" alt="Picture 54" src="https://github.com/adriapladevallblancafort/HOL02/assets/155838997/22c9835a-4b83-4a8b-9cef-4de0af11798e">

##### Figure 54. Synchronization script

Then, after creating the script, we make it executable with the command `chmod +x sync_to_s3.sh` command. This step grants execute permissions to the script, allowing it to be run as a program. Then to schedule the execution of the shell script at specific intervals, the user edits their crontab using the `crontab -e` command. Crontab is a time-based job scheduler in Unix-like operating systems. By configuring a cron job, the user automates the synchronization process, specifying when the script should run. So, we add a cron job entry to schedule the script to run at the desired interval. For example, we can synchronize the S3 bucket every hour with the following command, replacing `/path/to/sync_to_s3.sh` with the path to your `sync_to_s3.sh` script. This line will run the script every hour, and output will be logged to `/var/log/sync_to_s3.log`.

```bash
0 * * * * /path/to/sync_to_s3.sh >> /var/log/sync_to_s3.log 2>&1
```
After setting up the cron job, your EC2 instance will automatically synchronize the specified directory with the hol02-notebooks-adria S3 bucket every hour. You can adjust the schedule or the directory to sync as needed.


`[ec2-user@ip-10-0-2-161 ~]$ aws s3 cp s3://hol02-notebooks-adria/hol02-app.ipynb .
download: s3://hol02-notebooks-adria/hol02-app.ipynb to ./hol02-app.ipynb
[ec2-user@ip-10-0-2-161 ~]$ ls
hol02-app.ipynb  install.sh
[ec2-user@ip-10-0-2-161 ~]$
[ec2-user@ip-10-0-2-161 ~]$ nano sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ ls
hol02-app.ipynb  install.sh  sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ nano sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ nano sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ ls
hol02-app.ipynb  install.sh  sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ chmod +x sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ ls
hol02-app.ipynb  install.sh  sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$ crontab -e
crontab: installing new crontab
[ec2-user@ip-10-0-2-161 ~]$ ls
hol02-app.ipynb  install.sh  sync_to_s3.sh
[ec2-user@ip-10-0-2-161 ~]$`



