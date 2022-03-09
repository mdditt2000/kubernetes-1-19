### MultiHost VS & policy CRD profiles attached via LTM policy and not assigned globally #2251

#### Issue

MultiHost VS & policy CRD profiles attached via LTM policy and not assigned globally

#### Recreate

When using the **hostGroup** CIS creates one VirtualServer as shown in the diagram below:

![virtual](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/github/2251/diagram/2022-03-09_13-03-48.png)

The http routing logic is handled in the policy as shown below:

![policy](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/github/2251/diagram/2022-03-09_13-09-40.png)

http routing

![routing](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/github/2251/diagram/2022-03-09_13-10-23.png)


The WAF policy is associated to the VirtualServer and **not** the http routing policy as shown below:

![virtual](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.8/github/2251/diagram/2022-03-09_13-10-23.png)

#### Actual Problem

Customer not able to control the assigned policies on FWDN level - only on IP/Virtual Server level

#### PM Response 

Correct as shown above, policy is assigned to the Virtual

#### Workaround

Use a custom AS3 ConfigMap with Shared VirtualIPs to provide the above logic