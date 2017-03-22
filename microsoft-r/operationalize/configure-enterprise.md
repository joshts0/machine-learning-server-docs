---

# required metadata
title: "Enterprise Configuration of Operationalization for R Server | Microsoft R Server Docs"
description: "Enterprise Configuration of Operationalization for Microsoft R Server"
keywords: ""
author: "j-martens"
manager: "jhubbard"
ms.date: "3/20/2017"
ms.topic: "article"
ms.prod: "microsoft-r"
ms.service: ""
ms.assetid: ""

# optional metadata
ROBOTS: ""
audience: ""
ms.devlang: ""
ms.reviewer: ""
ms.suite: ""
ms.tgt_pltfrm: ""
ms.technology:
  - deployr
  - r-server
ms.custom: ""
---

# Configuring R Server for Operationalization (Enterprise Configuration)

**Applies to:  Microsoft R Server 9.0.1 & 9.1**

To benefit from Microsoft R Server’s deployment and operationalization features, you can configure R Server after installation to act as a deployment server and host analytic web services. This article describes how to perform an enterprise configuration of these features. 

With an enterprise configuration, you can work with your production-grade data within a scalable, multi-machine setup, and benefit from enterprise-grade security.

This configuration includes one or more web nodes and one or more compute nodes, each of which can scaled independently.  Scaling up compute nodes enables you to have more R execution shells and benefit from load balancing across these compute nodes. 

Scaling up web nodes enables an active-active configuration that allows you to load balance the incoming API requests.  Additionally, when you have multiple web nodes, you'll need to use a [SQL Server or PostgreSQL database](configure-remote-database.md) so that data and web services can be shared and available for all requests across web node services.   

For added security, you can [configure SSL](security-https.md) as well as authenticate against [Active Directory (LDAP) or Azure Active Directory](security-authentication.md).

![Enterprise Configuration](../media/o16n/configure-enterprise.png)

If you'd like to learn more about web nodes and compute nodes or about the one-box configuration, [see here](configuration-initial.md). 

>[!Important]
>The operationalization feature for Microsoft R Server is supported on:
>- Windows Server 2012 R2, Windows Server 2016
>- Ubuntu 14.04, Ubuntu 16.04,
>- CentOS/RHEL 7.x

## 1. Configure a database

By default, the web node configuration sets up a local SQLite database. If you want to use a different or remote database, follow these instructions to [configure that database](configure-remote-database.md) (SQL Server or PostgreSQL).

If you plan to configure multiple web nodes, then you **must** set up a [remote SQL Server or PostgreSQL database](configure-remote-database.md) so that data can be shared across web node services.

>[!NOTE] 
> Create this database and register it in the configuration file below BEFORE the service for the control node is started.

<a name="add-compute-nodes"></a>

## 2. Configure compute node(s)

>[!IMPORTANT]
>We highly recommend that you configure each node (compute or web) on its own machine for higher availability. 

1. On each machine, install the same R Server version you installed on the web node.

1. If on the following Linux flavors, then add a few symlinks:  (If on Windows, skip to the next step)

   + On CentOS 7.1, CentOS 7.2:
     ```
      cd /usr/lib64
      sudo ln -s libpcre.so.1   libpcre.so.0
      sudo ln -s libicui18n.so.50   libicui18n.so.36
      sudo ln -s libicuuc.so.50 libicuuc.so.36
      sudo ln -s libicudata.so.50 libicudata.so.36
     ```

   + On Ubuntu 14.04:
     ```
      sudo apt-get install libicu-dev

      cd /lib/x86_64-linux-gnu
      ln -s libpcre.so.3 libpcre.so.0
      ln -s liblzma.so.5 liblzma.so.0

      cd /usr/lib/x86_64-linux-gnu
      ln -s libicui18n.so.52 libicui18n.so.36
      ln -s libicuuc.so.52 libicuuc.so.36
      ln -s libicudata.so.52 libicudata.so.36
     ```

   + On Ubuntu 16.04:
     ```
      cd /lib/x86_64-linux-gnu
      ln -s libpcre.so.3 libpcre.so.0
      ln -s liblzma.so.5 liblzma.so.0

      cd /usr/lib/x86_64-linux-gnu
      ln -s libicui18n.so.55 libicui18n.so.36
      ln -s libicuuc.so.55 libicuuc.so.36
      ln -s libicudata.so.55 libicudata.so.36
     ```

1. [Launch the administration utility](admin-utility.md#launch) with administrator privileges.

1. From the main menu, choose the option to **Configure R Server for Operationalization**.

1. From the sub-menu, choose the option to **Configure a compute node**.

1. When the configuration is finished, open port 12805: 
   + On Windows: Add an exception to your firewall to open port 12805. And, for additional security, you can also restrict communication for a private network or domain using a profile.

   + On Linux: If using the IPTABLES firewall or equivalent service on Linux, then use the `iptables` command (or the equivalent) to open port 12805.

1. From the main utility menu, choose the option **Stop and start services** and restart the compute node to define it as a service.


Your compute node is now configured. Repeat these steps for each compute node you want to add.

<br><a name="webnode"></a>

## 3. Configure web node(s)

>[!IMPORTANT]
>We highly recommend that you configure each node (compute or web) on its own machine for higher availability. 

>[!Note]
>It is possible to run the operationalization web node service from within IIS.

1. On each machine, install Microsoft R Server:
   + On Windows, install [R Server for Windows](https://msdn.microsoft.com/en-us/library/mt671127.aspx). 
   + On Linux, install [R Server for Linux](../rserver-install-linux-server.md).  

1. Declare the IP addresses of every compute node with each web node.
   1. Open the external configuration file, `appsettings.json` file.

      + On Windows, this file is under `<MRS_home>\deployr\Microsoft.DeployR.Server.WebAPI\` where `<MRS_home>` is the path to the Microsoft R Server installation directory. To find this path, enter `normalizePath(R.home())` in your R console.

      + On Linux, this file is under `/usr/lib64/microsoft-deployr/9.0.1/Microsoft.DeployR.Server.WebAPI/`.

   1. In the file, search for the section starting with `"BackEndConfiguration": {` .

   1. Update the `"Uris": {` properties to declare each compute node:
      ```
      "Uris": {
         "Values": [
           "http://<IP-ADDRESS-OF-COMPUTE-NODE-1>:12805",
           "http://<IP-ADDRESS-OF-COMPUTE-NODE-2>:12805",
           "http://<IP-ADDRESS-OF-COMPUTE-NODE-3>:12805"       
         ]
       }
       ```

      > Do not update any other properties in this file at this point. It will be updated during the compute node configuration.

   1. Close and save the file.

   1. Repeat these steps on each web node to declare each and every compute node.

1. [Launch the administration utility](admin-utility.md#launch) with administrator privileges:
   1. From the main menu, choose the option to **Configure R Server for Operationalization**.

   1. From the sub-menu, choose the option to **Configure a web node**.     

   1. When prompted, provide a password for the built-in, local operationalization administrator account called `admin`.
        Later, you can configure R Server to authenticate against  [Active Directory (LDAP) or Azure Active Directory](security-authentication.md).

   1. From the main menu, choose the option to **Run Diagnostic Tests**. Verify the configuration by running [diagnostic test](admin-diagnostics.md) on each web node.

   1. From the main utility menu, choose the option **Stop and start services** and restart the web node to define it as a service.

   1. Exit the utility.

1. If on Linux and using the IPTABLES firewall or equivalent service, then use the `iptables` command (or the equivalent) to open port 12800 to the public IP of the web node so that remote machines can access it.

Your web node is now configured. Repeat these steps for each web node you want to add.

>[!Important]
>R Server uses Kestrel as the web server for its operationalization web nodes. Consequently, if you expose your application to the Internet, we recommend that you review the [guidelines for Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel) regarding reverse proxy set up.


## 4. Configure enterprise-grade security

In production environments, we strongly recommend the following approaches:

1. [Configure SSL/TLS](security-https.md) and install the necessary certificates.

1. Authenticate against [Active Directory (LDAP) or Azure Active Directory](security-authentication.md).  

1. For added security, restrict the list of IPs that can access the machine hosting the compute node.


## 5. Provision on the cloud

If you are provisioning on a cloud service, then you must also [create inbound security rule for port 12800 in Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-classic-setup-endpoints/) or open the port through the AWS console. This endpoint allows clients to communicate with the R Server's operationalization server.


## 6. Post configuration steps

1. [Update service ports](admin-utility.md#ports), if needed.

1. [Run diagnostic tests](admin-diagnostics.md).

1. [Evaluate](admin-evaluate-capacity.md) the configuration's capacity.

1. Set up the load balance of your choosing. 