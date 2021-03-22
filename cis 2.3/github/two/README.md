# Two Migration from CC to CIS

This document is created to assist with migration of customer two environment using CC with iApp templates to CIS with CRDs. To successfully migrate from CC using iApp template to CIS with CRD the following features are required. 

##  Virtual Server

### Virtual Server Definition

Virtual Server Definition is configured by using the virtualServerName parameter

    virtualServerName: "K8S_iApp_004_test"

example [crd](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/crd/virtual-definition/vs-virtual-defintion.yaml)

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/diagrams/2021-03-17_13-25-56.png)

### Virtual Server Type

VirtualServer uses standard "Virtual Server Type" by default. This parameter cannot be modified

example [crd](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/crd/virtual-definition/vs-virtual-defintion.yaml)

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/diagrams/2021-03-17_13-25-56.png)

### Virtual Server IP

Virtual Server IP is configured by using the virtualServerAddress parameter

    virtualServerAddress: "10.192.75.110"

example [crd](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/crd/virtual-definition/vs-virtual-defintion.yaml)

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/diagrams/2021-03-17_13-25-56.png)

### Virtual Server Port

Virtual Server Port is configured by using the **virtualServerHTTPPort** and **virtualServerHTTPSPort** parameter. **VirtualServerHTTPPort** supports custom http ports, example 8081 and **virtualServerHTTPSPort** support https ports example 8443

    virtualServerHTTPPort: 80

example [crd](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/crd/virtual-definition/vs-virtual-defintion.yaml)

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/diagrams/2021-03-17_13-25-56.png)

    virtualServerHTTPPort: 443

### Virtual Server TCP Client Profile

Virtual Server uses the following default TCP Client Profile, TCP Server Profile which cannot currently be modified

![diagram](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/diagrams/2021-03-22_13-35-02.png)

- Opened Jira [RFE] for enhancement to support custom TCP Client Profile, TCP Server Profile: 
  CONTCNTR-2555

### Virtual Server HTTP Profile

* Virtual_Definition.VS_HTTP_Profile "HTTP Profile"
* Virtual_Definition.VS_Persistence "Default Persistence Profile"

* Virtual_Definition.VS_SSL_Client_Profiles "SSL Client Profiles"
* Virtual_Definition.VS_SSL_Server_Profiles "SSL Server Profiles"

* Virtual_Definition.VS_FastL4_Client_Profile "FastL4 Client Profile"
* Virtual_Definition.VS_VLAN_Enabled "Enabled on VLAN (ext/int)"

* Virtual_Definition.VS_SNATPool "SNAT Pool"
* Virtual_Definition.VS_iRules "Activated iRules"
* Virtual_Definition.VS_Application_Security_Policy "Application Security Policy"
* Virtual_Definition.VS_IP_Intelligence_Policy "IP Intelligence Policy"
* Virtual_Definition.VS_DoS_Protection_Profile "DoS Protection Profile"
* Virtual_Definition.VS_Bot_Defense_Profile "Bot Defense Profile"
* Virtual_Definition.VS_Log_Profiles "Log Profiles"

##  Pool

* Pool_Definition "Pool Definition"
* Pool_Definition.Pool_Health_Monitors "Health Monitors"
* Pool_Definition.Pool_Load_Balancing_Method "Load Balancing Method"
* Pool_Definition.Pool_Slow_Ramp_Time "Slow Ramp Time"
* Pool_Definition.Pool_Reselect_Tries "Reselect Tries"
* Pool_Definition.Pool_Members "Pool Members"
* Pool_Definition.Pool_Members.Pool_Member_IP "IP "
* Pool_Definition.Pool_Members.Pool_Member_Port "Port "

