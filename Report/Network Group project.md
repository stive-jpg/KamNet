
# 1. Project Overview

Perimetre / Objectif

**Project Context:**

KamNet is a production company relocating to a new office. The company needs a new secure and efficient network. The goal is to modernize the actual setup with a cost-effective, well-structured plan that optimizes resource allocation while adhering to stringent security best practices.


**Constraints and Requirements:**

* **Client Requirements:**

    * DNS server
    * DHCP server
    * DMZ concept implemented through VLANs and access control lists (ACLs) (firewall alternative)
    * iSCSI storage server
    * Four network sectors:
        * Management/Secretariat (5 workstations)
        * Study (8 workstations)
        * Production (10 workstations)
        * Support (2 sectors, 10 workstations each) 
    - Extensive security managed through firewall or alternative


**Guidelines:**

* **Scalability and Manageability:** Design a network that can accommodate future growth without compromising efficiency.
* **Resource Optimization:** Allocate resources strategically, considering costs and future needs.
* **Clear Documentation:** Document IP addressing, VLANs (if used), and device configurations meticulously.
* **Robust Security:** Implement measures like ACLs, strong passwords, and encryption (where applicable).
* **Cost-Effectiveness:** Make budget-conscious choices in equipment selection and configuration.


# NETWORK DEFINITION

### 1. Current setup

The current infrastructure of KamNet is composed of 43 end users dispatched in 5 sectors. The infrastructure also hosts 3 servers. 

The network is divided like so : 

Four network sectors:
        * Management/Secretariat (5 workstations)
        * Study (8 workstations)
        * Production (10 workstations)
        * Support (2 sectors, 10 workstations each)

Total hosts 46.
### 2. Network / Subnet attributions

We decided to build the new infrastructure with 2 networks of 254 hosts each. The goal is to permit a reasonable growth of the company (from 46 to 254 hosts if required) while maintaining low costs and avoiding network congestion.

Each department is set on its own subnet, composed allowing its own possible growth based on the current setup. 

The figure below describes the possible growth of each department. 

| Name       | IP /CIDR                            | Current needs | Max Hosts | Potential growth |
| ---------- | ----------------------------------- | ------------- | --------- | ---------------- |
| Production | 192.168.1.0 /25                     | 10            | 126       | 1260%            |
| Study      | 192.168.1.128 /26                   | 8             | 62        | 775%             |
| Direction  | 192.168.1.192 /27                   | 5             | 30        | 600%             |
| Server     | 192.168.1.224 /28                   | 3             | 14        | 467%             |
| Support    | 192.168.2.0 /26<br>192.168.2.64 /26 | 20            | 124       | 620%             |
| ExtraPool1 | 192.168.1.240 /28                   | /             | 14        | /                |
| ExtraPool2 | 192.168.2.128 /25                   | /             | 126       | /                |

It is noticeable that the Support department is set up on its own network. Likewise, if each subnet reach its hosts limits, it is still possible to use the 126 available hosts from ExtraPool2 in order to extend the Support department capacity.

### 3. VLAN segmentation

To enhance inter-network security and reach manageability expectations, we decided to deploy a VLAN segmentation across the company network.

Each department is set on its own VLAN. This setup makes it easier to manage access to the different areas of the network. 
Since the end-user configuration is deployed through DHCP, adding extra computers to any department is almost Plug and Play.

The VLAN configuration is based on the Network/Subnet Definition

| Name       | VLAN<br>NUMBER | VLAN NAME | IP /CIDR          | Mask            | Gateway       |
| ---------- | -------------- | --------- | ----------------- | --------------- | ------------- |
| Server     | 40             | SERV      | 192.168.1.224 /28 | 255.255.255.240 | 192.168.1.225 |
| Direction  | 50             | MGMT      | 192.168.1.192 /27 | 255.255.255.224 | 192.168.1.193 |
| Study      | 60             | STUDY     | 192.168.1.128 /26 | 255.255.255.192 | 192.168.1.129 |
| Production | 70             | PROD      | 192.168.1.0 /25   | 255.255.255.128 | 192.168.1.1   |
| Support1   | 80             | SPT1      | 192.168.2.0 /26   | 255.255.255.192 | 192.168.2.1   |
| Support2   | 90             | SPT2      | 192.168.2.64 /26  | 255.255.255.192 | 192.168.2.65  |
| ExtraPool1 | 10             | EXP1      | 192.168.1.240 /28 | 255.255.255.240 | 192.168.1.241 |
| ExtraPool2 | 20             | EXP2      | 192.168.2.128 /25 | 255.255.255.128 | 192.168.2.129 |

### 4. Network Topology

The network is build regarding a mixed Star and Tree topology.

Each of 192.168.1.0 network 192.168.2.0 network is build around a central Switch, corresponding to a Star topology ensuring a cost effective and scalable infrastructure at a company level.

Both top switch from those networks are linked to a top router (Tree topology) for manageability purpose. 

Within each department/subnetwork, the hosts are linked to their own switch in order to ensure a fast and easy growth at a department level. 

# NETWORK SETUP


Every piece of hardware is a Cisco product.

| Role          | Hardware    | Name        | Desc                                          |
| ------------- | ----------- | ----------- | --------------------------------------------- |
| Main Router   | 1941 Switch | Router1<br> | Handle traffic inside and outside the network |
| Core Switch   | 2960 Switch | swCore1     | Central hub. Manage VLANs authorization       |
| VLAN40 Switch | 2960 Switch | swServer    |                                               |
| VLAN50 Switch | 2960 Switch | swDir       |                                               |
| VLAN60 Switch | 2960 Switch |             |                                               |
| VLAN70 Switch | 2960 Switch |             |                                               |
| VLAN80 Switch | 2960 Switch | swSupport1  |                                               |
| VLAN90 Switch | 2960 Switch | swSupport2  |                                               |
|               |             |             |                                               |


| NUMBER | NAME   |
| ------ | ------ |
| 10     | RADIUS |
| 20     | DNS    |
| 30     | ISCSI  |
| 40     | DHCP   |
| 50     | MGMT   |
| 60     | STUDY  |
| 70     | PROD   |
| 80     | SPT1   |
| 90     | SPT2   |
|        |        |


Router#int g0/0.60 
Router(config-subif)#encapsulation dot1Q 60 
Router(config-subif)#ip add 192.168.1.129 255.255.255.192 
Router(config-subif)#ip helper-address 192.168.1.226 
Router(config-subif)#exit 
Router#int g0/0 
Router#no shutdown 
***

(config)AAA new-model
(config)radius-server host 192.168.1.2 key cisco
(config)aaa authentication login AAA group radius
(config)line vty 0 4
(config-line) login authentication AAA
exit

***

ACL Standard

Router (config) # access-list 1 permit host 172.16.1.10

ou Router (config) # access-list 1 permit 172.16.1.10 + mask inverse

Router (config) # access-list 1 deny 176.16.1.0 0.0.0.255

Router (config) # access-list 1 permit any

Router (config) # int g0/0

Router (config-if) # ip access-group 1 out

***

- Apply an ACL `in` when you want to stop unwanted traffic as soon as it arrives at the router.
- Apply an ACL `out` when you need to control traffic that is leaving through the router's interface after processing.

