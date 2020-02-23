
# DIP-SIP Test Plan

### Table of Contests:

- [Overview](#overview)

  * [Scope](#scope)

  * [Testbed](#testbed)

- [Test Cases](#test-cases)

- [Expected Results](#expected-results)

- [Execution Example](#execution-example)


## Overview
The purpose of the test is to validate that HW supports routing of L3 packets with DIP=SIP. 
Two types of UDP packets are going to be sent - IPV4, IPV6. 


Test case assumes that for different types of topologies, we may have different set of interfaces.  
e.g. in case topology is t0 - it collects the information by LAG, and in case its t1 - it collects the information by PORT.


In different types of topologies there can be different settings, so the test needs information about the amount of LAG members, and which port id's to use in order to send and receive traffic. 

#### Scope
The test is targeting a running SONiC system, with basic setup configuration, in order to check support when DIP=SIP. 

#### Testbed

**Supported topologies:** 
t0, t0-16, t0-56, t0-64, t0-64-32, t0-116, t1, t1-lag t1-64-lag, t1-64-lag-clet.


##Test Cases
- Test will gather information about the testbed (e.g. minigraph information, LLDP information,
IPs, src-dst ports).
- PTF container will construct 2 types of packets - data packet and expected packet. 
- After sending the data packet, wait for expected packet.

## Expected Results
Testcase will pass successfully if packet is routed between router interfaces. 

## Execution Example:

`py.test --inventory ../ansible/inventory --host-pattern <host_name> --module-path ../ansible/library/ --testbed <host_name>-<topology> --testbed_file ../ansible/testbed.csv --show-capture=no --capture=no --log-cli-level debug -ra -vvvvv dip_sip/test_dip_sip.py`

