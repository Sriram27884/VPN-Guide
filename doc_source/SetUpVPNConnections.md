# Getting Started<a name="SetUpVPNConnections"></a>

Use the following procedures to manually set up the AWS Site\-to\-Site VPN connection with a virtual private gateway\. Alternatively, you can let the VPC creation wizard take care of many of these steps for you\. For more information about using the VPC creation wizard to set up the virtual private gateway, see [Scenario 3: VPC with Public and Private Subnets and AWS Site\-to\-Site VPN Access](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario3.html) or [ Scenario 4: VPC with a Private Subnet Only and AWS Site\-to\-Site VPN Access](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario4.html) in the *Amazon VPC User Guide*\.

To set up a Site\-to\-Site VPN connection, complete the following steps:
+ Step 1: [Create a Customer Gateway](#vpn-create-cgw)
+ Step 2: [Create a Virtual Private Gateway](#vpn-create-vpg)
+ Step 3: [Enable Route Propagation in Your Route Table](#vpn-configure-routing)
+ Step 4: [Update Your Security Group ](#vpn-configure-security-groups)
+ Step 5: [Create a Site\-to\-Site VPN Connection and Configure the Customer Gateway Device](#vpn-create-vpn-connection)

These procedures assume that you have a VPC with one or more subnets\.

For steps to create a Site\-to\-Site VPN connection on a transit gateway, see [Creating a Transit Gateway VPN Attachment](create-tgw-vpn-attachment.md)\.

## Create a Customer Gateway<a name="vpn-create-cgw"></a>

A customer gateway provides information to AWS about your customer gateway device or software application\. For more information, see [Customer Gateway](how_it_works.md#CustomerGateway)\.

If you plan to use a private certificate to authenticate your VPN, create a private certificate from a subordinate CA using AWS Certificate Manager Private Certificate Authority\. For information about creating a private certificate, see [Creating and Managing a Private CA](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreatingManagingCA.html) in the *AWS Certificate Manager Private Certificate Authority User Guide*\.

**Note**  
You must specify either an IP address, or an Amazon Resource Name of the private certificate\.

**To create a customer gateway using the console**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Customer Gateways**, and then **Create Customer Gateway**\.

1. Complete the following and then choose **Create Customer Gateway**:
   + \(Optional\) For **Name**, enter a name for your customer gateway\. Doing so creates a tag with a key of `Name` and the value that you specify\.
   + For **Routing**, select the routing type\.
   + For dynamic routing, for **BGP ASN**, enter the Border Gateway Protocol \(BGP\) Autonomous System Number \(ASN\)\.
   + \(Optional\) For **IP Address**, type the static, internet\-routable IP address for your customer gateway device\. If your customer gateway is behind a NAT device that's enabled for NAT\-T, use the public IP address of the NAT device\.
**Note**  
This is optional when you use a private certificate for VPN connections to a virtual private gateway \(VGW\)\.
   + \(Optional\) If you want to use a private certificate, for **Certificate ARN**, choose the Amazon Resource Name of the private certificate\.

**To create a customer gateway using the command line or API**
+ [CreateCustomerGateway](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-CreateCustomerGateway.html) \(Amazon EC2 Query API\)
+ [create\-customer\-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-customer-gateway.html) \(AWS CLI\)
+ [New\-EC2CustomerGateway](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2CustomerGateway.html) \(AWS Tools for Windows PowerShell\)

## Create a Virtual Private Gateway<a name="vpn-create-vpg"></a>

When you create a virtual private gateway, you can optionally specify the private Autonomous System Number \(ASN\) for the Amazon side of the gateway\. The ASN must be different from the BGP ASN specified for the customer gateway\.

After you create a virtual private gateway, you must attach it to your VPC\.

**To create a virtual private gateway and attach it to your VPC**

1. In the navigation pane, choose **Virtual Private Gateways**, **Create Virtual Private Gateway**\.

1. \(Optional\) Enter a name for your virtual private gateway\. Doing so creates a tag with a key of `Name` and the value that you specify\.

1. For **ASN**, leave the default selection to use the default Amazon ASN\. Otherwise, choose **Custom ASN** and enter a value\. For a 16\-bit ASN, the value must be in the 64512 to 65534 range\. For a 32\-bit ASN, the value must be in the 4200000000 to 4294967294 range\. 

1. Choose **Create Virtual Private Gateway**\.

1. Select the virtual private gateway that you created, and then choose **Actions**, **Attach to VPC**\.

1. Select your VPC from the list and choose **Yes, Attach**\.

**To create a virtual private gateway using the command line or API**
+ [CreateVpnGateway](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-CreateVpnGateway.html) \(Amazon EC2 Query API\)
+ [create\-vpn\-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpn-gateway.html) \(AWS CLI\)
+ [New\-EC2VpnGateway](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2VpnGateway.html) \(AWS Tools for Windows PowerShell\)

**To attach a virtual private gateway to a VPC using the command line or API**
+ [AttachVpnGateway](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-AttachVpnGateway.html) \(Amazon EC2 Query API\)
+ [attach\-vpn\-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-vpn-gateway.html) \(AWS CLI\)
+ [Add\-EC2VpnGateway](https://docs.aws.amazon.com/powershell/latest/reference/items/Add-EC2VpnGateway.html) \(AWS Tools for Windows PowerShell\)

## Enable Route Propagation in Your Route Table<a name="vpn-configure-routing"></a>

To enable instances in your VPC to reach your customer gateway, you must configure your route table to include the routes used by your Site\-to\-Site VPN connection and point them to your virtual private gateway\. You can enable route propagation for your route table to automatically propagate those routes to the table for you\.

For static routing, the static IP prefixes that you specify for your VPN configuration are propagated to the route table when the status of the Site\-to\-Site VPN connection is `UP`\. Similarly, for dynamic routing, the BGP\-advertised routes from your customer gateway are propagated to the route table when the status of the Site\-to\-Site VPN connection is `UP`\.

**Note**  
If your connection is interrupted, any propagated routes in your route table are not automatically removed\. You may have to disable route propagation to remove the propagated routes; for example, if you want traffic to fail over to a static route\.

**To enable route propagation using the console**

1. In the navigation pane, choose **Route Tables**, and then select the route table that's associated with the subnet\. By default, this is the main route table for the VPC\.

1. On the **Route Propagation** tab in the details pane, choose **Edit**, select the virtual private gateway that you created in the previous procedure, and then choose **Save**\.

**Note**  
For static routing, if you do not enable route propagation, you must manually enter the static routes used by your Site\-to\-Site VPN connection\. To do this, select your route table, choose **Routes**, **Edit**\. For **Destination**, add the static route used by your Site\-to\-Site VPN connection\. For **Target**, select the virtual private gateway ID, and choose **Save**\.

**To disable route propagation using the console**

1. In the navigation pane, choose **Route Tables**, and then select the route table that's associated with the subnet\.

1. Choose **Route Propagation**, **Edit**\. Clear the **Propagate** check box for the virtual private gateway, and choose **Save**\.

**To enable route propagation using the command line or API**
+ [EnableVgwRoutePropagation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-EnableVgwRoutePropagation.html) \(Amazon EC2 Query API\)
+ [enable\-vgw\-route\-propagation](https://docs.aws.amazon.com/cli/latest/reference/ec2/enable-vgw-route-propagation.html) \(AWS CLI\)
+ [Enable\-EC2VgwRoutePropagation](https://docs.aws.amazon.com/powershell/latest/reference/items/Enable-EC2VgwRoutePropagation.html) \(AWS Tools for Windows PowerShell\)

**To disable route propagation using the command line or API**
+ [DisableVgwRoutePropagation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-DisableVgwRoutePropagation.html) \(Amazon EC2 Query API\)
+ [disable\-vgw\-route\-propagation](https://docs.aws.amazon.com/cli/latest/reference/ec2/disable-vgw-route-propagation.html) \(AWS CLI\)
+ [Disable\-EC2VgwRoutePropagation](https://docs.aws.amazon.com/powershell/latest/reference/items/Disable-EC2VgwRoutePropagation.html) \(AWS Tools for Windows PowerShell\)

## Update Your Security Group<a name="vpn-configure-security-groups"></a>

To allow access to instances in your VPC from your network, you must update your security group rules to enable inbound SSH, RDP, and ICMP access\.

**To add rules to your security group to enable inbound SSH, RDP and ICMP access**

1. In the navigation pane, choose **Security Groups**, and then select the default security group for the VPC\.

1. On the **Inbound** tab in the details pane, add rules that allow inbound SSH, RDP, and ICMP access from your network, and then choose **Save**\. For more information about adding inbound rules, see [Adding, Removing, and Updating Rules](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#AddRemoveRules) in the *Amazon VPC User Guide\.*

For more information about working with security groups using the AWS CLI, see [Security Groups for Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) in the *Amazon VPC User Guide*\.

## Create a Site\-to\-Site VPN Connection and Configure the Customer Gateway Device<a name="vpn-create-vpn-connection"></a>

After you create the Site\-to\-Site VPN connection, download the configuration information and use it to configure the customer gateway device or software application\.

**Important**  
The configuration file is an example only and might not match your intended VPN connection settings\. For example, it specifies the minimum requirements of IKE version 1, AES128, SHA1, and DH Group 2 in most AWS Regions, and IKE version 1, AES128, SHA2, and DH Group 14 in the AWS GovCloud Regions\. It also specifies pre\-shared keys for [authentication](vpn-tunnel-authentication-options.md)\. You must modify the example configuration file to take advantage of IKE version 2, AES256, SHA256, other DH groups such as 2, 14\-18, 22, 23, and 24, and private certificates\.   
If you specified custom tunnel options when creating or modifying your Site\-to\-Site VPN connection, modify the example configuration file to match the custom settings for your tunnels\.

**To create a Site\-to\-Site VPN connection and configure the customer gateway**

1. In the navigation pane, choose **Site\-to\-Site VPN Connections**, **Create VPN Connection**\.

1. Complete the following information, and then choose **Create VPN Connection**:
   + \(Optional\) For **Name tag**, enter a name for your Site\-to\-Site VPN connection\. Doing so creates a tag with a key of `Name` and the value that you specify\.
   + Select the virtual private gateway that you created earlier\.
   + Select the customer gateway that you created earlier\.
   + Select one of the routing options based on whether your VPN router supports Border Gateway Protocol \(BGP\):
     + If your VPN router supports BGP, choose **Dynamic \(requires BGP\)**\.
     + If your VPN router does not support BGP, choose **Static**\. For **Static IP Prefixes**, specify each IP prefix for the private network of your Site\-to\-Site VPN connection\.
   + Under **Tunnel Options**, you can optionally specify the following information for each tunnel:
     + A size /30 CIDR block from the `169.254.0.0/16` range for the inside tunnel IP addresses\.
     + The IKE pre\-shared key \(PSK\)\. The following versions are supported: IKEv1 or IKEv2\.
     + Advanced tunnel information, which includes the encryption algorithms for phases 1 and 2 of the IKE negotiations, the integrity algorithms for phases 1 and 2 of the IKE negotiations, the Diffie\-Hellman groups for phases 1 and 2 of the IKE negotiations, the IKE version, the phase 1 and 2 lifetimes, the rekey margin time, the rekey fuzz, the replay window size, and the dead peer detection interval\.

     For more information about these options, see [Site\-to\-Site VPN Tunnel Options for Your Site\-to\-Site VPN Connection](VPNTunnels.md)\.

   It may take a few minutes to create the Site\-to\-Site VPN connection\. When it's ready, select the connection and choose **Download Configuration**\.

1. In the **Download Configuration** dialog box, select the vendor, platform, and software that corresponds to your customer gateway device or software, and then choose **Yes, Download**\. 

**To create a Site\-to\-Site VPN connection using the command line or API**
+ [CreateVpnConnection](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-CreateVpnConnection.html) \(Amazon EC2 Query API\)
+ [create\-vpn\-connection](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpn-connection.html) \(AWS CLI\)
+ [New\-EC2VpnConnection](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2VpnConnection.html) \(AWS Tools for Windows PowerShell\)

## Configure the Customer Gateway Device<a name="vpn-configure-customer-gateway-device"></a>

Give the configuration file that you downloaded to your network administrator, along with this guide: [AWS Site\-to\-Site VPN Network Administrator Guide](https://docs.aws.amazon.com/vpc/latest/adminguide/)\. The network administrator configures the customer gateway device with the settings that match the customer gateway\. After the network administrator configures the customer gateway device, the Site\-to\-Site VPN connection is operational\.

## Editing Static Routes for a Site\-to\-Site VPN Connection<a name="vpn-edit-static-routes"></a>

For static routing, you can add, modify, or remove the static routes for your VPN configuration\. 

**To add, modify, or remove a static route**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Site\-to\-Site VPN Connections**\.

1. Choose **Static Routes**, **Edit**\. 

1. Modify your existing static IP prefixes, or choose **Remove** to delete them\. Choose **Add Another Rule** to add a new IP prefix to your configuration\. When you are done, choose **Save**\.

**Note**  
If you have not enabled route propagation for your route table, you must manually update the routes in your route table to reflect the updated static IP prefixes in your Site\-to\-Site VPN connection\. For more information, see [Enable Route Propagation in Your Route Table](#vpn-configure-routing)\.

**To add a static route using the command line or API**
+ [CreateVpnConnectionRoute](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-CreateVpnConnectionRoute.html) \(Amazon EC2 Query API\)
+ [create\-vpn\-connection\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpn-connection-route.html) \(AWS CLI\)
+ [New\-EC2VpnConnectionRoute](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2VpnConnectionRoute.html) \(AWS Tools for Windows PowerShell\)

**To delete a static route using the command line or API**
+ [DeleteVpnConnectionRoute](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-DeleteVpnConnectionRoute.html) \(Amazon EC2 Query API\)
+ [delete\-vpn\-connection\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-vpn-connection-route.html) \(AWS CLI\)
+ [Remove\-EC2VpnConnectionRoute](https://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2VpnConnectionRoute.html) \(AWS Tools for Windows PowerShell\)