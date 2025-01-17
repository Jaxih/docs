---
title: Windows Service
description: RCL CertificateBot Windows Service for automatic SSL/TLS certificate installation and renewal in a Windows server 
parent: RCL CertificateBot
nav_order: 3
---

# RCL CertificateBot for Windows
**V6.0.10**

RCL CertificateBot runs as a **Windows Service** in a Windows Server. The Windows Service will run every seven (7) days to automatically renew and save SSL/TLS certificates from a user's subscription in the **RCL Portal** to a Windows hosting machine.

## Automatically Renew SSL/TLS Certificates

You can use RCL CertificateBot to automatically renew SSL/TLS certificates created in the **RCL Portal** using the the following creation options :

- Azure DNS (including SAN) - **Recommended**

**'Stand Alone' certificates are not supported by RCL CertificateBot.**

# Install RCL CertificateBot
## Download the Files

- The Windows Service files (``certificatebot-win-xx``) are available in the [GitHub Project](https://github.com/rcl-ssl/RCL.CertificateBot) page in the [Releases](https://github.com/rcl-ssl/RCL.CertificateBot/releases/tag/V6.0.10) section:

- Download the zip file with bitness

  - [win-x86](https://github.com/rcl-ssl/RCL.CertificateBot/releases/download/V6.0.10/certificatebot-win-x86.zip)
  - [win-x64](https://github.com/rcl-ssl/RCL.CertificateBot/releases/download/V6.0.10/certificatebot-win-x64.zip) 
  - [win-arm](https://github.com/rcl-ssl/RCL.CertificateBot/releases/download/V6.0.10/certificatebot-win-arm.zip)
  
  to match your Windows server bitness

- Extract the zip file to a folder on your server after it is downloaded

## Configure the Service

### Register an AAD Application

An Azure Active Directory (AAD) application must be registered to obtain permission to access a user's Azure resources (**DNS Zone**). 

Please refer to the following link to register an AAD application:

- [Registering an AAD Application](../authorization/aad-application)

### Set Access Control for the AAD Application

Access control must be set for the AAD application to access resources (**DNS Zone**) in a user's Azure subscription. Please refer to the following link to set access control :

- [Setting Access Control for the AAD Application](../authorization/access-control-app)

### Get the AAD Application Credentials 

To obtain the following credentials from the AAD application:

- ClientId
- ClientSecret
- TenantId

follow the instructions in this link :

- [Get the AAD Application Credentials](../authorization/aad-application#get-the-aad-application-credentials)

### Get the SubscriptionId

Get the **Subscription Id** in the RCL Portal.

![install](../images/autorenew_configure/add_subscriptionid.png)

- Scroll down and copy the 'Subscription Id' 

![install](../images/autorenew_configure/add_subscriptionid2.png)

### Register the AAD Application's ``Client Id`` in the RCL Portal

The AAD Application must be associated with a user's RCL subscription. This is achieved by registering the AAD Application's ``Client Id`` in the **RCL Portal**.

To add the AAD Application's ``Client Id`` to the portal, please follow the instructions in this link :

- [Add the Client Id in the RCL Portal](../api/authorization#add-the-client-id-in-the-rcl-portal)


### Add the Configuration variables

- In the folder containing the files for the Windows Service that you extracted, find and open the **appsettings.json** file

- Add the credentials for the AAD Application and SubscriptionId in the **RCLSDK** section :
  - ClientId
  - ClientSecret
  - TenantId
  - SubscriptionId

- In the **CertificateBot** section, set a folder path to save the SSL/TLS certificates. Recommended path : C:/ssl

  - saveCertificatePath

- **Note : when setting the folder path , use forward slashes(``/``) in the path name, eg. ``C:/ssl``**

- Create the folder in the server and ensure it has read/write permissions so that the certificates can be saved to it. 

- The ``includeCertificates`` settings will allow for including specific certificates by its name 
(eg:  "contoso.com"  or "contoso.com, *.contoso.com" - for SAN) for the certificate(s) you want to save on the server. 

- **Note: use forward slashes (/) when setting folder paths (eg: C:/ssl)**

Example
```
  "CertificateBot": {
    "saveCertificatePath": "C:/ssl",
     "includeCertificates": [
       "contoso.com",
       "fabricam.com",
       "acme.com,*.acme.com",
       "adworks.com, www.adworks.com"
       ],
  },
```

## Example of a configured **appsettings.json** file

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    },
    "EventLog": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    }
  },
  "RCLSDK": {
    "ApiBaseUrl": "https://rclapi.azure-api.net/public",
    "SourceApplication": "RCL CertificateBot Windows",
    "ClientId": "35ca82aa-9ff3-5a67-bb7f-c3c71027eecf",
    "ClientSecret": "hdytev539dgw~_8-g4lNI84V01.yIDUMHh",
    "TenantId": "22cd4a8c-bc2c-3618-b1c3-4610c1b9b3e8",
    "SubscriptionId": "879"
  },
  "CertificateBot": {
    "IISBindings": [],
    "SaveCertificatePath": "C:/ssl",
    "IncludeCertificates": [
      "shopeneur.com,*.shopeneur.com"
    ]
  }
}
```

- Save the **appsettings.json** file when you are done.

# Create the Windows Service

- Open a **Command Prompt** in the Windows server as an **Administrator**

- Run the following command to install the Windows Service. Replace the < file-path > placeholder with the actual path where your windows service zip files were extracted

```
sc.exe create CertificateBot binpath= <file-path>\RCL.CertificateBot.Windows.exe
```

- After the service in installed, open **Services** in Windows, look for the ``CertificateBot`` service and **Start** the service

![image](../images/certbot/winservice-start.png)

- Set the **Properties** of the service to start automatically when the server starts

![image](../images/certbot/winservice-automatic.png)

# View the Event Logs

- Open **Event Viewer**, under 'Windows Logs > Application', look for the ``RCL.CertificateBot.Windows`` events

![image](../images/certbot/winservice-events.PNG)

- Ensure that there are no error events for the service. If there are error events, the service is misconfigured and will not function

- Each time a certificate is downloaded and saved in the server or a certificate is scheduled for renewal, a log will be written

# Fixing Errors

If you encounter error events for the service in the Event Viewer, please stop the service and delete it completely. 

Ensure the 'appsettings' configuration is correct for the AAD Application and the certificate save path settings point to a folder that exists. 

Fix any other errors that are reported Then, re-install and restart the service.

# Deleting the Windows Service

If you need to remove the Windows Service for any reason, run the command to delete the service

```
sc.exe delete CertificateBot  
```

# Installing Certificates in Web Servers

RCL CertificateBot will automatically save renewed SSL/TLS certificate files to a folder in the server. You should then configure the web server to use these files to implement SSL/TLS in your website.

## Certificate Files

The SSL/TLS certificate files will be stored at the path you specified in the ``appsettings.json`` configuration file. In this example, we used the path ``C:/ssl`` to store the certificate files.

When configuring the web servers, you will reference the specific certificate files stored at that path in a folder generated by CertificateBot for a specified domain.

The following files are downloaded and saved on the server :

  - ``certificate.pfx`` - The PFX certificate file
  - ``primaryCertificate.crt`` The primary certificate file
  - ``fullChainCertificate.crt`` - The full chain certificate file
  - ``caBundle.crt`` - The intermediate certificate file
  - ``privateKey.key`` - The certificate's private key file

## Configuring the Web Servers

Please follow the links below to configure your web server:

- [Installing SSL/TLS Certificates in Apache Server](../installations/apache)
- [Installing SSL/TLS Certificates in Apache Tomcat](../installations/apache-tomcat)
- [Installing SSL/TLS Certificates in NGINX](../installations/nginx)
- [Installing SSL/TLS Certificates in Web Servers and Hosting Services](../installations/web-servers)

