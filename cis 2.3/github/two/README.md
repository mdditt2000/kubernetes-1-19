# Two Migration from CC to CIS

This document is created to assist with migration of customer two environment using CC with iApp templates to CIS with CRDs. To successfully migrate from CC using iApp template to CIS with CRD the following features are required. 

##  Virtual Server

### Virtual_Definition "Virtual Server Definition"

CIS allows the user to change the Virtual Server Definition by using the virtualServerName parameter

    virtualServerName: "Virtual Server Definition"

! example [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/two/crd/virtual-definition/vs-virtual-defintion.yaml)

 
* Virtual_Definition.VS_Type "Virtual Server Type"
* Virtual_Definition.VS_IP "Virtual Server IP"
* Virtual_Definition.VS_Port "Virtual Server Port"
* Virtual_Definition.VS_TCP_Client_Profile "TCP Client Profile"
* Virtual_Definition.VS_TCP_Server_Profile "TCP Server Profile"
* Virtual_Definition.VS_HTTP_Profile "HTTP Profile" 
* Virtual_Definition.VS_SSL_Client_Profiles "SSL Client Profiles"
* Virtual_Definition.VS_SSL_Server_Profiles "SSL Server Profiles"
* Virtual_Definition.VS_FastL4_Client_Profile "FastL4 Client Profile"
* Virtual_Definition.VS_VLAN_Enabled "Enabled on VLAN (ext/int)"
* Virtual_Definition.VS_SNATPool "SNAT Pool"
* Virtual_Definition.VS_Persistence "Default Persistence Profile"
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

