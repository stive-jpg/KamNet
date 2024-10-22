Yes, it is possible to use **subnetting** within the **192.168.0.0/24** range to configure VLANs and DHCP as described, but you'll need to divide the **192.168.0.0/24** network into smaller subnets. 

Since **192.168.0.0/24** provides a total of 256 IP addresses (0-255), we will break it down into smaller subnets using a **subnet mask** that allows for enough addresses per VLAN. You can use **Variable Length Subnet Masking (VLSM)** to allocate different numbers of IP addresses based on the needs of each VLAN.

Here's a detailed guide on how to do this:

### **Step 1: Subnet Planning**

We’ll divide **192.168.0.0/24** into subnets based on the required number of hosts in each VLAN. You can use **VLSM** to allocate subnets more efficiently.

Let’s assume you need:
- **VLAN 10 (Management)**: 5 hosts
- **VLAN 20 (Sales)**: 8 hosts
- **VLAN 30 (Support)**: 10 hosts

#### **Subnetting Example for VLSM:**

To meet the host requirements, we’ll create subnets with an appropriate number of IP addresses. Each subnet will reserve 2 IP addresses for network and broadcast, leaving the rest for hosts.

- **VLAN 10 (Management)**: Needs 5 hosts ⇒ Allocate 8 addresses (subnet mask /29).
- **VLAN 20 (Sales)**: Needs 8 hosts ⇒ Allocate 16 addresses (subnet mask /28).
- **VLAN 30 (Support)**: Needs 10 hosts ⇒ Allocate 16 addresses (subnet mask /28).

Here’s how the network can be broken down:

| VLAN  | Subnet              | Subnet Mask | IP Range           | Total Hosts |
|-------|---------------------|-------------|--------------------|-------------|
| VLAN 10 (Management) | 192.168.0.0/29    | 255.255.255.248 | 192.168.0.1 - 192.168.0.6 | 6           |
| VLAN 20 (Sales)      | 192.168.0.8/28    | 255.255.255.240 | 192.168.0.9 - 192.168.0.14 | 14           |
| VLAN 30 (Support)    | 192.168.0.16/28   | 255.255.255.240 | 192.168.0.17 - 192.168.0.30 | 14           |

### **Step 2: VLAN Configuration on the Switch**

1. **Create VLANs on the Layer 3 Switch**:
   - The configuration for VLAN creation remains the same as before:
     ```bash
     vlan 10
     name Management
     vlan 20
     name Sales
     vlan 30
     name Support
     ```

2. **Assign Switch Ports to VLANs**:
   - For VLAN 10 (Management) on **FastEthernet 0/1**:
     ```bash
     interface FastEthernet 0/1
     switchport mode access
     switchport access vlan 10
     ```
   - For VLAN 20 (Sales) on **FastEthernet 0/2**:
     ```bash
     interface FastEthernet 0/2
     switchport mode access
     switchport access vlan 20
     ```
   - For VLAN 30 (Support) on **FastEthernet 0/3**:
     ```bash
     interface FastEthernet 0/3
     switchport mode access
     switchport access vlan 30
     ```

3. **Configure Inter-VLAN Routing**:
   You will need **Switch Virtual Interfaces (SVIs)** to route traffic between VLANs:
   - For **VLAN 10 (Management)**:
     ```bash
     interface vlan 10
     ip address 192.168.0.1 255.255.255.248
     no shutdown
     ```
   - For **VLAN 20 (Sales)**:
     ```bash
     interface vlan 20
     ip address 192.168.0.9 255.255.255.240
     no shutdown
     ```
   - For **VLAN 30 (Support)**:
     ```bash
     interface vlan 30
     ip address 192.168.0.17 255.255.255.240
     no shutdown
     ```

4. **Enable IP Routing** on the switch to allow inter-VLAN routing:
   ```bash
   ip routing
   ```

### **Step 3: Configure the DHCP Server**

1. **Static IP for the DHCP Server**:
   - Assign the DHCP server a static IP address in **VLAN 10 (Management)**, for example:
     - IP: `192.168.0.2`
     - Subnet Mask: `255.255.255.248`
     - Gateway: `192.168.0.1` (the SVI of VLAN 10)

2. **DHCP Pool Configuration** for each VLAN:
   - Go to the **DHCP tab** under **Services** on the DHCP server.

   - **DHCP Pool for VLAN 10 (Management)**:
     - Pool Name: `VLAN_10_POOL`
     - Default Gateway: `192.168.0.1`
     - Subnet: `192.168.0.0`
     - Subnet Mask: `255.255.255.248`
     - Start IP: `192.168.0.3`
     - End IP: `192.168.0.6`
     - Add.

   - **DHCP Pool for VLAN 20 (Sales)**:
     - Pool Name: `VLAN_20_POOL`
     - Default Gateway: `192.168.0.9`
     - Subnet: `192.168.0.8`
     - Subnet Mask: `255.255.255.240`
     - Start IP: `192.168.0.10`
     - End IP: `192.168.0.14`
     - Add.

   - **DHCP Pool for VLAN 30 (Support)**:
     - Pool Name: `VLAN_30_POOL`
     - Default Gateway: `192.168.0.17`
     - Subnet: `192.168.0.16`
     - Subnet Mask: `255.255.255.240`
     - Start IP: `192.168.0.18`
     - End IP: `192.168.0.30`
     - Add.

### **Step 4: Configure Hosts for DHCP**

1. **PCs in VLAN 10, 20, and 30**:
   - For each PC, go to **IP Configuration** under **Desktop** and select **DHCP**.

2. The DHCP server will assign the PCs IP addresses from their respective DHCP pools based on the VLAN they are connected to.

### **Step 5: Verify the Configuration**

- **Test IP Assignment**:
   - Check each host to ensure they receive the correct IP address from the appropriate subnet based on their VLAN.

- **Test Inter-VLAN Communication**:
   - Ping across VLANs to verify inter-VLAN routing is working. For example:
     - From a host in **VLAN 10**, ping a host in **VLAN 20**.
     - From a host in **VLAN 30**, ping the DHCP server.

### **Summary**

In this configuration, you used **subnetting within the 192.168.0.0/24 range** by creating smaller subnets using **VLSM**:
- **VLAN 10 (Management)**: 192.168.0.0/29 (6 hosts)
- **VLAN 20 (Sales)**: 192.168.0.8/28 (14 hosts)
- **VLAN 30 (Support)**: 192.168.0.16/28 (14 hosts)

You configured the **Layer 3 switch** for **inter-VLAN routing**, set up the **DHCP server**, and assigned **IP addresses dynamically** to hosts in different VLANs. This setup ensures optimal use of IP addresses and maintains a clean separation between network segments.