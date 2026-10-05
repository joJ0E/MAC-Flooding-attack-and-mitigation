[[labs]]
# MAC Flooding attack and mitigation
# PART1
# Summary
This report analyses a Layer 2 MAC Flooding attack using Kali Linux and Wireshark. It demonstrates mitigating the threat in Cisco Packet Tracer by implementing Port Security to automatically shut down unauthorized switch ports.
# Evidence
## Part 1: Attack Execution & Traffic Analysis (Kali Linux & Wireshark)

### Step 1: Executing the Attack and Capturing Traffic

- **Objective:** Overflow the Switch CAM Table using synthetic frames and log the generated network traffic.
    
- **Execution:**
1. Initiated the MAC flooding attack using `macof` on interface `eth0`: **sudo macof -i eth0**
2. Simultaneously recorded 20,000 frames using `tcpdump` to a PCAP capture file: 
   **sudo tcpdump -i eth0 -U -c 20000 -w ~/mac-flood-capture.pcap**
![[1.png|478]]
### Step 2: Traffic Analysis in Wireshark

- **Objective:** Inspect packet properties and evaluate the volume of generated MAC addresses.
    
- **Analysis:**
    
    1. Opened `mac-flood-capture.pcap` in Wireshark.
     ![[2 1.png|394]]
        
    2. Navigated to **Statistics → Capture File Properties** to review the total traffic volume and the short duration in which the 20,000 packets were generated. 
        ![[3 1.png|391]]
    3. Navigated to **Statistics → Endpoints** and inspected the **Ethernet** tab.
        ![[4 1.png|400]]
        - **Observation:** Thousands of unique, randomized source MAC addresses were observed, confirming an active MAC Flooding (CAM Table Overflow) attack designed to force the switch into fail-open (hub-like) mode.
# PART2
### Step 1: Topology Setup & Initial State

- **Topology:**
    
    - **Switch:** SW1 (Cisco Switch)
        
    - **Legitimate Hosts:**
        
        - `PC1`: 10.10.10.10
            
        - `PC2`: 10.10.10.11
            
    - **Rogue Host:**
        
        - `PC3`: 10.10.10.12
        - **Populating the CAM Table:** On `PC1`, sent ICMP traffic to `PC2`: **ping 10.10.10.11**
        - Verified SW1 CAM Table: **show mac address-table**
### Step 2: Demonstrating Rogue Device Spoofing

- Connect `PC3` (Rogue PC) to interface `Fa0/1` (previously used by `PC1`) and send ICMP traffic: **ping 10.10.10.11**
- On SW1, execute: **show mac address-table**
- **Observation:** SW1 learned the rogue MAC address on `Fa0/1` without restrictions, proving that default port settings are vulnerable.
![[Screenshot 2026-10-02 200828.png|475]]
### Step 3: Hardening the Switch with Port Security

To secure interface `FastEthernet0/1`, the dynamic MAC address table was cleared and Port Security policies were applied:
**clear mac address-table dynamic**
**configure terminal**
**interface fastEthernet0/1**
**switchport mode access**
**switchport port-security**
**switchport port-security maximum 1**
**switchport port-security violation shutdown**
**end**
**show port-security interface fastEthernet0/1**
![[Screenshot 2026-10-02 201219.png|427]]
### Step 4: Testing & Violation Trigger

1. **Valid Device Traffic:**
    
    Ran `ping 10.10.10.11` from `PC1`.
    
    - Verification: Port status displayed **Secure-up** with `1` authorized MAC address logged.
        
        ![[Screenshot 2026-10-02 200853.png|411]]
        
2. **Unauthorized Device Traffic:**
    
    Connected `PC3` to `Fa0/1` and attempted `ping 10.10.10.11`.
    
    - Verification: Executed `show port-security interface fastEthernet0/1` on SW1.
        
    - **Result:** Port status changed to **Secure-shutdown** , effectively blocking unauthorized access and neutralizing MAC flooding/spoofing attempts.
        
        ![[Screenshot 2026-10-02 201410.png|386]]
        ![[Screenshot 2026-10-02 201424.png|390]]
        ![[Screenshot 2026-10-02 201436.png|399]]
### Step 5: Port Recovery Procedure
**configure terminal**
**interface fastEthernet0/1**
**shutdown**
**no shutdown**
**end**
**Validation:** Verified connectivity by issuing `ping 10.10.10.11` from `PC1`. Ping succeeded and port status returned to normal operational status.
![[Screenshot 2026-10-02 201538.png|401]]
![[Screenshot 2026-10-02 201610.png|399]]
# Recommendation
- **Enable Sticky MAC Learning:** Use `switchport port-security mac-address sticky` to dynamically learn and write authorized MAC addresses to the running configuration.
    
- **Disable Unused Ports:** Shut down all unused switch ports manually (`shutdown`) and place them in an isolated VLAN.
    
- **Implement DHCP Snooping & DAI:** Combine Port Security with DHCP Snooping and Dynamic ARP Inspection (DAI) for comprehensive Layer 2 defense.
