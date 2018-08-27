---
title: Google Cloud VPN Interop Guide for Alibaba Cloud VPN Gateway
description: Describes how to build site-to-site IPsec VPNs between Cloud VPN on Google Cloud Platform (GCP) and  Alibaba Cloud VPN Gateway
author: David Lin
tags: VPN, interop, alibaba, alibaba cloud vpn gateway
date_published: 2018-08-24
---

**BEGIN: HOW TO USE THIS TEMPLATE**

1. Make a copy of this template.
1. On your local computer, update and add information as indicated. Note:
    + Fill in the meta information (title, description, author, date_published at the top
      of this file.
    + There are notes in the template for you, the author, that explain where you need to
      make changes. These notes are enclosed in brackets (&lt; &gt;).
    + The template contains placeholders for things like the vendor and 
      product name. These are also enclosed in brackets—for example, 
      every place you see `<vendor-name>` and `<product-name>`, 
      substitute approriate names.
    + After you've made appropriate updates, _remove_ bracketed content.
    + Remove these instructions.
    + Because this is a template for a variety of setups, it might contain
      content that isn't releavent to your scenario. Remove (or update)
      any sections that don't apply to you.
1. Fork the [GoogleCloudPlatform/community/](https://github.com/GoogleCloudPlatform/community/) repo.
4. In your fork, add a new folder named `/tutorials/[YOUR_TUTORIAL]`. For the
   folder name, use hyphens to separate words. We recommend that you 
   include a product name in the folder name, such as `https-load-balancing-ingix`.
5. Copy the updated file to the `index.md` file of the new folder.
6. Create a branch.
7. Issue a PR to get your new content into the community site.
   
<**END: HOW TO USE THIS TEMPLATE**>

# Using Cloud VPN with Alibaba Cloud VPN Gateway

Learn how to build site-to-site IPsec VPNs between
[Cloud VPN](https://cloud.google.com/vpn/docs/) on Google Cloud Platform (GCP) and
Alibaba Cloud VPN Gateway.

- [Google Cloud VPN Interop Guide](#google-cloud-vpn-interop-guide)
- [Introduction](#introduction)
- [Topology](#topology)
- [Product environment](#product-environment)
- [Before you begin](#before-you-begin)
    - [GCP account and project](#gcp-account-and-project)
    - [Permissions](#permissions)
    - [IP Ranges](#ip-ranges)
 - [Configuration Overview](#configuration-overview)
    - [Configure the GCP side](#configure-the-gcp-side)
    - [Configure the Alibaba Cloud side](#configure-the-alibaba-cloud-side)
 - [Configure the GCP side](#configure-the-gcp-side)
    - [Initial tasks](#initial-tasks)
        - [Select a GCP project name](#select-a-gcp-project-name)
        - [Create a custom VPC network and subnet](#create-a-custom-vpc-network-and-subnet)
        - [Create the GCP external IP address](#create-the-gcp-external-ip-address)
    - [Configuring route-based IPsec VPN using static routing](#configuring-route-based-ipsec-vpn-using-static-routing)
        - [Configure the VPN gateway](#configure-the-vpn-gateway)
        - [Configure firewall rules](#configure-firewall-rules)
- [Configure the Alibaba Cloud side](#configure-the-alibaba-cloud-side)
    - [Create an Alibaba Cloud VPC](#create-an-alibaba-cloud-vpc)
    - [Create an Alibaba Cloud VSwitch](#create-an-alibaba-cloud-vswitch)
    - [Create an Alibaba Cloud VPN Gateway](#create-an-alibaba-cloud-vpn-gateway)
    - [Configure an Alibaba Cloud Customer Gateway](#configure-an-alibaba-cloud-customer-gateway)
    - [Configure an Alibaba Cloud IPSec Connection](#configure-an-alibaba-cloud-ipsec-connection)
    - [Configure an Alibaba Cloud Static Route Entry](#configure-an-alibaba-cloud-static-route-entry)
    - [Testing the configuration](#testing-the-configuration)
- [Troubleshooting IPsec on Alibaba Cloud VPN Gateway](#troubleshooting-ipsec-on-alibaba-cloud-vpn-gateway)
- [Reference documentation](#reference-documentation)
    - [GCP documentation](#gcp-documentation)
    - [Alibaba Cloud VPN Gateway documentation](#alibaba-cloud-vpn-gateway-documentation)
- [Appendix: Using gcloud commands](#appendix-using-gcloud-commands)
    - [Running gcloud commands](#running-gcloud-commands)
    - [Configuration parameters and values](#configuration-parameters-and-values)
    - [Setting environment variables for gcloud command parameters](#setting-environment-variables-for-gcloud-command-parameters)
    - [Configuring an IPsec VPN using dynamic routing](#configuring-an-ipsec-vpn-using-dynamic-routing)
    - [Configuring route-based IPsec VPN using static routing](#configuring-route-based-ipsec-vpn-using-static-routing)

<Put trademark statements here>: <vendor terminology> and the <vendor> logo are
trademarks of <vendor company name> or its affiliates in the United States
and/or other countries.

_Disclaimer: This interoperability guide is intended to be informational in
nature and shows examples only. Customers should verify this information by
testing it._

Author: cloudservices-gcp-support@eplus.com

## Introduction

This guide walks you through the process of configuring
Alibaba Cloud VPN Gateway for integration with the
[Cloud VPN service](https://cloud.google.com/vpn/docs) on GCP.

If you are using this guide to configure your Alibaba Cloud VPN
Gateway implementation, be sure to substitute the correct IP
information for your environment.

For more information about Cloud VPN, see the
[Cloud VPN Overview](https://cloud.google.com/compute/docs/vpn/overview).

## Terminology

Below are definitions of terms used throughout this guide.

**GCP Terminology**
-  **GCP VPC network**—A single virtual network within a single GCP project.
-  **On-premises gateway**—The VPN device on the non-GCP side of the
connection, which is usually a device in a physical data center or in
another cloud provider's network. GCP instructions are written from the
point of view of the GCP VPC network, so "on-premises gateway" refers to the
gateway that's connecting _to_ GCP.
-  **External IP address** or **GCP peer address**—a single static IP address
within a GCP project that exists at the edge of the GCP network.
-  **Static routing**—Manually specifying the route to subnets on the GCP
side and to the on-premises side of the VPN gateway.

**Alibaba Terminology**
-  **Alibaba Cloud VPC**-A private network established in Alibaba Cloud. VPCs 
are logically isolated from other virtual networks in Alibaba Cloud. VPCs allow 
you to launch and use Alibaba Cloud resources in your VPC.
-  **Alibaba Cloud VSwitch**-A VSwitch is a basic network device of a VPC and 
used to connect different cloud product instances. When creating a cloud product 
instance in a VPC, you must specify the VSwitch that the instance is located.
-  **Alibaba Cloud Zone**-Zones are physical areas with independent power grids 
and networks in one region. Alibaba recommends creating different VSwitches in 
different zones to achieve disaster Recovery.
-  **Alibaba Cloud VPN Gateway**-The VPN gateway is the IPsec VPN gateway created 
on the Alibaba Cloud side. One VPN gateway can have multiple VPN connections.
-  **Alibaba Cloud Customer Gateway**-The customer gateway is the VPN service 
deployed in the on-premises data Center or, in this case, the GCP Cloud VPN gateway. 
By creating a customer gateway, you can register the VPN information to the cloud, 
and then create a VPN connection to connect the VPN gateway and the customer gateway.
-  **Alibaba Cloud VRouter**-A VRouter is a hub in the VPC that connects all 
VSwitches in the VPC and serves as a gateway device that connects the VPC to 
other networks. VRouter routes the network traffic according to the configurations 
of route entries.
-  **Alibaba Cloud Route Entry**-A route entry specifies the next hop address for the 
network traffic destined to a CIDR block. It has two types of entries: system route 
entry and custom route entry.
-  **Alibaba Cloud Route Table**-A route table is a list of route entries in a VRouter.

## Topology

Cloud VPN supports the following topology with Alibaba Cloud VPN Gateway:

-  A site-to-site IPsec VPN tunnel configuration using static routing.

For detailed topology information, see the following resources:

-  For basic VPN topologies, see 
[Cloud VPN Overview](https://cloud.google.com/vpn/docs/concepts/overview).
-  For redundant topologies,  the
[Cloud VPN documentation on redundant and high-throughput VPNs](https://cloud.google.com/vpn/docs/concepts/redundant-vpns).

Disclaimer: At time of this writing, site-to-site IPsec VPN tunnel configuration using dynamic routing between Cloud VPN and Alibaba Cloud VPN Gateway is not supported. 

## Product environment

The on-premise VPN gateway used in this guide is as follows:

-  Vendor—Alibaba Cloud
-  Service—VPN Gateway

## Before you begin

Follow the steps in this section to prepare for VPN configuration.

**Note**: This guide assumes that you have basic knowledge of the
[IPsec](https://wikipedia.org/wiki/IPsec) protocol.

### GCP account and project

Make sure you have a GCP account. When you begin, you must select or create a
GCP project where you will build the VPN. For details, see
[Creating and Managing Projects](https://cloud.google.com/resource-manager/docs/creating-managing-projects).

### Permissions

To create a GCP network, a subnetwork, and other entities described in this
guide, you must be able to sign in to GCP as a user who has
[Network Admin](https://cloud.google.com/compute/docs/access/iam#network_admin_role)
permissions. For details, see
[Required Permissions](https://cloud.google.com/vpn/docs/how-to/creating-vpn-dynamic-routes#required_permissions)
in the article
[Creating a VPN Tunnel using Dynamic Routing](https://cloud.google.com/vpn/docs/how-to/creating-vpn-dynamic-routes).

### IP Ranges

The IP address ranges of the GCP VPC and the Alibaba VPC must not overlap.


## Configuration Overview

The Google Cloud VPN with Alibaba Cloud VPN Gateway configuration is comprised of the following steps.

1. Configure the GCP side
1. Configure the Alibaba Cloud side

## Configure the GCP side

This section covers the steps for creating a GCP IPsec VPN using static routing.
Both route-based Cloud VPN and policy-based Cloud VPN use static routing.  For
information on how this works, see the 
[Cloud VPN Overview](https://cloud.google.com/compute/docs/vpn/overview).

There are two ways to create VPN gateways on GCP: using the Google Cloud
Platform Console and using the
[gcloud command-line tool](https://cloud.google.com/sdk/).
This section describes how to perform the tasks using the GCP Console. To see
the `gcloud` commands for performing these tasks, see the
[appendix](#appendix-using-gcloud-commands).

### Initial tasks

Complete the following procedures before configuring a static GCP VPN gateway 
and tunnel.

**Important:** Throughout these procedures, you assign names to entities like
the VPC network and subnet, IP address, and so on. Each time you assign a name,
make a note of it, because you often need to use those names in later
procedures.

#### Select a GCP project name

1. [Open the GCP Console](https://console.google.com).
1. At the top of the page, select the GCP project you want to use.

    **Note**: Make sure that you use the same GCP project for all of the GCP
    procedures in this guide.

#### Create a custom VPC network and subnet

1. In the GCP Console,
[go to the VPC Networks page](https://pantheon.corp.google.com/networking/networks/list).
1. Click **Create VPC network**.
1. For **Name**, enter a name such as `vpn-vendor-test-network`. Remember
this name for later.
1. Under **Subnets, Subnet creation mode**, select the **Custom** tab and
then populate the following fields:

+ **Name**—The name for the subnet, such as `vpn-subnet-1`.
+ **Region**—The region that is geographically closest to the
    on-premises gateway, such as  `us-east1`.
+ **IP address range**—A range such as `172.16.1.0/24`.

1. In the **New subnet** window, click **Done**.
1. Click **Create**. You're returned to the **VPC networks** page, where it
takes about a minute for this network and its subnet to appear.

#### Create the GCP external IP address

1.  In the GCP Console,
[go to the External IP addresses page](https://pantheon.corp.google.com/networking/addresses/list).

1. Click **Reserve Static Address**.
1. Populate the following fields for the Cloud VPN address:

-  **Name**—The name of the address, such as `vpn-test-static-ip`.
    Remember the name for later.
-  **Region**—The region where you want to locate the VPN gateway.
    Normally, this is the region that contains the instances you want to
    reach.

1. Click **Reserve**. You are returned to the **External IP addresses** page.
After a moment, the page displays the static external IP address that you
have created.

1. Make a note of the IP address that is created so that you can use it to
configure the VPN gateway later.

#### Configure the VPN gateway

1. In the GCP Console, 
[go to the VPN page](https://console.cloud.google.com/networking/vpn/list).
1. Click **Create VPN connection**.
1. Populate the following fields for the gateway:

-  **Name**—The name of the VPN gateway. This name is displayed in the
    console and used in by the gcloud tool to reference the gateway. Use a
    name like `vpn-test-[VENDOR_NAME]-gw-1`, where `[VENDOR_NAME]` is a
    string that identifies the vendor.
-  **Network**—The VPC network that you created previously (for
    example,  `vpn-vendor-test-network`) that contains the instances that the
    VPN gateway will serve.
-  **Region**—The region where you want to locate the VPN gateway.
    Normally, this is the region that contains the instances you want to reach.
-  **IP address**—Select the 
    [static external IP address](#create-the-gcp-external-ip-address)
    (for example, `vpn-test-static-ip`) that you created for this gateway
    in the previous section.

1. Populate the fields for at least one tunnel:

-  **Name**—The name of the VPN tunnel, such as `vpn-test-tunnel1`.
-  **Remote peer IP address**—The public external IP address of the
    on-premises VPN gateway.
-  **IKE version**—`IKEv2` or `IKEv1`. IKEv2 is preferred, but IKEv1 is
    supported if it is the only supported IKE version that the on-premises
    gateway can use.
-  **Shared secret**—A character string used in establishing encryption
    for the tunnel. You must enter the same shared secret into both VPN
    gateways. For more information, see
    [Generating a Strong Pre-shared Key](https://cloud.google.com/vpn/docs/how-to/generating-pre-shared-key).

1. Under **Routing options**, select the **Route based** or **Policy based** tab.
1. Populate the following fields:

-  **Remote network IP range**—The range or ranges of the on-premises network,
   which is the network on the other side of the tunnel from the Cloud VPN gateway
   you are currently configuring. 
-  **Local subnetworks**—The local subnet or subnets of the Cloud VPN's VPC. 

1. Click **Create**. The GCP VPN gateway is initiated, and the tunnel is initiated.

This procedure automatically creates a static route to the on-premises subnet as
well as forwarding rules for UDP ports 500 and 4500 and for ESP traffic. The VPN
gateways will not connect until you've configured the on-premises gateway and
created firewall rules in GCP to allow traffic through the tunnel between the
Cloud VPN  gateway and the on-premises gateway.

#### Configure firewall rules

Next, you configure GCP firewall rules to allow inbound traffic from the
on-premises network subnets. You must also configure the on-premises network
firewall to allow inbound traffic from your VPC subnet prefixes.

1. In the GCP Console,
[go to the GCP Firewall rules page](https://console.cloud.google.com/networking/firewalls).
1. Click **Create firewall rule**.
1. Populate the following fields:

1. **Name**—A name such as `vpnrule1`.
1. **Network**—The name of the VPC network that you created
    previously (for example,  `vpn-vendor-test-network`).
1. **Target tags:**—All instances in the network  
1. **Source filter**—A filter to apply your rule to specific sources of
    traffic. In this case, choose source IP ranges.
1. **Source IP ranges**—The on-premises IP ranges to accept from the
    on-premises VPN gateway.
1. **Allowed protocols and ports**—The string `tcp;udp;icmp`.

1. Click **Create**.

## Configure the Alibaba Cloud side

This section includes sample tasks that describe how to configure the
on-premises side of the VPN gateway configuration using Alibaba Cloud VPN Gateway.

### Create an Alibaba Cloud VPC

### Create an Alibaba Cloud VSwitch

### Create an Alibaba Cloud VPN Gateway

### Configure an Alibaba Cloud Customer Gateway

### Configure an Alibaba Cloud IPSec Connection

### Configure an Alibaba Cloud Static Route Entry


### GCP-compatible settings for IPSec and IKE

Configuring the vendor side of the VPN network requires you to use IPsec and IKE
settings that are compatible with the GCP side of the network. The following
table lists settings and information about values compatible with GCP VPN.
Use these settings for the procedures in the subsections that follow.

<table>
<thead>
<tr>
<th><strong>Setting</strong></th>
<th><strong>Description or value</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>IPsec Mode</td>
<td>ESP+Auth Tunnel mode (Site-to-Site)</td>
</tr>
<tr>
<td>Auth Protocol</td>
<td>Pre-shared Key (psk)</td>
</tr>
<tr>
<td>Shared Secret</td>
<td>Also known as an IKE pre-shared key. Choose a strong password by following
<a
href="https://cloud.google.com/vpn/docs/how-to/generating-pre-shared-key">these
guidelines</a>. The shared secret is very sensitive as it allows access
into your network.</td>
</tr>
<tr>
<td>Start</td>
<td><code>Auto</code> (on-premises device should automatically restart the
connection if it drops)</td>
</tr>
<tr>
<td>PFS (Perfect Forward Secrecy)</td>
<td>group1, <code>group2</code> (default), group5, group14, group24</td>
</tr>
<tr>
<td>IKE ciphers</td>
<td><code>aes</code> (default), aes192, aes256, des, 3des <br>(For details about IKE ciphers for IKEv1 or IKEv2 supported by GCP,
including the additional ciphers for PFS, see <a
href="https://cloud.google.com/vpn/docs/concepts/supported-ike-ciphers">Supported
IKE Ciphers</a>).</td>
</tr>
</tbody>
</table>

#### Configure the IKE proposal and policy

<Insert the instructions for creating the IKE proposal and policy here. Below
are some examples of IKE algorithms to specify as part of the instructions.>

-  **Encryption algorithm**—<code>aes</code> (default), aes192, aes256, des, 3des
-  **Integrity algorithm**—<code>sha1</code> (default), md5
-  **Diffie-Hellman group—**—group1, <code>group2</code> (default), group5, group14, group24
-  **Lifetime/SA Life Cycels—**—<code>86400</code> seconds (default)


### Configuring static routing

Follow the procedure in this section to configure static routing of traffic to
the GCP network through the VPN tunnel interface.

```
<insert configuration code snippet here>
```

For more recommendations about on-premises routing configurations, see
[GCP Best Practices](https://cloud.google.com/router/docs/resources/best-practices).

### Saving the configuration

Follow the procedure in this section to save the on-premises configuration.

< insert the instructions for saving the configuration here.>

```
<insert configuration code snippet here>
```

### Testing the configuration

It's important to test the VPN connection from both sides of a VPN tunnel. 
For either side, make sure that the subnet that a machine or virtual machine 
is located in is being forwarded through the VPN tunnel.

First, create virtual machines (VMs) on both sides of the tunnel. Make sure 
that you configure the VMs on a subnet that will pass traffic through the VPN 
tunnel.

-  Instructions for creating virtual machines in Compute Engine are located
in the [Getting Started Guide](https://cloud.google.com/compute/docs/quickstart).
-  Instructions for creating virtual machines in Alibaba Cloud Elastic Compute Service (ECS)
are located at [ECS operation instructions](https://www.alibabacloud.com/help/doc-detail/25430.htm).
(Note: When creating an ECS instance, select **Pay-as-You-Go** for the billing method unless you
intend to place the instance under the **Subscription** billing method.)

After VMs have been deployed on both the GCP and Alibaba Cloud platforms, 
you can use an ICMP echo (ping) test to test network connectivity
through the VPN tunnel.

On the GCP side, use the following instructions to test the connection to a
machine that's behind the on-premises gateway:

1. In the GCP Console,
[go to the VM Instances page](https://console.cloud.google.com/compute?).
1. Find the GCP virtual machine you created.
1. In the **Connect** column, click **SSH**. A browser window opens at the VM
command line.
1. Ping a machine that's behind the on-premises gateway.


## Troubleshooting IPsec on Alibaba Cloud VPN Gateway

For troubleshooting information, see the [Alibaba IPSec Connections Troubleshooting Guide](https://partners-intl.aliyun.com/help/doc-detail/65802.htm?spm=a2c63.o282931.b99.27.4973a641RT0hOX) and [View IPSec Connections Logs](https://www.alibabacloud.com/help/doc-detail/65288.htm?spm=5176.11182206.0.0.3c5b4c8f4JJqGH#h2-view-ipsec-connection-logs5).


## Reference documentation

You can refer to the following Alibaba Cloud VPN Gateway documentation and
Cloud VPN documentation for additional information about both products.

### GCP documentation

To learn more about GCP networking, see the following documents:

-  [VPC Networks](https://cloud.google.com/vpc/docs)
-  [Cloud VPN Overview](https://cloud.google.com/compute/docs/vpn/overview)
-  [Creating Route-based VPNs](https://cloud.google.com/vpn/docs/how-to/creating-route-based-vpns)
-  [Creating Policy-based VPNs](https://cloud.google.com/vpn/docs/how-to/creating-policy-based-vpns)
-  [Advanced Cloud VPN Configurations](https://cloud.google.com/vpn/docs/concepts/advanced)
-  [Troubleshooting Cloud VPN](https://cloud.google.com/compute/docs/vpn/troubleshooting)

### Alibaba Cloud VPN Gateway documentation

For more product information on Alibaba Cloud VPN Gateway, see the following
Alibaba Cloud VPN Gateway feature configuration guides and datasheets:

-  [Alibaba VPN Gateway](https://www.alibabacloud.com/product/vpn-gateway)
-  [Document Center: VPN Gateway](https://partners-intl.aliyun.com/help/product/65234.htm?spm=a2c63.m28257.a1.26.25cb4fca3Qi2Bv)
-  [Configure a Site-to-Site VPN](https://partners-intl.aliyun.com/help/doc-detail/65072.htm?spm=a2c63.l28256.a3.7.13b5e889MWMzAE)
-  [Alibaba Cloud Region and Endpoint Names](https://www.alibabacloud.com/help/doc-detail/31837.htm)


## Appendix: Using gcloud commands

The instructions in this guide focus on using the GCP Console. However, you can
perform many of the tasks for the GPC side of the VPN configuration by using the
[gcloud command-line tool](https://cloud.google.com/sdk/gcloud/). Using `gcloud`
commands can be faster and more convenient if you're comfortable with using a
command-line interface.

### Running gcloud commands

You can run `gcloud` commands on your local computer by installing the [Cloud
SDK](https://cloud.google.com/sdk/). Alternatively, you can run `gcloud`
commands in [Cloud Shell](https://cloud.google.com/shell/), a browser-based
command line. If you use Cloud Shell, you don't need to install the SDK on your
own computer, and you don't need to set up authentication.

**Note**: The `gcloud` commands presented in this guide assume you are working
in a Linux environment. (Cloud Shell is a Linux environment.)

### Configuration parameters and values

The `gcloud` commands in this guide include parameters whose value you must
provide. For example, a command might include a GCP project name or a region or
other parameters whose values are unique to your context. The following table
lists the parameters and gives examples of the values. The section that follows
the table describes how to set Linux environment variables to hold the values
you need for these parameters.

<table>
<thead>
<tr>
<th><strong>Parameter description</strong></th>
<th><strong>Placeholder</strong></th>
<th><strong>Example value</strong></th>
</tr>
</thead>
<tbody>

<tr>
<td>Vendor name</td>
<td><code>[VENDOR_NAME]<code></td>
<td>(Your product's vendor name. This value should have no spaces or
punctuation in it other than underscores or hyphens, because it will be
used as part of the names for GCP entities.)</td>
</tr>

<tr>
<td>GCP project name </td>
<td><code>[PROJECT_NAME]<code></td>
<td><code>vpn-guide<code></td>
</tr>

<tr>
<td>Shared secret</td>
<td><code>[SHARED_SECRET]<code></td>
<td>See <a
href="https://cloud.google.com/vpn/docs/how-to/generating-pre-shared-key">Generating
a Strong Pre-shared Key</a>.</td>
</tr>

<tr>
<td>VPC network name</td>
<td><code>[VPC_NETWORK_NAME]<code></td>
<td><code>vpn-vendor-test-network<code></td>
</tr>

<tr>
<td>Subnet on the GCP VPC network (for example, <code>vpn-vendor-test-network</code>)</td>
<td><code>[VPC_SUBNET_NAME]<code></td>
<td><code>vpn-subnet-1<code></td>
</tr>

<tr>
<td>GCP region. Can be any region, but it should be geographically close to the
on-premises gateway.</td>
<td><code>[REGION]<code></td>
<td><code>us-east1<code></td>
</tr>

<tr>
<td>Pre-existing external static IP address that you configure for the internet
side of the Cloud VPN gateway.</td>
<td><code>[STATIC_EXTERNAL_IP]<code></td>
<td><code>vpn-test-static-ip<code></td>
</tr>

<tr>
<td>IP address range for the GCP VPC subnet (<code>vpn-subnet-1</code>)</td>
<td><code>[SUBNET_IP]<code></td>
<td><code>172.16.100.0/24<code></td>
</tr>

<tr>
<td>IP address range for the on-premises subnet. You will use this range when
creating rules for inbound traffic to GCP.</td>
<td><code>[IP_ON_PREM_SUBNET]<code></td>
<td><code>10.0.0.0/8<code></td>
</tr>

<tr>
<td>External static IP address for the internet interface of <vendor
name><product-name></td>
<td><code>[CUST_GW_EXT_IP]</code> </td>
<td>For example, <code>199.203.248.181</code></td>
</tr>

<tr>
<td>Cloud Router name (for dynamic routing)</td>
<td><code>[CLOUD_ROUTER_NAME]<code></td>
<td><code>vpn-test-vendor-rtr<code></td>
</tr>

<tr>
<td>BGP interface name</td>
<td><code>[BGP_IF]<code></td>
<td><code>if-1<code></td>
</tr>

<tr>
<td>BGP session name (for dynamic routing)</td>
<td><code>[BGP_SESSION_NAME]<code></td>
<td><code>bgp-peer1<code></td>
</tr>

<tr>
<td>The name for the first GCP VPN gateway.</td>
<td><code>[VPN_GATEWAY_1]<code></td>
<td><code>vpn-test-[VENDOR_NAME]-gw-1</code>, where <code>[VENDOR_ NAME]</code>
is the <code>[VENDOR_NAME]</code> string</td>
</tr>

<tr>
<td>The name for the first VPN tunnel for
<code>vpn-test-[VENDOR_NAME]-gw-1</code></td>
<td><code>[VPN_TUNNEL_1]<code></td>
<td><code>vpn-test-tunnel1<code></td>
</tr>

<tr>
<td>The name of a firewall rule that allows traffic between the on-premises
network and GCP VPC networks</td>
<td><code>[VPN_RULE]<code></td>
<td><code>vpnrule1<code></td>
</tr>

<tr>
<td>The name for the <a
href="https://cloud.google.com/sdk/gcloud/reference/compute/routes/create">static
route</a> used to forward traffic to the on-premises network.<br>
<br>
<strong>Note</strong>: You need this value only if you are creating a VPN
using a static route.</td>
<td><code>[ROUTE_NAME]<code>

</td>
<td><code>vpn-static-route<code>

</td>
</tr>
<tr>
<td>The name for the forwarding rule for the <a
href="https://wikipedia.org/wiki/IPsec#Encapsulating_Security_Payload">ESP
protocol</a></td>
<td><code>[FWD_RULE_ESP]<code></td>
<td><code>fr-esp<code></td>
</tr>

<tr>
<td>The name for the forwarding rule for the <a
href="https://wikipedia.org/wiki/User_Datagram_Protocol">UDP
protocol</a>, port 500</td>
<td><code>[FWD_RULE_UDP_500]<code></td>
<td><code>fr-udp500<code></td>
</tr>

<tr>
<td>The name for the forwarding rule for the UDP protocol, port 4500</td>
<td><code>[FWD_RULE_UDP_4500]<code></td>
<td><code>fr-udp4500<code></td>
</tr>

</tbody>
</table>

### Setting environment variables for gcloud command parameters

To make it easier to run `gcloud` commands that contain parameters, you can
create environment variables to hold the values you need, such as your project
name, the names of subnets and forwarding rules, and so on. The `gcloud`
commands presented in this section reference variables that contain your
values.

To set the environment variables, run the following commands at the command line
_before_ you run `gcloud` commands, substituting your own values for all the
placeholders in square brackets, such as `[PROJECT_NAME]`, `[VPC_NETWORK_NAME]`,
and `[SUBNET_IP]`. If you don't know what values to use for the placeholders,
use the example values from the parameters table in the preceding section.

```
export PROJECT_NAME=[PROJECT_NAME]
export REGION=[REGION]
export VPC_SUBNET_NAME=[VPC_SUBNET_NAME]
export VPC_NETWORK_NAME=[VPC_NETWORK_NAME]
export FWD_RULE_ESP=[FWD_RULE_ESP]
export FWD_RULE_UDP_500=[FWD_RULE_UDP_500]
export FWD_RULE_UDP_4500=[FWD_RULE_UDP_4500]
export SUBNET_IP=[SUBNET_IP]
export VPN_GATEWAY_1=[VPN_GATEWAY_1]
export STATIC_EXTERNAL_IP=[STATIC_EXTERNAL_IP]
export VPN_RULE=[VPN_RULE]
export IP_ON_PREM_SUBNET=[IP_ON_PREM_SUBNET]
export CLOUD_ROUTER_NAME=[CLOUD_ROUTER_NAME]
export BGP_IF=[BGP_IF]
export BGP_SESSION_NAME=[BGP_SESSION_NAME]
export VPN_TUNNEL_1=[VPN_TUNNEL_1]
export CUST_GW_EXT_IP=[CUST_GW_EXT_IP]
export ROUTE_NAME=[ROUTE_NAME]
```

### Configuring route-based IPsec VPN using static routing

This section describes how to use the `gcloud` command-line tool to configure
IPsec VPN with static routing. To perform the same task using the GPC Console,
see
[Configuring IPsec VPN using static routing](#configuring-route-based-ipsec-vpn-using-static-routing)
earlier in this guide.

The procedure suggests creating a custom VPC network. This is preferred over
using an auto-created network. For more information, see
[Networks and Tunnel Routing](https://cloud.google.com/vpn/docs/concepts/choosing-networks-routing#network-types)
in the Cloud VPN documentation.

**Note**: Before you run the `gcloud` commands in this section, make sure that
you've set the variables as described earlier under
[Setting environment variables for gcloud command parameters](#setting-environment-variables-for-gcloud-command-parameters).

1. Create a custom VPC network. Make sure there is no conflict with your
local network IP address range. Note the following:

    -  For `[RANGE]`, substitute an appropriate CIDR range, such as
    `172.16.100.0/24`.

    ```
    gcloud compute networks create $VPC_NETWORK_NAME \
        --project $PROJECT_NAME \
        --subnet-mode custom

    gcloud compute networks subnets create $VPC_SUBNET_NAME \
        --project $PROJECT_NAME \
        --network $VPC_NETWORK_NAME \
        --region $REGION \
        --range [RANGE]
    ```

1. Create a VPN gateway in the region you are using. Normally, this is the
region that contains the instances you want to reach.

    ```
    gcloud compute target-vpn-gateways create $VPN_GATEWAY_1 \
        --project $PROJECT_NAME \
        --network $VPC_NETWORK_NAME \
        --region $REGION
    ```

This step creates an unconfigured VPN gateway in your GCP VPC network.

1. Reserve a static IP address in the VPC network and region where you
created the VPN gateway. Make a note of the address that is created for use
in future steps.

    ```
    gcloud compute addresses create $STATIC_EXTERNAL_IP \
        --project $PROJECT_NAME \
        --region $REGION
    ```

1. Create three forwarding rules, one each to forward ESP, IKE, and NAT-T
traffic to the Cloud VPN gateway. Note the following:

    -  For `[STATIC_IP_ADDRESS]`, use the static IP address that you reserved in
    the previous step.

    ```
    gcloud compute forwarding-rules create $FWD_RULE_ESP \
        --project $PROJECT_NAME \
        --region $REGION \
        --ip-protocol ESP \
        --target-vpn-gateway $VPN_GATEWAY_1 \
        --address [STATIC_IP_ADDRESS]

    gcloud compute forwarding-rules create $FWD_RULE_UDP_500 \
        --project $PROJECT_NAME \
        --region $REGION \
        --ip-protocol UDP \
        --target-vpn-gateway $VPN_GATEWAY_1 \
        --ports 500 \
        --address [STATIC_IP_ADDRESS]

    gcloud compute forwarding-rules create $FWD_RULE_UDP_4500 \
        --project $PROJECT_NAME \
        --region $REGION \
        --ip-protocol UDP \
        --target-vpn-gateway $VPN_GATEWAY_1 \
        --ports 4500 \
        --address [STATIC_IP_ADDRESS]
    ``` 

1. Create a VPN tunnel on the Cloud VPN Gateway that points to the external
IP address of your on-premises VPN gateway. Note the following:

-  Set the IKE version. The following command sets the IKE version to
    2, which is the default, preferred IKE version. If you need to set it to
    1, use `--ike-version 1`.
-  For `[SHARED_SECRET]`, supply the shared secret.  For details, see
    [Generating a Strong Pre-shared Key](https://cloud.google.com/vpn/docs/how-to/generating-pre-shared-key).
-  For `[LOCAL_TRAFFIC_SELECTOR_IP]`, supply an IP address range, like
    `172.16.100.0/24`,  that will be accessed on the GCP side of the  tunnel,
    as described in
    [Traffic selectors](https://cloud.google.com/vpn/docs/concepts/choosing-networks-routing#static-routing-networks)
    in the GCP VPN networking documentation.

    ```
    gcloud compute vpn-tunnels create $VPN_TUNNEL_1 \
        --project $PROJECT_NAME \
        --peer-address $CUST_GW_EXT_IP \
        --region $REGION \
        --ike-version 2 \
        --shared-secret [SHARED_SECRET] \
        --target-vpn-gateway $VPN_GATEWAY_1 \
        --local-traffic-selector [LOCAL_TRAFFIC_SELECTOR_IP]
    ``` 

    After you run this command, resources are allocated for this VPN tunnel, but it
    is not yet passing traffic.

1. Use a
[static route](https://cloud.google.com/sdk/gcloud/reference/compute/routes/create)
to forward traffic to the destination range of IP addresses in your
    on-premises network. The region must be the
    same region as for the VPN tunnel.

    ```
    gcloud compute routes create $ROUTE_NAME \
        --project $PROJECT_NAME \
        --network $VPC_NETWORK_NAME \
        --next-hop-vpn-tunnel $VPN_TUNNEL_1 \
        --next-hop-vpn-tunnel-region $REGION \
        --destination-range $IP_ON_PREM_SUBNET
    ```

1. If you want to pass traffic from multiple subnets through the VPN tunnel,
repeat the previous step to forward the IP address of each of the subnets.

1. Create firewall rules to allow traffic between the on-premises network and
GCP VPC networks.

    ```
    gcloud compute firewall-rules create $VPN_RULE \
        --project $PROJECT_NAME \
        --network $VPC_NETWORK_NAME \
        --allow tcp,udp,icmp \
        --source-ranges $IP_ON_PREM_SUBNET
    ```
