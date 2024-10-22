
To set up **DHCP with VLANs** in **Cisco Packet Tracer**, you'll need a **Layer 3 switch** (for inter-VLAN routing) and a **DHCP server**. The Layer 3 switch will handle the VLANs and route traffic between them, while the DHCP server will assign IP addresses dynamically to devices in each VLAN.

Here’s a step-by-step guide to set up DHCP with VLANs in **Cisco Packet Tracer**.

### **Step 1: Network Design**
- **Layer 3 Switch**: This switch will create and manage VLANs and route traffic between them.
- **DHCP Server**: This server will assign IP addresses to devices in different VLANs.
- **Hosts**: PCs or other devices that will receive IP addresses from the DHCP server.
  
Assume the following VLAN setup:
- **VLAN 10 (Management)**: 192.168.10.0/24
- **VLAN 20 (Sales)**: 192.168.20.0/24
- **VLAN 30 (Support)**: 192.168.30.0/24

### **Step 2: VLAN Configuration on the Switch**

1. **Add the Layer 3 switch and hosts in Cisco Packet Tracer:**
   - Add a **Cisco 3560 Switch** (Layer 3 switch).
   - Add PCs (or other devices) for each VLAN.

2. **Create VLANs on the Layer 3 Switch:**

   a. **Enter the switch’s command line interface (CLI)**:
   ```bash
   enable
   configure terminal
   ```

   b. **Create VLANs**:
   - For **VLAN 10 (Management)**:
     ```bash
     vlan 10
     name Management
     ```
   - For **VLAN 20 (Sales)**:
     ```bash
     vlan 20
     name Sales
     ```
   - For **VLAN 30 (Support)**:
     ```bash
     vlan 30
     name Support
     ```

3. **Assign Switch Ports to VLANs:**

   a. **Select the interface for each VLAN**:
   - Assign **FastEthernet 0/1** to **VLAN 10 (Management)**:
     ```bash
     interface FastEthernet 0/1
     switchport mode access
     switchport access vlan 10
     ```
   - Assign **FastEthernet 0/2** to **VLAN 20 (Sales)**:
     ```bash
     interface FastEthernet 0/2
     switchport mode access
     switchport access vlan 20
     ```
   - Assign **FastEthernet 0/3** to **VLAN 30 (Support)**:
     ```bash
     interface FastEthernet 0/3
     switchport mode access
     switchport access vlan 30
     ```

4. **Set up Trunk Port for Inter-VLAN Communication** (if required for communication with a router or another switch):
   ```bash
   interface FastEthernet 0/24
   switchport mode trunk
   ```

### **Step 3: Configure Inter-VLAN Routing on the Layer 3 Switch**

To allow communication between the VLANs, the Layer 3 switch will need to route the traffic. This is achieved by creating **SVIs (Switch Virtual Interfaces)** for each VLAN.

1. **Enter global configuration mode**:
   ```bash
   configure terminal
   ```

2. **Create SVIs for each VLAN and assign IP addresses**:
   - For **VLAN 10**:
     ```bash
     interface vlan 10
     ip address 192.168.10.1 255.255.255.0
     no shutdown
     ```
   - For **VLAN 20**:
     ```bash
     interface vlan 20
     ip address 192.168.20.1 255.255.255.0
     no shutdown
     ```
   - For **VLAN 30**:
     ```bash
     interface vlan 30
     ip address 192.168.30.1 255.255.255.0
     no shutdown
     ```

3. **Enable IP routing** on the Layer 3 switch to route traffic between the VLANs:
   ```bash
   ip routing
   ```

### **Step 4: Configure the DHCP Server**

1. **Add the DHCP server to the network**:
   - In **Packet Tracer**, add a server from the devices list and connect it to the switch. Use **FastEthernet 0/24** for the connection.

2. **Assign a static IP to the DHCP server**:
   - Go to the server’s **Desktop tab**, then to **IP Configuration**.
   - Assign the server a static IP in the **Management VLAN (VLAN 10)**, for example:
     - IP: `192.168.10.2`
     - Subnet Mask: `255.255.255.0`
     - Gateway: `192.168.10.1` (the VLAN 10 SVI on the Layer 3 switch)

3. **Configure DHCP Pools**:
   - Go to the **Services tab** of the server, and click on **DHCP**.
   - Create separate DHCP pools for each VLAN:

   - For **VLAN 10 (Management)**:
     - Pool Name: `VLAN_10_POOL`
     - Default Gateway: `192.168.10.1`
     - Subnet: `192.168.10.0`
     - Subnet Mask: `255.255.255.0`
     - DNS Server (optional): `192.168.10.2`
     - Start IP: `192.168.10.10`
     - End IP: `192.168.10.50`
     - Click **Add**.

   - For **VLAN 20 (Sales)**:
     - Pool Name: `VLAN_20_POOL`
     - Default Gateway: `192.168.20.1`
     - Subnet: `192.168.20.0`
     - Subnet Mask: `255.255.255.0`
     - DNS Server (optional): `192.168.10.2`
     - Start IP: `192.168.20.10`
     - End IP: `192.168.20.50`
     - Click **Add**.

   - For **VLAN 30 (Support)**:
     - Pool Name: `VLAN_30_POOL`
     - Default Gateway: `192.168.30.1`
     - Subnet: `192.168.30.0`
     - Subnet Mask: `255.255.255.0`
     - DNS Server (optional): `192.168.10.2`
     - Start IP: `192.168.30.10`
     - End IP: `192.168.30.50`
     - Click **Add**.

### **Step 5: Configure Hosts for DHCP**

1. **Go to the PCs (or other devices)** that are connected to the switch.
2. **Configure them to obtain IP addresses via DHCP**:
   - Click on the **PC** > **Desktop tab** > **IP Configuration**.
   - Select **DHCP**.

### **Step 6: Testing the Configuration**

1. **Verify DHCP assignments**:
   - Check each PC to confirm that it receives an IP address from the correct DHCP pool corresponding to its VLAN.
   - For example:
     - A PC in **VLAN 10** should get an IP address in the range `192.168.10.x`.
     - A PC in **VLAN 20** should get an IP address in the range `192.168.20.x`.

2. **Test inter-VLAN communication**:
   - Ping from one PC in **VLAN 10** to a PC in **VLAN 20** or **VLAN 30** to ensure that inter-VLAN routing is working.
   - Ping the **DHCP server** from all PCs to confirm network connectivity.

### **Summary of Key Configuration Commands**

- **Creating VLANs:**
  ```bash
  vlan <ID>
  name <VLAN_NAME>
  ```

- **Assigning Ports to VLANs:**
  ```bash
  interface FastEthernet <port_number>
  switchport mode access
  switchport access vlan <ID>
  ```

- **Configuring Trunk Port:**
  ```bash
  interface FastEthernet <port_number>
  switchport mode trunk
  ```

- **Creating SVI (Switch Virtual Interfaces):**
  ```bash
  interface vlan <ID>
  ip address <IP_address> <subnet_mask>
  no shutdown
  ```

- **Enable IP Routing:**
  ```bash
  ip routing
  ```

- **Configuring DHCP on the Server:**
  - Use the **DHCP** tab in the **Services** section of the DHCP server and define IP address pools for each VLAN.

By following these steps, you will have successfully set up **DHCP with VLANs** in Cisco Packet Tracer, allowing each VLAN to dynamically receive IP addresses and communicate as needed across VLANs.