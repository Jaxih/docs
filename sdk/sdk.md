---
title: RCL SDK
description: The RCL SDK provides a C# library to make authorized requests to the RCL Public API 
has_children: false
nav_order: 7
---

# RCL SDK

The [RCL SDK](https://github.com/rcl-ssl/RCL.SDK) provides a C# .NET Core library to make authorized requests to the [RCL Public API](../api/api.md) . In this section, you will learn how to use the SDK.

## Project's GitHub Page

The project's GitHub page is located at :

[https://github.com/rcl-ssl/RCL.SDK](https://github.com/rcl-ssl/RCL.SDK)

## Installing the SDK

You can install the SDK from NuGet :

```
Install-Package RCL.SDK -Version 6.0.10
```

## Configure the Application using the SDK

Configure the application consuming the SDK by using the app's configuration files (appsettings.json, local.settings.json) or UserSecrets for testing.

### Example Settings in UserSecrets

```
{
  "RCLSDK:ClientId": "0era74aa-7455-4a43-a32f-c3c56g56cf",
  "RCLSDK:ClientSecret": "df84yehrer~_6-f5lNI84B0ER.dft4",
  "RCLSDK:TenantId": "se34rdc-bc7c-4929-b9c2-45dfdt5540e7",
  "RCLSDK:ApiBaseUrl": "https://rclapi.azure-api.net/public",
  "RCLSDK:SubscriptionId": "879"
}
```

The configuration must be changed to match the user's AAD Application credentials.

## Register an AAD Application

To obtain the values for the AAD configuration, a user would would need to Register an AAD Application and obtain the credentials for the application.

Please refer to the following link for instructions :

- [Register an AAD Application](../authorization/aad-application)

## Get the AAD Credentials

To obtain the following credentials from the AAD application:

- ClientId
- ClientSecret
- TenantId

follow the instructions in this link :

- [Get the AAD Application Credentials](../authorization/aad-application#get-the-aad-application-credentials)

## Get the SubscriptionId

Get the **Subscription Id** in the RCL Portal.

![install](../images/autorenew_configure/add_subscriptionid.png)

- Scroll down and copy the 'Subscription Id' 

![install](../images/autorenew_configure/add_subscriptionid2.png)

## Set Access Control for the AAD Application

Access Control must be set for the AAD application to access a user's Azure Resources.

Please refer to the following link for instructions :

- [Set Access Control for the AAD application](../authorization/access-control-app)

## Register the AAD Application's ``Client Id`` in the RCL Portal

The AAD Application must be associated with a user's RCL subscription. This is achieved by registering the AAD Application's ``Client Id`` in the **RCL Portal**.

To add the AAD Application's ``Client Id`` to the portal, please follow the instructions in this link :

- [Add the Client Id in the RCL Portal](../api/authorization#add-the-client-id-in-the-rcl-lets-encrypt-portal)

## Add the Services

 Set up the application using the SDK to use **Dependency Injection (DI)**; and add the SDK services ``AddRCLSDKService`` to the ``IServiceCollection``.

### Example Dependency Injection

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace RCL.SDK.Test
{
    public static class DependencyResolver
    {
        public static ServiceProvider ServiceProvider()
        {
            ConfigurationBuilder builder = new ConfigurationBuilder();
            builder.AddUserSecrets<TestProject>();
            IConfiguration configuration = builder.Build();

            IServiceCollection services = new ServiceCollection();

            services.AddRCLSDKService(options => configuration.Bind("RCLSDK", options));

            return services.BuildServiceProvider();
        }
    }

    public class TestProject
    {
    }
}
```

### Example ASP.NET Core App

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRCLSDK(options => Configuration.Bind("RCLSDK", options));
}
```

## Inject the ``CertificateService`` in your Class or Controller

### Example Console App

```csharp
using System;
using System.Threading.Tasks;

namespace RCL.SDK.ConsoleSample
{
    class Program
    {
        private static readonly ICertificateService _certificateService;

        static Program()
        {
            _certificateService = (ICertificateService)Startup
                .ServiceProvider().GetService(typeof(ICertificateService));
        }

        static async Task Main(string[] args)
        {
        }
    }
}
```

### Example 

```csharp
namespace RCL.SDK.Test
{
    public class CertificateServiceTest
    {
        private readonly ICertificateRequestService _certificateRequestService;

        public CertificateServiceTest()
        {
            _certificateRequestService = (ICertificateRequestService)DependencyResolver
                 .ServiceProvider().GetService(typeof(ICertificateRequestService));
        }
    }
}
```

### Example ASP.NET Core

```csharp
namespace RCL.SDK
{
    public class GetCertificate
    {
        private readonly ICertificateRequestService _certificateRequestService;

        public GetCertificate( ICertificateRequestService certificateRequestService)
        {
            _certificateRequestService = certificateRequestService;
        }
    }
}
```

## Using the ``CertificateService``

### Run the Certificate Test API to test the API connectivity

[POST Certificate Test](../api/post-certificate-test.md)

```csharp
await _certificateRequestService.GetTestAsync();
```

### Get a Certificate in a User's Subscription

[POST Certificate](../api/post-certificate.md)

```csharp
Certificate certificate = new Certificate
{
    certificateName = "shopeneur.com"
};

Certificate _certificate = await _certificateRequestService.GetCertificateAsync(certificate);
```

### Get a List of Certificates that should be renewed in a User's Subscription

[POST Certificate Renew Get List](../api/post-certificate-renew-get-list.md)

```csharp
List<Certificate> certificates = await _certificateRequestService.GetCertificatesToRenewAsync();
```

### Renew a Certificate in a User's Subscription

[POST Certificate Renew](../api/post-certificate.renew.md)

```csharp
Certificate certificate = new Certificate
{
    certificateName = "shopeneur.com"
};

await _certificateRequestService.RenewCertificateAsync(certificate);
```

## Production Apps

The following productions apps were built with the RCL SDK :

- [RCL AutoRenew Function](../autorenew/autorenew.md)