---
title: "OSPF Stub Area Lab (CML-bitsize-labs)"
categories: ["02-ospf-stub-areas"]
source_repo: "CML-bitsize-labs"
source_path: "labs/ospf/02-OSPF-Stub-Areas/OSPF-Stub-Area-Lab.md"
---

This lab is part of the Bitsize series, where we will be covering OSPF Stub Areas and Virtual Links. In this lab, we will be configuring OSPF Stub Areas in a network topology, and implementing Virtual Links to connect non-backbone areas to the backbone area. We will also be verifying our OSPF configurations and testing connectivity between different areas of the network.

# OSPF Stub Areas
You will find all the labs in the Bitsize series on my GitHub, link to repo: [CML-bitsize-labs](https://github.com/dannebrob/CML-bitsize-labs/tree/main).

## Learning Objectives
- Implement OSPF normal areas and Stub Areas in a network topology.
- Implement Virtual Links in OSPF to connect non-backbone areas to the backbone area.
- Verify OSPF configurations.
- Document the OSPF Stub Area configuration in the Lab notebook text file.

## Lab Overview
I'm glad you are here, in this lab we will be configuring OSPF Stub Areas in a network topology. We will also be implementing Virtual Links to connect a non-backbone area to the backbone area, and configuring OSPF to ensure connectivity with the rest of the network.

If you need to brush up on the concepts, I have some study notes that you are more than welcome to review, here is the [link](https://github.com/dannebrob/CCNP-study-notes), I'm sure it will help you in your journey towards mastering OSPF, OSPF stub areas and virtual links.

## Design decisions
This lab is designed to give you hands-on experience with configuring OSPF Stub Areas and Virtual Links. The lab topology is designed to be simple and easy to understand, while still providing enough complexity to allow you to practice the concepts of OSPF Stub Areas and Virtual Links. I have tried to make the lab instructions simple enough to guide you through the configuration process, but I encourage you to not use this as an tutorial and keep you in the "tutorial hell". For the sake of not making the lab too complex, we will be using a simple topology with only a few routers and areas, but it will still provide you with the opportunity to practice the concepts of OSPF Stub Areas and Virtual Links, but what this lacks is real-world complexity, which is something you probably won't find in a real-world scenario. The lab is designed to be completed in a reasonable amount of time, while still providing you with the opportunity to learn and practice the concepts of OSPF Stub Areas and Virtual Links.  

## Lab Instructions
1. **Setup**: Clone the lab repository and navigate to the `02-OSPF-Stub-Areas` folder. Review the provided startup configurations for the routers in the lab topology.
2. **Configuration**: Import the `02-OSPF_Stub_Areas_Lab.yaml` file into the CML environment. 
Follow the instructions in the Lab Tasks to configure OSPF Stub Areas on the appropriate routers. Use the CLI to apply the necessary commands and verify your configuration.
3. **Testing**: After configuring stub areas, test the network connectivity and routing tables to ensure that the stub areas are working as expected.

## Lab Tasks
0. **Review the Lab Environment**: Familiarize yourself with the lab environment and the routers involved.
1. **Configure OSPF area 0-4**: Configure the appropriate routers to be part of the stub areas, ensuring that they are correctly connected to the backbone area (Area 0). One of the areas should be a normal area (not stub), one should be a normal stub area, one should be a totally stub area and one should be a NSSA-TS. Verify configuration as you go along and document the changes in the `lab-notebook.txt`, you should also include the show commands you use to verify the configuration.

2. **Configure Virtual Links**: If necessary, configure virtual links to connect non-backbone areas to the backbone area. Verify the configuration.

4. **Verify Configuration**: Use OSPF commands to verify that the stub areas are correctly configured and that the routers are advertising the correct routes.

5. **Test Connectivity**: Test the connectivity between different areas of the network to ensure that the OSPF Stub Areas are functioning correctly.

## Walkthrough
0. **Review the Lab Environment**: 
We have a total of 8 routers in this lab, R1, R2, R3, R4, R5, R6, R7 and R8. None of the routers are configured with OSPF, but they have at least IP addresses and subnet masks configured on their interfaces. The interfaces connecting the routers are using p2p links (/30).
The HR, IT and SALES LANs are not configured in any other way, they are just connected to the appropriate routers and are assigned IP addresses and subnet masks. The routes have tags with the area they should be configured with. 

1. **Configure OSPF Areas**
- backbone area (Area 0): R1 and R2
<br>
We first need to configure OSPF on R1 and R2, and assign them to Area 0.
Open up the CLI for R1 and R2 and enter the following commands:
<br>

```
R1:
router ospf 1
router-id 1.1.1.1
network 10.10.10.0 0.0.0.3 area 0

R2:
router ospf 1
router-id   2.2.2.2
network 10.10.10.0 0.0.0.3 area 0
```

- Now we need to configure R3, R4 and R5 to be part of Area 1, We will configure Area 1 to be a normal area.
Open up the CLI for R2, R3, R4 and R5 and enter the following commands:
<br>

```
R2:
router ospf 1
network 10.20.20.0 0.0.0.3 area 1
network 10.20.20.4 0.0.0.3 area 1

R3:
router ospf 1
router-id 3.3.3.3
network 10.20.20.8 0.0.0.3 area 1

R4:
router ospf 1
router-id 4.4.4.4
network 10.20.20.12 0.0.0.3 area 1

R5:
router ospf 1
router-id 5.5.5.5
network 10.20.20.8 0.0.0.3 area 1
network 10.20.20.12 0.0.0.3 area 1
```
<br>
- Now we need to configure R5 and R6 to be part of Area 2, and we will configure Area 2 to be a totally stubby area.
Open up the CLI for R5 and R6 and enter the following commands:

```
R5:
router ospf 1
network 10.30.30.0 0.0.0.3 area 2
area 2 stub no-summary

R6:
router ospf 1
router-id 6.6.6.6
network 10.30.30.0 0.0.0.3 area 2
area 2 stub no-summary
```
- We need to configure Area 3 on R3, as a NSSA:
<br>

```
R3:
router ospf 1
network 10.40.40.0 0.0.0.3 area 3
area 3 nssa

R8:
router ospf 1
router-id 8.8.8.8
network 10.40.40.0 0.0.0.3 area 3
area 3 nssa
```
<br>
- And the last area, area 4, needs to be configured on R4, as a NSSA-TS:

```
R4:
router ospf 1
network 10.50.50.0 0.0.0.3 area 4
area 4 nssa no-summary

R7:
router ospf 1
router-id 7.7.7.7
network 10.50.50.0 0.0.0.3 area 4
area 4 nssa no-summary
```
<br>


2. **Configure Virtual Links**: If necessary, configure virtual links to connect non-backbone areas to the backbone area.

Since area 2, 3 and 4 are not directly connected to the backbone area (Area 0), they will need to configure with virtual links between R2 and the respective areas to connect them to Area 0.
Open up the CLI for respective routers and enter the following commands:
```
R2:
router ospf 1
area 1 virtual-link 5.5.5.5
area 1 virtual-link 4.4.4.4
area 1 virtual-link 3.3.3.3

R5:
router ospf 1
area 1 virtual-link 2.2.2.2

R4:
router ospf 1
area 1 virtual-link 2.2.2.2

R3:
router ospf 1
area 1 virtual-link 2.2.2.2
```
This will create a virtual link between the routers, allowing area 2,3 and 4 to be connected to Area 0 through Area 1. It's easy to think of it as a tunnel between the routers, allowing OSPF to exchange routing information between the two areas.
It don't matter if its a physical or virtual link, as long as the OSPF process is configured correctly, the routers will be able to exchange routing information and the network will function as expected.


4. **Verify Configuration**: Use OSPF commands to verify that the stub areas are correctly configured and that the routers are advertising the correct routes.
- You can use the `show ip ospf neighbor` command to verify that the routers are forming adjacencies with their neighbors and that the correct areas are being advertised.
- To confirm that area 2, 3 and 4 are stub areas, you can use the `show ip ospf` command to verify that the areas are configured as stub areas and that the correct routes are being advertised.
- You can also use the `show ip route` command to verify that the correct routes are being advertised and that the routing tables are correct.
- You can also use the `show ip ospf database` command to verify that the OSPF database is correct and that the correct routes are being advertised.
- You can also use the `show ip ospf interface` command to verify that the interfaces are correctly configured and that the correct areas are being advertised on the interfaces.

- You can also use the `show ip ospf virtual-links` command to verify that the virtual links are correctly configured and that they are up and running. The virtual links work because all endpoints are ABRs and they share Area 1 as a normal transit area. Cisco IOS permits this even when the ABRs’ other areas are stub or NSSA types. 

5. **Test Connectivity**: Test the connectivity between different areas of the network to ensure that the OSPF Stub Areas are functioning correctly.
- You can use the `ping` command to test connectivity between different areas of the network. 
Ping from R1 to R6 should work, as well as pinging from R3 to R4, and from R5 to R6.
Ping from R1 to R7 should also work
Ping from R1 to R8 should also work
- You can also use the `show ip route` command to verify that the correct routes are being advertised and that the routing tables are correct.

