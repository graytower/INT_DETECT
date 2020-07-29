# GRAY FAILURES DETECTION AND LOCALIZATION
## Introduction
Network reliability becomes increasingly important in modern data center networks (DCNs). The DCNs are expected to work sustainably under internal failures and assist network operators in troubleshooting them rapidly. However, some network failures will happen silently with packets discarded without producing any explicit notification before causing tremendous damage to the network. To troubleshoot these “gray failures”, in this work, we present a rapid gray failure detection and localization mechanism based on the recently proposed In-band Network Telemetry (INT). Specifically, we leverage simplified INT probe packets to conduct network-wide telemetry to help the servers under ToR switches obtain all the feasible paths between sources and destinations. Once a network failure occurs, the affected thus unavailable paths will immediately be detected and flushed out of the path information table at each server by a timeout mechanism. Hence, servers can proactively perform source routing-based fast traffic reroute to avoid massive packet loss and retain uninterrupted quality of experience. At the meantime, all the aged path entries will be uploaded to a remote controller for centralized failure localization by identifying common path elements. To verify the feasibility of our design, we build a virtual network testbed with software P4 switches and a Redis database. Evaluation shows that our system can successfully detect network gray failures and reroute the affected traffic in no time while complete failure localization within only a few seconds.

## System
The system includes six modules: `bmv2_model`, `controller`, `flow_table`, `p4_source_code`, `packet`, `topology`.

****

### bmv2_model
The bmv2 target used in the network. 
#### simple_switch
Simple_switch is one of the P4 software switch targets.

****

### p4_source_code
The P4 file which defines the packet processing logic of the switches.
#### my_int.py
Line 9-51: the header definition.
Line 57-99: the parse for each type of packets.
Line 113-221: the match-action field. The switch will forward the data packets according to its SR field and add the INT header into the INT probes.
#### my_int.json
The output of compiling `my_int.p4`.
#### Others
Not used.

****

### topology
Create the virtual network.
#### clos.py
Create the clos architecture network with customied scale.
#### p4_mininet.py
The reference for adding P4 switches into the network.

****

### flow_table
Include the flow tables, flow table generator.
#### ./flow_table
The flow tables.
#### ./flow_table/flow_table_gen.py
Generate the flow tables for customized clos architecture topology. And the output files will be in `flow_table`.
#### ./flow_table/command.sh
Dump the flow tables into the P4 switches.
#### ./flow_table/simple_switch_CLI
The control plane of simple switches.

****

### packet
Include the packet sending and receiving scripts.
#### ./send
The packet sending scripts.
#### ./send/send_int_probe.py
Send int probes.
#### ./send/send_udp.py
Send data packets.
#### ./receive
The packet receiving and processing scripts.
#### ./receive/parse.py
Parse the packets.
#### ./receive/receive.py
Receive all packets and use `parse.py` to parse the packets. And store the INT information into the database.
#### ./receive/processor.py
Not used.

****

### controller
The source code of the controller.
#### controller.py
The controller will subscribe the Redis database and catches all the expire events. And use these information for failure localization.

***

## Requisite third parties
P4 development enviornment: behavioral_model, p4c  
Database: Redis. And modify the configuration of redis to enable the unix socket and sub/pub function.

## How to run
First, enter the root direction of the system.
```
cd INT_DETECT
```
Second, start the topology and enter the CLI of Mininet.
```
sudo python topology/clos.py
```
Third, start the controller.
```
sudo python controller/controller.py
```
You can verify the controller by cut down some links by entering some commands in the CLI of mininet, like
```
link p1_l1 p1_t1 down
```
to make the link between switch p1_l1 and switch p1_t1 down.
And then you can the localization result of the failure in the controller.
