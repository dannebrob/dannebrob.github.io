---
title: "OSPF Summarization Lab (CML-bitsize-labs)"
categories: ["01-summarization"]
source_repo: "CML-bitsize-labs"
source_path: "labs/ospf/01-summarization/OSPF-Summarization-Lab.md"
---

A Cisco Modeling Lab exercise focused on OSPF summarization, where you will learn how to configure and verify OSPF summarization in a network environment. The lab will cover the benefits of summarization, how to implement it on a router, and how to troubleshoot any issues that may arise from its configuration.

# 01- Summarization Lab
You will find all the labs in the Bitsize series on my Github, link to repo: [CML-bitsize-labs](https://github.com/dannebrob/CML-bitsize-labs/tree/main).


## Learning Objectives
- Understand the concept of OSPF summarization and its benefits in reducing routing table size and improving network efficiency.
- Learn how to configure OSPF summarization on a router and verify its functionality.
- Gain hands-on experience with OSPF summarization in a lab environment.
- Develop troubleshooting skills related to OSPF summarization and its effects on network connectivity.

## Lab Overview

Learn more about summarization in my Encore notebook on github: [OSPF Summarization - Encor/CCNP Notebook](https://github.com/dannebrob/CCNP-study-notes)

## Lab Instructions
1. **Setup**: Clone the lab repository and navigate to the `01-summarization` folder. Review the provided startup configurations for the routers in the lab topology.
2. **Configuration**: Import the cml_import.yaml file into the CML environment. 
Follow the instructions in the Lab Tasks to configure OSPF summarization on the appropriate routers. Use the CLI to apply the necessary commands and verify your configuration.
3. **Testing**: After configuring summarization, test the network connectivity and routing tables to ensure that the summarization is working as expected.

## Lab Tasks
1. **Identify the Networks**: Review the network topology and identify the networks that need to be summarized.

R1 is part of area 10 of the OSPF network and is connected to R2 via a transit link. R1 has 3 loopback interfaces (lo1-3) that are part of the ospf area 10 and are advertised into the ospf network. The other loopback interfaces (lo4-6) are part of a external network and are not advertised into the OSPF network.

R1 has the following networks and configurations:
```
Interfaces:
Ethernet0/0: 10.10.0.2/30 #Transit to R2
lo1: 10.10.1.1/30 #Loopback1
lo2: 10.10.2.1/30 #Loopback2
lo3: 10.10.3.1/30 #Loopback3
lo4: 10.11.1.1/30 #Loopback4
lo5: 10.11.2.1/30 #Loopback5
lo6: 10.11.3.1/30 #Loopback6

Static routes in routing table:
10.11.1.0 255.255.255.252 Loopback4
10.11.2.0 255.255.255.252 Loopback5
10.11.3.0 255.255.255.252 Loopback6

Ospf:
router-id: 1.1.1.1
Area 10 - lo1, lo2, lo3
External - lo4, lo5, lo6
```


R2:
```
Ethernet0/0: 10.10.0.1/30 #Transit to R1
Ethernet0/1: 10.0.0.1/30 #Transit to R3

ospf:
router-id: 2.2.2.2
area 10 - ethernet0/0
area 0 - ethernet0/1
```

R3:
```
Ethernet0/0: 10.0.0.2/30 #Transit to R2
Ethernet0/1: 10.20.0.2/30 #Transit to R4

ospf:
router-id: 3.3.3.3
area 0 - ethernet0/0
area 20 - ethernet0/1
```

R4:
```
Ethernet0/0: 10.20.0.1/30 #Transit to R3

ospf: 
router-id: 4.4.4.4
area 20 - ethernet0/0
```

2. **Configure Summarization**: On R1, configure OSPF summarization for the loopback interfaces. Use the appropriate OSPF commands to create a summary route that encompasses all the loopback interfaces.

3. **Verify Configuration**: After configuring summarization, verify that the summary route is being advertised correctly. Use OSPF show commands to check the routing tables on R2 and R3 to ensure that they are receiving the summarized route.

Find these facts:
Q1: What is the summary route that R1 is advertising for the loopback interfaces?
Q2: What is the subnet mask of the summary route?
Q3: Are the individual loopback interfaces still being advertised, or are they being replaced by the summary route? 

Answers will be found below in the walkthrough section, but try to answer them on your own first before checking the walkthrough.

4. **Test Connectivity**: Test the connectivity from R2 and R3 to the loopback interfaces on R1. Use ping and traceroute commands to verify that the summarized route is functioning correctly and that the individual loopback interfaces are reachable.

## Walkthrough
This is a walkthrough of the lab, where you can find the commands and steps to complete the lab tasks. It is recommended to attempt the lab on your own before referring to the walkthrough.

1. **Identify the Networks**: First off, the task is simple: Review the network topology and identify the networks that need to be summarized. That is done with the following command on R1:
```
show ip interface brief
show ip ospf interface
```
I like to first check the interfaces and their IP addresses with `show ip interface brief` to get an overview of the interfaces and their configurations. Then, I use `show ip ospf interface` to see which interfaces are part of the OSPF process and their associated areas.

From the output, we can identify that the loopback interfaces (lo1, lo2, lo3) are part of area 10 and are being advertised into the OSPF network. The other loopback interfaces (lo4, lo5, lo6) are not part of any OSPF area and are not being advertised into the OSPF network. These are the interfaces that we will be summarizing. Make sure they are configured as static routes in the routing table, which is done with the following command:
```
show ip route static
```

We also need to identify which roles the routers role in the OSPF network. This will be displayed with the following command on R1:
```
show ip ospf neighbor
```
This command will show us the OSPF neighbors and their associated areas. From the output, we can see that R1 is connected to R2 in area 10, and R2 is connected to R3 in area 0. R3 is connected to R4 in area 20. This means that R1 is an internal router in area 10, R2 is an ABR between area 10 and area 0, R3 is an ABR between area 0 and area 20, and R4 is an internal router in area 20.


2. **Configure Summarization**: To configure OSPF summarization on R1 for the loopback interfaces, we can use the following command in OSPF router configuration mode:
```
router ospf 1
area 10 range 10.10.0.0 255.255.252.0
```
This command tells R1 to summarize the routes in area 10 into a single summary route of 10.10.0.0/22.

Next is to inject the external routes into the OSPF network. This is done with the following command:
```router ospf 1
redistribute static subnets
```
This command tells R1 to redistribute the static routes (which are the loopback interfaces lo4, lo5, lo6) into the OSPF network as external routes.

3. **Verify Configuration**: After configuring summarization, we can verify that the summary route is being advertised correctly by checking the routing tables on R2 and R3. On R2, we can use the following command:

```
show ip route ospf
```
This command will show us the OSPF routes in the routing table. We should see the summary route of 10.10.0.0/22 and the external routes for the loopback interfaces lo4, lo5, lo6.

Answers to the Find the Facts questions:
Find these facts:
Q1: What is the summary route that R1 is advertising for the loopback interfaces?
   A1: 10.10.0.0/22 
Q2: What is the subnet mask of the summary route?
   A2: 255.255.252.0
>Q3: Are the individual loopback interfaces still being advertised, or are they being replaced by the summary route? 
   A3: The individual loopback interfaces are replaced by the summary route.

## Resources
- [Cisco OSPF Documentation](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13684-12.html)
- [Cisco OSPF Configuration Guide](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13684-12.html)
- [Cisco OSPF Summarization Documentation](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13684-12.html)