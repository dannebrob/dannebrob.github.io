---
title: "OSFP Troubleshooting (CML-bitsize-labs)"
categories: ["03-troubleshooting"]
source_repo: "CML-bitsize-labs"
source_path: "labs/ospf/03-Troubleshooting/OSFP-Troubleshooting.md"
---

This lab is part of the Bitsize series, where you are set out to troubleshoot the network to find out why OSPF are malfunctioning. 

# OSPF Stub Areas
You will find all the labs in the Bitsize series on my GitHub, link to repo: [CML-bitsize-labs](https://github.com/dannebrob/CML-bitsize-labs/tree/main).

## Learning Objectives
- Real-world troubleshooting in a small network
- Verify OSPF configurations.
- Ticket documentation

## Lab Overview
This is a short one, a quick troubleshooting lab to test your skills to find the errors and practice how to handle it like a real-world ticket.

If you need to brush up on the concepts, I have some study notes that you are more than welcome to review, here is the [link](https://github.com/dannebrob/CCNP-study-notes), I'm sure it will help you in your journey towards mastering OSPF, OSPF stub areas and virtual links.

## Design decisions
This lab is designed to give you hands-on experience with troubleshooting OSPF. The lab topology is designed to be simple and easy to understand, while still providing enough complexity to allow you to practice the concepts of OSPF.

## Lab Instructions
1. **Setup**: Clone the lab repository and navigate to the `./ospf/03-Troubleshooting` folder. Review the provided startup configurations for the routers in the lab topology.
2. **Configuration**: Import the `03-OSPF_Troubleshooting_Lab.yaml` file into the CML environment. 
Follow the instructions in the Lab Tasks to troubleshoot the OSPF on all the appropriate routers. Use the CLI to apply the necessary commands and verify your configuration.
3. **Testing**: After the troubleshooting, test the network connectivity to ensure that the hosts are able to comunicate with eachother.

## Lab Tasks
0. **Review the Lab Environment**: Familiarize yourself with the lab environment and the routers involved.
1. **Review the ticket**: Read the ticket `ticket.txt` found in the folder. 
1. **Solve the ticket and document the changes**: Verify configuration as you go along and document the changes in the `lab-notebook.txt`, you should also include the show commands you use to verify the configuration.

## Walkthrough

0. **Review the Lab Environment**: When looking at the lab, you will see five devices. Two hosts (Ubuntu1 and Ubuntu3) in respective subnets, and three routers that make up the OSPF network: R1, R2 and R3.

1. **Review the Ticket**: The ticket describes a real-world issue where East Campus users cannot reach shared resources. This is a vague service desk ticket with no technical details—typical of production incidents.

2. **Verify Basic Connectivity**:
   - From Ubuntu1, attempt to ping Ubuntu3: `ping 192.168.3.10`
   - Expected: Should fail with "Destination host unreachable"
   - This confirms the issue exists

3. **Check OSPF Neighbor Adjacencies**:
   - SSH into R1 and run: `show ip ospf neighbor`
     - Should see R2 (1.1.1.1) in FULL state ✅
   - SSH into R2 and run: `show ip ospf neighbor`
     - Should see R1 (1.1.1.1) in FULL state ✅
     - Should see R3 (3.3.3.3) in EXSTART or EXCHANGE state ❌ (BUG!)
   - SSH into R3 and run: `show ip ospf neighbor`
     - Should see R2 (2.2.2.2) in EXSTART or EXCHANGE state ❌ (BUG!)

4. **Diagnose the R2↔R3 Adjacency Failure**:
   - On R2, run: `show ip ospf interface Ethernet0/1`
     - Check the "Network Type" field
     - Should show: `POINT_TO_MULTIPOINT`
   - On R3, run: `show ip ospf interface Ethernet0/0`
     - Check the "Network Type" field
     - Should show: `BROADCAST` (default)
   - **Root Cause #1 Found**: Network type mismatch prevents adjacency formation

5. **Fix Bug #1 - Network Type Mismatch**:
   - On R3, enter configuration mode:
 R3# configure terminal
 R3(config)# interface Ethernet0/0
 R3(config-if)# ip ospf network broadcast
 R3(config-if)# end
   - Verify the change:
 R3# show ip ospf interface Ethernet0/0
 (Network Type should now show BROADCAST)
   - Wait 30-40 seconds for adjacency to form
   - Verify on R2: `show ip ospf neighbor` (R3 should now be FULL)

6. **Check OSPF Database for Missing Routes**:
   - On R1, run: `show ip ospf database`
     - Look for loopback 3.3.3.3 in Area 0
     - It should now appear (was missing before)
   - Attempt ping again: `ping 192.168.3.10`
     - Still fails with "Destination host unreachable" ❌ (Second bug remains)

7. **Diagnose Missing Routes in OSPF Database**:
   - On R3, check the OSPF configuration:
 R3# show running-config | section router ospf
   - Look for the network statement for the 192.168.3.0/24 subnet
   - Should see something like: `network 192.168.3.0 0.0.255.255 area 2`
   - **Root Cause #2 Found**: Wildcard mask `0.0.255.255` is incorrect for a /24 network (should be `0.0.0.255`)

8. **Fix Bug #2 - Incorrect Wildcard Mask**:
   - On R3, enter configuration mode:
 R3# configure terminal
 R3(config)# router ospf 1
 R3(config-router)# no network 192.168.3.0 0.0.255.255 area 2
 R3(config-router)# network 192.168.3.0 0.0.0.255 area 2
 R3(config-router)# end
   - Save configuration:
 R3# write memory

9. **Verify Both Fixes**:
   - Check R3 configuration:
 R3# show running-config | include network
 (Should show correct wildcard: 0.0.0.255)
   - Check OSPF database on R1:
 R1# show ip ospf database
 (Should now include 3.3.3.3 and 192.168.3.0/24 routes)
   - Check routing table:
 R1# show ip route ospf
 (Should show route to 192.168.3.0/24)

10. **Final Verification - End-to-End Connectivity**:
    - From Ubuntu1, ping Ubuntu3:
  ubuntu1$ ping -c 4 192.168.3.10
  PING 192.168.3.10 (192.168.3.10) 56(84) bytes of data.
  64 bytes from 192.168.3.10: icmp_seq=1 ttl=62 time=4.2 ms
  64 bytes from 192.168.3.10: icmp_seq=2 ttl=62 time=3.8 ms
  64 bytes from 192.168.3.10: icmp_seq=3 ttl=62 time=4.1 ms
  64 bytes from 192.168.3.10: icmp_seq=4 ttl=62 time=3.9 ms
  --- 192.168.3.10 statistics ---
  4 packets transmitted, 4 received, 0% packet loss ✅ SUCCESS
    - From Ubuntu3, ping Ubuntu1:
  ubuntu3$ ping -c 4 192.168.1.10
  (Should also succeed with 0% packet loss)

11. **Document the Solution**:
    - **Bug #1**: R2 Ethernet0/1 configured with `ip ospf network point-to-multipoint` while R3 Ethernet0/0 used default `broadcast` type. This prevented OSPF adjacency formation.
    - **Bug #2**: R3 OSPF network statement used incorrect wildcard mask `0.0.255.255` instead of `0.0.0.255` for the 192.168.3.0/24 subnet, preventing the subnet from being advertised into OSPF.
    - **Resolution**: Changed R3 E0/0 network type to broadcast and corrected the wildcard mask. Both routers now have complete OSPF databases and all subnets are reachable.

12. **Ticket Resolution**:
    - Update ticket with findings
    - Mark as RESOLVED
    - East Campus users can now access shared resources