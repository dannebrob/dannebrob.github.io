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
1. **Review the ticket**: Read the ticket `ticket.md` found in the folder. 
1. **Solve the ticket and document the changes**: Verify configuration as you go along and document the changes in the `lab-notebook.txt`, you should also include the show commands you use to verify the configuration.
