# what-just-happened Test Plan

## Table of Contents
- [Introduction](#introduction)
- [Pre test](#pre-test)
- [Test Plan](#test-plan)
    - [Overview](#overview)
    - [Test Cases](#test-cases)
        - [L2 Drops](#l2-drops)
        - [L3 Drops](#l3-drops)


## Introduction
what-just-happened is a feature intended to track lost packets and display the drop reason.

The purpose of this test is:

1. Check that wrong packets are dropped. 
2. Check that the description in ```show what-just-happened``` command output suits the dropped packet.
 
The test will focus on L2, L3 raw packets drops. 

The implementation will be done using PyTest framework.

## Pre test
Before all test cases execution:

1. The test should check whether what-just-happened feature is available and enabled. 

    For this purpose, an ansible module named "feature_facts" will be implemented doing the following:

    - Execute ```show feature``` command on DUT. 
    - Parse output table:
        ```bash
        Feature             Status
        ------------------  --------
        telemetry           enabled
        sflow               disabled
        what-just-happened  enabled
        ```
    - Return a dictionary contains feature_name - enabled/disabled as key - value.
2. The test needs to verify that debug mode is enabled. 

    A pre test function named "get_global_configuration()" will do the following:
    - Execute ```show what-just-happened configuration global``` command on DUT.
    - Parse output table:
        ```bash
        Mode      PCI Bandwidth (%)    Nice Level
        ------  -------------------  ------------
        debug                    50             1
        ```
    - Return the enabled modes. 

3. The test should check on what channels WJH is enabled. 

    A pre test fixture named "get_channel_configuration()" will do the following:
    - Execute ```show what-just-happened configuration channels``` command on DUT. 
    - Parse output table:
        ```bash
        Channel     Type    Polling Interval (s)    Drop Groups
        ----------  ------  ----------------------  -------------
        forwarding  raw     N/A                     L2,L3,Tunnel
        ```
    - Return the drop groups.

A pre test fixture will do the following:
- Call "feature_facts" module and check if 'what-just-happened' appears in the table (availability), and if it is enabled.
- Call get_global_configuration() function and check that debug mode is enabled.
- If one of the checks on previous phases failed, skip the test.




## Test plan

### Overview
The test will use test cases that appear in a file named drop_packets.py, which will be shared with test_drop_counters.py file.
- test_drop_counters.py sends wrong packets and checks that the right counter (e.g. L2, L3) is incremented. 
- test_what_just_happened.py sends wrong packets and checks that the drop reason appears in what-just-happened table. 


For example: 

```test_equal_smac_dmac_drop``` function, generates a TCP packet which equal source mac and destination mac addresses.

After each test case generates the packet, it calls do_test() function which implemented inside test_what_just_happened.py file. 

do_test() function will send the packet, check if packet indeed dropped, and parse the output of 'show what-just-happened' command.

the output should suit to the packet in all table entries, e.g. src_ip, dst_ip, src_mac etc. 

For example, a packet which destination ip address is loopback:

```bash
#  Timestamp              sPort       dPort  VLAN  sMAC               dMAC               EthType  Src IP:Port            Dst IP:Port          IP Proto  Drop   Severity  Drop reason - Recommended action                
                                                                                                                                                        Group                                                            
-- ---------------------- ----------- ------ ----- ------------------ ------------------ -------- ---------------------- -------------------- --------- ------ --------- ------------------------------------------------
1  20/04/13 07:27:42.574  Ethernet60  N/A    N/A   00:11:22:33:44:55  b0:29:3f:a6:33:20  IPv4     1.1.1.1:20 (ftp-data)  127.0.0.1:80 (http)  tcp       L3     Error     Destination IP is loopback address - Bad packet 
                                                                                                                                                                         was received from the peer                      
```

NOTE: 
- In case what-just-happened test will need to drop additional packets which drop_counters won't need, the test cases will be written in test_what_just_happened.py file. 
- Backround traffic can run during the test execution and will not harm the results, as long as the number of packets lost is smaller than 100 - the number of entries in table. 



### Test Cases
-------------------

Before test cases executions, the test will use the output of get_channel_configuration() fixture to make sure the drops can be catched by WJH daemon. 

#### L2 Drops:

- #### test_equal_smac_dmac_drop
    Create a packet with equal SMAC and DMAC and verify it is dropped.

- #### test_multicast_smac_drop
    Create a packet with multicast Source MAC and verify it is dropped.

- #### test_reserved_dmac_drop
    Create packets with reserved Destination MAC and verify they dropped.
    
    used_mac_address:  
    - 01:80:C2:00:00:05 - reserved for future standardization
    - 01:80:C2:00:00:08 - provider Bridge group address

- #### test_not_expected_vlan_tag_drop
    Create a packet tagged with VLAN which does not match ingress port VLAN ID and verify it is dropped.

- #### test_ip_pkt_with_exceeded_mtu
    Create an IP packet with exceeded MTU and verify it is dropped.

#### L3 Drops:
- #### test_dst_ip_is_loopback_addr
    Create a packet with loopback destination IP adress and verify it is dropped.

    This case will be tested on both IPV4 and IPV6. 

- #### test_src_ip_is_loopback_addr
    Create a packet with loopback source IP adress and verify it is dropped.

    This case will be tested on both IPV4 and IPV6. 

- #### test_dst_ip_absent
    Create a packet with absent destination IP address and verify it is dropped.

- #### test_src_ip_is_multicast_addr
    Create a packet with multicast source IP adress and verify it is dropped.

    This case will be tested on both IPV4 and IPV6. 

- #### test_src_ip_is_class_e
    Create a packet with source IP address in class E and verify it is dropped.

- #### test_ip_is_zero_addr
    Create a packet with "0.0.0.0" source or destination IP address and verify it is dropped. 

- #### test_ip_link_local
    Create a packet with link-local address "169.254.0.0/16" and verify it is dropped.

- #### test_loopback_filter
    Create a packet drops by loopback-filter - route to the host with DST IP of received packet exists on received interface. Verify it is dropped. 

- #### test_ip_pkt_with_expired_ttl
    Create an IP packet with TTL=0 and verify it is dropped.

- #### test_non_routable_igmp_pkts
    Create IGMP non-routable packets and verify they are dropped by DUT. 

- #### test_absent_ip_header
    Create a packet with absent IP header and verify it is dropped.

- #### test_unicast_ip_incorrect_eth_dst
    Create a packet with multicast/broadcast ethernet dst and verify it is dropped on L3 interfaces.

- #### test_egress_drop_on_down_link
    Create a packet with RIF link down and verify ingress port are dropped.



----

After each test case, parse the output of ```show what-just-happened``` command, 
and make sure the correct drop reason appears with correct packet information.
