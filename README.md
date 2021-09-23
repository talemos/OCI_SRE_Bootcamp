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
## Network Config



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
