# OCI LB TrafficManagement
This repository has practical use case for LB and Global Region Traffic Management Policies

# IMPORTANT
To complete this lab, you must have an OCI account, not all services are free tier, but you should be able to complete all the lab with an active OCI trial account, if don't have one [Click Here](https://www.oracle.com/cloud/free/)

Also, to complete the Traffic Management Policies on OCI, you need to have an active DNS registration name, if don't have one, you can use [Freenom](https://www.freenom.com/)
## Summary
In this lab you are going to configure and create on OCI:
- **Network**
- **Security Lists / Network Security Groups**
- **Create Compute Instances**
- **Install and Configure HTTPD**
- **Create and Configure Load Balancers**
- **Create Public Zones**
- **Configure Public DNS Zones and Specific IPs**
- **Create an Global Traffic Management Policy**
- **Create Health Checks for your Infrastructure**
- **Put it all together to manage the network traffic of your application**
## Required tools to create this DEMO

- [**OCI TENANT**]
- [**Free Register of DNS name**]
# START
## TASK1: Generate SSH Keys
The SSH (Secure Shell) protocol is a method for secure remote login from one computer to another. SSH enables secure system administration and file transfers over insecure networks using encryption to secure the connections between endpoints. SSH keys are an important part of securely accessing Oracle Cloud Infrastructure compute instances in the cloud.

We recommend you use the [Oracle Cloud Shell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro.htm) to interface with the OCI compute instance you will create. Oracle Cloud Shell is browser-based, does not require installation or configuration of software on your laptop, and works independently of your network setup.

IMPORTANT: If the SSH key is not created correctly, you will not be able to connect to your environment and will get errors. Please ensure you create your key properly.

To start the Oracle Cloud shell, go to your Cloud console and click the cloud shell icon at the top right of the page.
<p align="center">
  <img src="./Images/CloudShell.jpg">
</p>

Once the cloud shell has started, enter the following commands. Choose the key name you can remember. This will be the keyname you will use to connect to any compute instances you create. Press Enter twice for no passphrase

```hcl
mkdir .ssh
cd .ssh
ssh-keygen -b 2048 -t rsa -f <your SSH key name>
```

Note in the output that there are two files, a private key and a public key. Keep the private key safe and don't share its content with anyone. The public key will be needed for various activities and can be uploaded to certain systems as well as copied and pasted to facilitate secure communications in the cloud.

To list the contents of the public key, use the cat command:
```hcl
cat <your SSH key name>.pub
```
Copy the contents of the public key and save it somewhere for later. When pasting the key into the compute instance in future labs, make sure that you remove any hard returns that may have been added when copying. The .pub key should be one line.

## TASK2: Create Network - VCN
From the OCI Services menu, click Virtual Cloud Networks under Networking. Select the compartment assigned to you from drop down menu on left part of the screen under Networking and Click Start VCN Wizard.
*NOTE:* Ensure the correct Compartment is selected under COMPARTMENT list

Fill out the dialog box:

```hcl
VCN NAME: Provide a name
COMPARTMENT: Ensure your compartment is selected
VCN CIDR BLOCK: Provide a CIDR block (10.0.0.0/16)
PUBLIC SUBNET CIDR BLOCK: Provide a CIDR block (10.0.1.0/24)
PRIVATE SUBNET CIDR BLOCK: Provide a CIDR block (10.0.2.0/24)
```

Click Next
Verify all the information and Click Create.

This will create a VCN with followig components:
VCN, Public subnet, Private subnet, Internet gateway (IG), NAT gateway (NAT), Service gateway (SG)
## TASK3: Create Two Compute Instance and Install Web Server
Switch to the OCI console. From OCI services menu, Click Instances under Compute.

Click Create Instance. Fill out the dialog box:

```hcl
Name your instance: Enter a name
Choose an operating system or image source: For the image, we recommend using the Latest Oracle Linux available. It is the default selection.
Availability Domain: Select availability domain
Instance Shape: Click on change shape if you want to use a different shape from the default one
Under Configure Networking
Virtual cloud network compartment: Select your compartment
Virtual cloud network: Choose the VCN you created earlier
Subnet Compartment: Choose your compartment.
Subnet: Choose the Private Subnet
Use network security groups to control traffic : Leave un-checked
Configure Boot Volume: Leave the default choices
Add SSH Keys: Choose 'Paste SSH Keys' and paste the Public Key you created in Cloud Shell earlier. Ensure when you are pasting that you paste one line
```

NOTE: The lab instruction places the instances on a private subnets. If you wish to access them, you can create the Bastion Service and create an SSH Session.

Click Create.

```hcl
CREATE USER mnocidemo IDENTIFIED BY "XXXXXXXXX";
GRANT CONNECT, RESOURCE TO mnocidemo;
GRANT UNLIMITED TABLESPACE TO mnocidemo;
```

<p align="center">
  <img src="./docs/image-1.png">
</p>

2-) Download ATP Wallet file

Next setp download the wallet for JDBC configuration. I'll use the same password from user, but it's not recommended. 
You should create your own wallet password

<p align="center">
  <img src="./docs/image-2.png">
</p>

Save the file and extract the contents of the wallet into the Micronaut Application tmp/wallet dir, with this command:

```hcl
unzip /path/to/Wallet.zip -d /tmp/wallet
```

<p align="center">
  <img src="./docs/image-3.png">
</p>

Autonomous Database have different connections pools to configure with your application, if you open the file "tnsnames.ora" you should see 3 different connection pools:
<your DB>_high
<your DB>_medium
<your DB>_low

You can pick the high for this example, but if you want to know more details about the connection pools, please see this link:
[ATP Connection Settings](https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/connect-jdbc-thin-wallet.html#GUID-1640CC02-BF3E-48C2-8FFE-A596614A6A40)

Just save this connection pool name for latter, we'll need it to configure the application.yml file latter !

3-) Now let's configure the Minonaut Application !!

First, use the quick launch micronaut site to create the sample application structure and files:
[Micronaut Launch](https://micronaut.io/launch/)

<p align="center">
  <img src="./docs/image-4.png">
</p>

Download the zip, expand on your machine and let's configure the DEMO

```hcl
micronaut:
  application:
    name: demoATP
  executors:
    io:
      type: fixed
      nThreads: 75
datasources:
  default:
    url: jdbc:oracle:thin:@mnociatp_high?tns_admin=/tmp/wallet
    driverClassName: oracle.jdbc.OracleDriver
    username: mnocidemo
    password: YOUR PASS
    dialect: ORACLE
    data-source-properties:
      oracle:
        jdbc:
          fanEnabled: false
flyway:
  datasources:
    default:
      enabled: true
```

<p align="center">
  <img src="./docs/image-5.png">
</p>

Update the password of the application.yml with the created password of the schema user created

In the Gradle build file "build.gradle" add the following dependencys:

```hcl
runtimeOnly("com.oracle.database.security:oraclepki:21.1.0.0")
runtimeOnly("com.oracle.database.security:osdt_cert:21.1.0.0")
runtimeOnly("com.oracle.database.security:osdt_core:21.1.0.0")
runtimeOnly("io.micronaut.flyway:micronaut-flyway")
```

<p align="center">
  <img src="./docs/image-6.png">
</p>

4-) Build and Test the App

[**GRADLE BUILD TESTS ERROS:**]

<p align="center">
  <img src="./docs/image-7.png">
</p>

You might get some erros during the build / test with [Gradle](https://gradle.org/)

<p align="center">
  <img src="./docs/image-8.png">
</p>


io.micronaut.context.exceptions.BeanInstantiationException: Bean definition [javax.sql.DataSource] could not be loaded: Error instantiating bean of type [javax.sql.DataSource]: Failed to initialize pool: IO Error: could not resolve the connect identifier  "<your connection pool>" (CONNECTION_ID=RkjDYM/DQG6+wHufZR2HVw==)

If that is the case, check if your wallet files are in the correct directory and check also the application.yml file configuration :)

5-) Success Running App \o/

[**BUILD SUCCESSFUL RUNNING THE APPLICATION**]

<p align="center">
  <img src="./docs/image-9.png">
</p>

Check the application:
<p align="center">
  <img src="./docs/image-10.png">
</p>

<p align="center">
  <img src="./docs/image-11.png">
</p>

Congratulations, you can now deploy the APP to OCI instance

## OPTIONAL - Build the native image
After the build and run success we can install the GraalVM package to build the native image for deployment
To get the GraalVM on your SO, go to the link and proceed with the install steps
- [Download](https://github.com/graalvm/graalvm-ce-builds/releases)
- [Install Steps](https://www.graalvm.org/docs/getting-started/#install-graalvm)

**Explanation to install native-image with gu update tool:**
- [Install Native Image gu tool](https://docs.oracle.com/en/graalvm/enterprise/19/guide/reference/native-image/native-image.html)

**Links to Oracle DOCs GraalVM 21:**
- [GraalVM Enterprise](https://docs.oracle.com/en/graalvm/enterprise/21/docs/getting-started/#install-graalvm-enterprise)
- [Getting Started](https://docs.oracle.com/en/graalvm/enterprise/21/docs/getting-started/)

## Micronaut 2.4.1 Documentation

- [User Guide](https://docs.micronaut.io/2.4.1/guide/index.html)
- [API Reference](https://docs.micronaut.io/2.4.1/api/index.html)
- [Configuration Reference](https://docs.micronaut.io/2.4.1/guide/configurationreference.html)
- [Micronaut Guides](https://guides.micronaut.io/index.html)
---

## Feature http-client documentation

- [Micronaut HTTP Client documentation](https://docs.micronaut.io/latest/guide/index.html#httpClient)
