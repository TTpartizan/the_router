# Configuring TheRouter

There are three groups of configuration options and commands:

 * command line options
 * configuration file commands
 * rcli commands
	
## Command line options
Command line options can be modified by editing the run script /usr/local/sbin/router_run.sh.
Most of them are DPDK EAL command line options, therefore you can
refer to DPDK documentation <a href="http://dpdk.org/doc/guides/testpmd_app_ug/run_app.html?highlight=eal%20options">EAL Command-line Options</a>
for detailed description of them.

Before running TheRouter you must check the following options and use your own values depending on the hardware you use:
 * -c
 
Set the hexadecimal bitmask of the cores to run on.
	
 * --lcores
 
 !  Note: the_router.a0.01.tar.gz version has supported only a 4 cores cpu. You must use at least a 4 cores cpu and then configure
 TheRouter to use 4 lcores using --lcores='0@0,1@1,2@2,3@3' parameter
 
 
Map lcore set to physical cpu set

 * -n
 
Set the number of memory channels to use.

 * -w
 
Add a PCI device in white list.

Example of the startup script cmd options:

	the_router --proc-type=primary -c 0xF --lcores='0@0,1@1,2@2,3@3' --syslog='daemon' -n2 -w 0000:01:00.0 -w 0000:01:00.1 -- -c $1 -d

Note:
Lcore 0 will be used for TheRouter's control plane task and can be shared with any linux tasks.
The other cores will be used in TheRouter's data plane process. You should isolate them during the linux starup process by using
linux kernel command line parameters isolcpus. Otherwise, performance of TheRouter's working threads
could be very low due the context switching.

## Configuration file options

This options are stored in the /etc/router.conf file.

Configuration file consists of the sections:
 * startup
 * runtime

Each section contains commands. Everything on the same line is considered as a single command.
Symbol # is used to comment a whole line.

	startup {
		startup_command_1
		startup_command_2
		...
		startup_command_n
	}
	
	
	runtime {
		runtime_command_1
		runtime_command_2
		...
		runtime_command_n
	}

### Configuration file example

	startup {
	  # total number of mbufs
	  sysctl set mbuf 8192
	
	  port 0 mtu 1500 tpid 0x8100 state enabled
	  port 1 mtu 1500 tpid 0x8100 state enabled
	
	  rx_queue port 0 queue 0 lcore 1
	  rx_queue port 0 queue 1 lcore 2
	  rx_queue port 0 queue 2 lcore 3
	
	  rx_queue port 1 queue 0 lcore 3
	  rx_queue port 1 queue 1 lcore 2
	  rx_queue port 1 queue 2 lcore 1
	
	  sysctl set global_packet_counters 1
	
	#  sysctl set arp_cache_timeout 300
	}

	runtime {
	  vif add name p0 port 1 type untagged
	  ip addr add 10.0.0.1/24 dev p0
	
	  vif add name p1 port 0 type untagged
	  ip addr add 10.0.1.1/24 dev p1
	
	  ip route add 0.0.0.0/0 via 10.0.1.2 src 10.0.1.1
	
	  npf load "/etc/npf.conf"
	}

### NPF configaration file example

	group default {
	  pass final on p0 all
	  pass final on p1 all
	}

## Startup commands

Startup commands are the commands that are used to initilize TheRouter's susbystem and properties 
that can't be modified once TheRouter have started. This commands can only be used in
the startup section of the configuration file and can't be used by the rcli interface.

 * port

		port <dpdk_port_number> mtu <mtu_size> tpid <tpid_value> state enabled
 
 * rx_queue
 
	 	rx_queue port <dpdk_port_number> queue <queue_number> lcore <lcore>
 	
 * sysctl

		sysctl set <name> <value>
 
 * npf load

	 	npf load "<path_to_npf_configuration_file>"

	 
 Note: that you should enclose path to file with '"'.

## Runtime commands 
Runtime commands are the commands that can be either executed via rcli interface or used in the runtime section of
the configuration file.

### ip addr
 * ip addr add

		rcli ip addr add <net>/<mask> dev <vif_name>
	
 * ip addr del

		rcli ip addr del <net>/<mask> dev <vif_name>

 * sh ip addr  

		rcli sh ip addr

### ip route tables
 * ip route table add

		rcli ip route table add <route_table_name>
		
 * ip route table del

		rcli ip route table del <route_table_name>
	
 * rcli sh ip route tables

		rcli rcli sh ip route tables

### U32 sets
 * u32set create

		rcli u32set create <u32set_name> size <size> bucket_size <bucket_size>
		
 * u32set destroy

		rcli u32set destroy <u32set_name>

 * ipset add

		rcli ipset add <u32set_name> <ipv4>
 
 * ipset del

		rcli ipset del <u32set_name> <ipv4>
 
 * ipset test

		rcli ipset test <u32set_name> <ipv4>
		
 * l2set add

		rcli l2set add <u32set_name> port <port_number> svid <svid> cvid <cvid>

 * l2set del

		rcli l2set del <u32set_name> port <port_number> svid <svid> cvid <cvid>
		
 * l2set test

		rcli l2set test <u32set_name> port <port_number> svid <svid> cvid <cvid>
		

### PBR rules
 * sh ip pbr rules

		rcli sh ip pbr rules
		
 * ip pbr rule add

		ip pbr rule add prio <prio_num> u32set <u32set_name> type "ip" table <route_table_name>

		ip pbr rule add prio <prio_num> u32set <u32set_name> type "l2" table <route_table_name>
		
		rcli ip pbr rule add prio <prio_num> from <net/mask> <route_table_name>
		
 * ip pbr rule del

		rcli ip pbr rule del prio <prio_num>

 * ip pbr flush

		rcli ip pbr flush

	
### ip route
 * ip route add

		rcli ip route add <net>/<mask> dev <vif_name> src <src_ip> [table <table_name>]
		
	or
	
		ip route add <net>/<mask> via <gw_ip> src <src_ip> [table <table_name>]
		
	or
	
		ip route add <net>/<mask> unreachable [table <table_name>]
   
 * ip route del
 
		ip route del <net/mask> [table <table_name>]
   
 * sh ip route
 
		rcli sh ip route
   
### vif
 * vif add

		rcli vif add name <name> port <port_num> type <type> [svid <svid>] [cvid <cvid>] [flags <flag1,flag2...>]

   Type parameter can take one of the following values:
	 - untagged
     - dot1q
     - qinq
   
   Flags:
     - npf_on
	 - kni
	 - proxy_arp
	 - flow_acct
   	   
 * vif del

		rcli vif del <name>

 * sh vif

		rcli sh vif
   
 * sh vif counters

		rcli sh vif counters
   
 * clear vif counters

		rcli clear vif counters

### arp
   
 * arp add

		rcli arp add <ip> <mac> dev <vif_name> [static]

 * arp del
 
		rcli arp del <ip> dev <vif_name>
    
 * sh arp cache
 
		rcli sh arp cache

### sysctl

 * sysctl set

		rcli sysctl set <name> <value>

 * sysctl set

		rcli sysctl get <name> <value>

### ping

 * ping

		rcli ping --help
		Usage: ping [-c,--count count] [-i,--interval interval_in_ms] [-s icmp_payload_size]
		[-f,--dont_frag] [-a,--source_address ip_source_address] [-w,--nowait]
		[-h,--help] destination

### NPF
 * sh npf conndb size

		rcli sh npf conndb size
   
 * sh npf conndb summary

		rcli sh npf conndb summary

 * sh npf stat

		rcli sh npf stat

 * npf clear stat

		rcli npf clear stat

#### NPF sysctl variables controlling connection tracking state timeouts

	* NPF_TCPS_CLOSED
	* NPF_TCPS_SYN_SENT
	* NPF_TCPS_SIMSYN_SENT
	* NPF_TCPS_SYN_RECEIVED
	* NPF_TCPS_ESTABLISHED
	* NPF_TCPS_FIN_SENT
	* NPF_TCPS_FIN_RECEIVED
	* NPF_TCPS_CLOSE_WAIT
	* NPF_TCPS_FIN_WAIT
	* NPF_TCPS_CLOSING
	* NPF_TCPS_LAST_ACK
	* NPF_TCPS_TIME_WAIT
	* NPF_ANY_CONN_CLOSED
	* NPF_ANY_CONN_NEW
	* NPF_ANY_CONN_ESTABLISHED

Example:
	
	startup {
		...
	
		# any protocol timeouts (UDP)
		sysctl set NPF_ANY_CONN_CLOSED 2
		sysctl set NPF_ANY_CONN_NEW 30
		sysctl set NPF_ANY_CONN_ESTABLISHED 60
		
		# TCP timeouts
		sysctl set NPF_TCPS_CLOSED 10
		sysctl set NPF_TCPS_SYN_SENT 30
		sysctl set NPF_TCPS_SIMSYN_SENT 30
		sysctl set NPF_TCPS_SYN_RECEIVED 60
		sysctl set NPF_TCPS_ESTABLISHED 600
		sysctl set NPF_TCPS_FIN_SENT 240
		sysctl set NPF_TCPS_FIN_RECEIVED 240
		sysctl set NPF_TCPS_CLOSE_WAIT 45
		sysctl set NPF_TCPS_FIN_WAIT 60
		sysctl set NPF_TCPS_CLOSING 30
		sysctl set NPF_TCPS_LAST_ACK 30
		sysctl set NPF_TCPS_TIME_WAIT 120
	
		...
	}

### NAT events logging via IPFIX

 * Enable nat events 

		sysctl set ipfix_nat_events 1


 * Setup ipfix collector

		ipfix_collector addr 192.168.20.2

### Other commands

 * shutdown

		rcli shutdown

### Router statistic commands

 * sh port ext stat

		rcli sh port ext stat

 * sh port stat
 
		rcli sh port stat

 * sh cmbuf stats

		rcli sh cmbuf stats

 * sh mbuf stats

		rcli sh mbuf stats

 * sh stats

		rcli sh stats
   
 * clear stats
 
		rcli clear stats

## IPv6

 * ipv6 enable

		ipv6 enable dev <vif_name>

  Enables IPv6 protocol on an interface, create link-local address using eui-64 scheme

 * ipv6 disable

		ipv6 disable dev <vif_name>

  Disables IPv6 protocol on the interface. Delete all addresses and routes depending on an interface

* sh ipv6 addr

		sh ipv6 addr

  Outputs ipv6 addresses assignes to interfaces

 * ipv6 addr add eui-64

		ipv6 addr add eui-64 <prefix>/<length> eui-64 dev <vif_name>

  Generates an address using prefix and interface id (EUI64 scheme) and assign the address to
an interface

 * ipv6 addr add

		ipv6 addr add <address>/<length> dev <vif_name>

  Assigns the given ipv6 address to an interface

 * ipv6 addr link-local

		ipv6 addr link-local <address> dev <vif_name>

  Assigns the given ipv6 link-local address to an interface

 * ipv6 addr link-local eui-64

		ipv6 addr link-local eui-64 dev <vif_name>

  Generates a link-local address using the eui-64 scheme and assigns it to an interface

 * ipv6 addr auto

		ipv6 addr auto dev <vif_name> enable|disable

  Enables SLAAC client for an interface. Once enabled the router starts assigning dynamic 
  ipv6 addresses at an interface based on the information received in Router Advertisement messages

 * ipv6 addr del

		ipv6 addr del <address or prefix>/<length> dev <vif_name>

  Deletes an address from an interface

 * sh ipv6 route

		sh ipv6 route

  Outputs ipv6 routing table

 * ipv6 route add

		ipv6 route add <prefix/prefix-length> dev <vif_name>

  Adds a connected route to a prefix into the main routing table

 * ipv6 route add

		ipv6 route add <prefix/length> dev <vif_name> via <ipv6-address> [table <ipv6_routing_table>]

  Adds a route to a prefix via a gateway with given address into a routing table

 * ipv6 route add

		ipv6 route add ::/0 via <ipv6-address> [table <ipv6_routing_table>]

  Adds the default route into a routing table

 * ipv6 route add

		ipv6 route add <prefix/length> unreachable [table <ipv6_routing_table>]

  Adds an unrechable route into a routing table

 * ipv6 route del

		ipv6 route del <prefix/length> [table <ipv6_routing_table>]

  Deletes a route from a routing table

 * ipv6 route default auto

		ipv6 route default auto dev <vif_name> enable|disable

  Enables intallation of default routes based on the information in Router Advertisement messages.
  Once enabled the router will create the default route based on the first RA message received on the interface
  and will associate a timer with that route. The timer is set to RA.lifetime value. When it expires the default
  route will be deleted and the router will install a new default route based on another RA message received at the interface.

 * ipv6 route table add

		ipv6 route table add <route table name>

  Add an ipv6 route table to the FIB

 * ipv6 route table del

		ipv6 route table del <route table name>

  Deletes an ipv6 route table from the FIB

 * ipv6 pbr rule add prefix

		ipv6 pbr rule add prio <rule priority number> from <ipv6 prefix/length> table <route table name>

  Adds an ipv6 pbr rule to the rule list at the position "rule priority number"

 * ipv6 pbr rule add set

		ipv6 pbr rule add prio <rule priority number> u32set <set name> <set type> table <route table name>

  Adds an ipv6 pbr rule to the rule list at the position "rule priority number".
  Only "l2" set type value is supported so far. 

 * ipv6 pbr rule del

		ipv6 pbr rule del prio <rule priority number>

  Deletes an ipv6 pbr rule from the rule list from the position "rule priority number"

 * ipv6 nd ra

		ipv6 nd ra enable|disable dev <vif_name>

  Enables or disables Router Advertisements at an interface.
  If disabled router will not transmit Router Advertisement messages at an interface
  and will not answer to Router Solicitation messages

 * sh ipv6 arp

		sh ipv6 arp

  Outputs ipv6 neighbor cache entries

 * ipv6 arp add

		ipv6 arp add <ipv6-address> <mac-address> dev <vif_name> [static]

  Creates or alters an ipv6 neighbor cache entry

 * ipv6 arp del

		ipv6 arp del <ipv6-address> dev <vif_name>

  Deletes an ipv6 neighbor cache entry

 * icmp6 error msg

		icmp6 error msg type <number> code <number> enable|disable

  Enables or disables generation of an icmp messages with the given type and code

 * sh icmp6 error msg

		sh icmp6 error msg type <number> code <number>

  Outputs state of icmp error message

 * ipv6 nd ra lifetime

		ipv6 nd ra lifetime <number> dev <vif_name>

  Configures lifetime field value of Router Advertisement messages sent 
  from an interface

 * ipv6 nd ra interval

		ipv6 nd ra interval <min_number> <max_number> dev <vif_name>

  Configures the MinRtrAdvInterval and MaxRtrAdvInterval values (seconds)
  See https://tools.ietf.org/html/rfc4861#page-40
  6.2.1.  Router Configuration Variables

 * ipv6 nd ra reachable

		ipv6 nd ra reachable <number> dev <vif_name>

  Configures the value to be placed in the Reachable Time field
  in the Router Advertisement messages sent by the router.
  The value zero means unspecified (by this router).  
  MUST be no greater than 3,600,000 milliseconds (1 hour).  

 * ipv6 nd ra retrans_timer

		ipv6 nd ra retrans_timer <number> dev <vif_name>

  Configures the value to be placed in the Retrans Timer field
  in the Router Advertisement messages sent by the router.  
  The value zero means unspecified (by this router).

 * ipv6 nd ra hop_limit

		ipv6 nd ra hop_limit <number> dev <vif_name>

  Configures the default value to be placed in the Cur Hop Limit
  field in the Router Advertisement messages sent by
  the router.  The value should be set to the current
  diameter of the Internet.  The value zero means
  unspecified (by this router).

 * ipv6 nd ra prefix add|update

		ipv6 nd ra prefix add|update <prefix/length> [valid_lt <number>] 
		  [preferred_lt <number>] [flags O,A] dev <vif_name>

  Adds or updates a prefix to/in Router Advertisement messages sent from an interface
  
 * ipv6 nd ra prefix del

		ipv6 nd ra prefix add <prefix/length> dev <vif_name>

  Deletes a prefix from Router Advertisement messages sent from an interface		

## IPv6 sysctl variables

 * nd_retrans_timer

  The time in milliseconds between retransmissions of Neighbor
  Solicitation messages to a neighbor when
  resolving the address or when probing the
  reachability of a neighbor.

 * fib6_max_route_tables

  Max number of ipv6 routing tables

 * fib6_max_routes

  Max number of ipv6 route entries

 * fib6_max_next_hops

  Max number of ipv6 next hop entries

 * fib6_max_lpm_tbl8

  Max number of lpm6 tbl8. See https://doc.dpdk.org/guides/prog_guide/lpm6_lib.html

 * max_num_solicited_node_addrs

  Max number of solicited node addresses

 * nd_neighbor_cache_size

  ipv6 neighbor cache size

 * nd_neighbor_cache_entry_ttl

  ipv6 neigbour cache entry time to live. Seconds.

 * icmp6_packet_rate

  Icmpv6 error transmission rate in packets per seconds.

 * icmp6_transmission_burst

  Max number of icmpv6 error messages that could be send at once

 * icmp6_num_buckets

  Num icmp6 buckets

 * max_rtr_solicitation_delay

  number of seconds to delay the transmission of router solicitation messages

 * dad_attempts

  Number of attempts for Duplicate address detection algorithm

## IPV6 VRRP version 3

 * vrrp create group

		vrrp create group <vrrp_id> dev <vif_name> address-family af_ipv6 version 3

  Creates vrrp group

 * vrrp group ipv6 add

		vrrp group <vrrp_id> dev <vif_name> ipv6 add <link-local ipv6 address>

  Setup or change ipv6 link-local address of a vrrp3 ipv6 group
  
 * vrrp group ipv6 add

		vrrp group <vrrp_id> dev <vif_name> ipv6 add <ipv6_address> secondary

  Add secondary ipv6 global address to a vrrp3 ipv6 group
  
 * vrrp group ipv6 del

		vrrp group <vrrp_id> dev <vif_name> ipv6 del <ipv6_address> secondary

  Del secondary ipv6 global address from a vrrp3 ipv6 group

 * vrrp group prio

		vrrp group <vrrp_id> dev <vif_name> prio <value>

  Change priority of a vrrp group

 * vrrp group advert_int

		vrrp group <vrrp_id> dev <vif_name> advert_int <value>

  Change advertisement transmission interval (cetiseconds, 100 centiseconds == 1 sec) of a vrrp group

 * vrrp group accept_mode

		vrrp group <vrrp_id> dev <vif_name> accept_mode <on|off>

  Change accept_mode of a vrrp group

 * vrrp group preempt_mode

		vrrp group <vrrp_id> dev <vif_name> preempt_mode <on|off>

  Change preempt_mode of a vrrp group
  
 * sh vrrp

		sh vrrp

  Shows vrrp group information
  
 * vrrp group del 

		vrrp del group <vrrp_id> dev <vif_name>

  Deletes vrrp group
  
 * vrrp group nd ra enable/disable

		vrrp group <vrrp_id> dev <vif_name> nd ra enable|disable

  Enables or disables transmission of ND Router Advertisement messages for a VRRP IPV6 group

 * vrrp group nd ra lifetime

		vrrp group <vrrp_id> dev <vif_name> nd ra lifetime <value>

  Configures lifetime field value of Router Advertisement messages sent 
  for a VRRP IPV6 group

 * vrrp group nd ra interval

		vrrp group <vrrp_id> dev <vif_name> nd ra interval <min_number> <max_number>

  Configures the MinRtrAdvInterval and MaxRtrAdvInterval values (seconds)
  See https://tools.ietf.org/html/rfc4861#page-40
  6.2.1.  Router Configuration Variables

 * vrrp group nd ra reachable

		vrrp group <vrrp_id> dev <vif_name> nd ra reachable <number>

  Configures the value to be placed in the Reachable Time field
  in the Router Advertisement messages sent by the router.
  The value zero means unspecified (by this router).  
  MUST be no greater than 3,600,000 milliseconds (1 hour).
  
 * vrrp group nd ra retrans_timer

		vrrp group <vrrp_id> dev <vif_name> nd ra retrans_timer <number>

  Configures the value to be placed in the Retrans Timer field
  in the Router Advertisement messages sent by the router.  
  The value zero means unspecified (by this router).

 * vrrp group nd ra hop_limit

		vrrp group <vrrp_id> dev <vif_name> nd ra hop_limit <number>

  Configures the default value to be placed in the Cur Hop Limit
  field in the Router Advertisement messages sent by
  the router.  The value should be set to the current
  diameter of the Internet.  The value zero means
  unspecified (by this router).

 * vrrp group nd ra prefix add|update

		vrrp group <vrrp_id> dev <vif_name> nd ra prefix add|update <prefix/length> [valid_lt <number>] 
		  [preferred_lt <number>] [flags O,A] dev <vif_name>

  Adds or updates a prefix to/in Router Advertisement messages sent for a VRRP IPV6 group
  
 * vrrp group nd ra prefix del

		vrrp group <vrrp_id> dev <vif_name>  nd ra prefix add <prefix/length> dev <vif_name>

  Deletes a prefix from Router Advertisement messages sent for a VRRP IPV6 group

## Flow accounting (IPFIX)

### sysctl variables

use 'sysctl get <varname>' or 'sysctl set <varname> <varvalue>' command
to view or modity a sysctl variable. 

 * flow_acct

    Flow account state.

		0 - disabled
		1 - enabled

 * flow_acct_dropped_pkts

    If enabled do flow accounting for dropped packets.

		0 - disabled
		1 - enabled

 * flow_acct_idle_timeout

    Idle timeout of a flow, secs. When the idle timeout expires an idle flow is exported.

 * flow_acct_active_timeout

    active timeout of a flow, secs. When the active timeout expires an active flow is exported.

 * flow_ipv4_max

    maximum number of concurrent ipv4 flows entries

 * flow_ipv6_max

    maximum number of concurrent ipv6 flows entries

 * flow_ipv4_worker_max

    maximum number of ipv4 flows entries that a single lcore (worker) can process
    concurrently

 * flow_ipv6_worker_max

    maximum number of ipv6 flows entries that a single lcore (worker) can process
    concurrently

## rcli commands

 * flow ipfix_collector

		flow ipfix_collector addr <ipv4 address> [port <portnumber>]

  Configures flow accounting ipfix collector address and port. Default port value is 4739.

 * sh flow stat

		sh flow stat

  Show flow accounting statistic counters.


## Enabling flow accounting on an interface
To enable flow accounting on a particular VIF use VIF flag "flow_acct". For example:

	vif add name v3 port 2 type dot1q cvid 3 flags npf_on, kni, flow_acct

## Access control lists (ACL)

Access control lists can be used to filter traffic incoming to an interface (ingress) or
outging from an interface. Multiple ACL can be applied to the same interface and a single ACL
can be applied to many interfaces. 

Each interface has two ACL list: ingress and egress. An interface ACL list stores ACL
in sorted order. The position of ACL in the list is defined by a priority specified by a user.

An ACL, in turn, consists of rules. The position of a rule in an ACL is also defined by priority.

When a packet comes into an interface it is compared to the rules of ACLs from the ingress list,
when a packet is transmitted from an interface it is compared to the rules of ACLs from the egress list.
First, a packet is compared to the rules of the ACL with the minimum priority. Then the process 
goes to the next ACL with greater priority.

When a packet is matched to an ACL rule, the ACL process is stopped and the action defined
by the ACL is taken to the packet. It could be a drop or permit action.

If a packet is not matched to any ACL rule the process goes to the next ACL in the list.
If there are no more ACL in the list, then the action opposite to the action defined by ACL is taken.
For example, if a packet is not matched to any rule of a deny ACL, then the packet is permitted.
And when a packet isn't matched to any rule of a permit ACL the packet is dropped.
So, the last ACL in the list defines the fate of a packet when no matches are found.
Note that empty ACLs are not included into interfaces list of ACL and won't be taken
into account when a packet isn't matched to any ACL rules. 

## RCLI commands

### vif acl create

		vif acl create aclid <acl_id> type <acl_type> <action>

Creates new access control list.

Parameters:
 - acl_id - unique numeric identificator
 - acl_type - type. It can be one of the following types:
   - ipv4_tuple - define an ACL that can classify packets using combination 
of following fields: protocol type, ipv4 source address, ipv4 destination address, 
l4 source port, l4 destination port
   - ipv6_tuple - define an ACL that can classify packets using combination 
of following fields: protocol type, ipv6 source address, ipv6 destination address, 
l4 source port, l4 destination port
 - action: action to take for a packet when a match is found. It can be one of the following
 values:
   - deny - drop a packet when a match is found
   - permit - permit a packet when a match is found

Example:

		vif acl create aclid 10 type ipv6_tuple deny


### vif acl destroy

		vif acl destroy aclid <acl_id>

Deletes an access control list.

Parameters:
 - acl_id - unique numeric identificator of a ACL to delete

Example:

		vif acl destroy aclid 10

### vif acl add

		vif acl add dev <vif_name> dir <direction> aclid <acl_id> prio <prio>

Use an ACL on an interface. Command adds an ACL to the ingress or egress list of ACLs of an interface
at position number prio.

Parameters:
 - vif_name - name of an interface to add an ACL to
 - direction - specifies the interface list of ACLs to add an ACL to.
 Can be one the two values: ingress or egress
 - acl_id - numeric identificator of the ACL to add to an interface
 - prio - position in the interface's ACL list to put an ACL to
 
Example:

		vif acl add dev v5 dir ingress aclid 10 prio 30

### vif acl del

		vif acl del dev <vif_name> dir <direction> aclid <acl_id>

Stop using an ACL on an interface. Command deletes an ACL
from the ingress or egress list of ACLs of an interface.

Parameters:
 - vif_name - name of an interface to delete an ACL from
 - direction - specifies the interface list of ACLs to delete an ACL from.
 Can be one the two values: ingress or egress
 - acl_id - numeric identificator of the ACL to delete from an interface
 
Example:

		vif acl del dev v5 dir ingress aclid 10

### vif acl mod

		vif acl modify dev <vif_name> dir <direction> aclid <acl_id> prio <prio>

Changes the position of an ACL in the list of ACLs on an interface

Parameters:
 - vif_name - name of an interface
 - direction - specifies the interface list of ACLs.
 Can be one the two values: ingress or egress
 - acl_id - numeric identificator of the ACL to modify
 - prio - new position of an ACL
 
Example:

		vif acl modify dev v5 dir ingress aclid 10 prio 40

### vif acl flush

		vif acl flush aclid <acl_id>

Deletes all rules from an ACL.

Parameters:
 - acl_id - numeric identificator of the ACL to delete rules from
 
Example:

		vif acl flush aclid 10

### vif acl rule add

		vif acl rule <ip_version> add aclid <acl_id> prio <prio> [proto <protocol_number>]
		[src <src_prefix>] [dst <dst_prefix>] [sport <src_port_range>] [dport <dst_port_range>] 

Adds a rule to an ACL.

Parameters:
 - ip_version - version of the IP protocol. Can be on of two values: ipv4 or ipv6
 - acl_id - numeric identificator of an ACL to add a rule to
 - prio - position of a rule in the ACL
 - proto - ip protocol number
 - src_prefix - source ip prefix
 - dst_prefix - destination ip prefix
 - src_port_range - l4 source port range: for example: 8080 8090
 - dst_port_range - l4 destination port range
 
Examples:

		vif acl rule ipv4 add aclid 11 prio 21 proto 6 src 10.1.0.0/24 dst 10.2.0.0/24 sport 10 20 dport 80
		or
		vif acl rule ipv4 add aclid 11 prio 21 dst 10.1.0.0/24 dport 80
		or
		vif acl rule ipv6 add aclid 10 prio 20 dst 2a00:1450:400c:c07::8b dport 80

### vif acl rule del

		vif acl rule <ip_version> del aclid <acl_id> prio <prio>

Deletes a rule from an ACL.

Parameters:
 - ip_version - version of the IP protocol. Can be on of two values: ipv4 or ipv6
 - acl_id - numeric identificator of an ACL to add a rule to
 - prio - position of a rule in the ACL
 
Example:

		vif acl rule ipv4 del aclid 11 prio 21

### vif acl rule mod

		vif acl rule <ip_version> modify aclid <acl_id> prio <prio> [proto <protocol_number>]
		[src <src_prefix>] [dst <dst_prefix>] [sport <src_port_range>] [dport <dst_port_range>] 

Modify a rule in an ACL.

Parameters:
 - ip_version - version of the IP protocol. Can be on of two values: ipv4 or ipv6
 - acl_id - numeric identificator of an ACL to add a rule to
 - prio - position of a rule in the ACL
 - proto - ip protocol number
 - src_prefix - source ip prefix
 - dst_prefix - destination ip prefix
 - src_port_range - l4 source port range: for example: 8080 8090
 - dst_port_range - l4 destination port range
 
Examples:

		vif acl rule ipv4 modify aclid 11 prio 21 proto 6 src 10.1.0.0/24 dst 10.2.0.0/24 sport 10 20 dport 80
		or
		vif acl rule ipv4 modify aclid 11 prio 21 dst 10.1.0.0/24 dport 80
		or
		vif acl rule ipv6 modify aclid 10 prio 20 dst 2a00:1450:400c:c07::8b dport 80
		
### sh vif acl rules

		sh vif acl rules aclid <acl_id>

Shows ACL.

Parameters:
 - acl_id - numeric identificator of an ACL to show

 
Example:

		h5 ~ # rcli sh vif acl rules aclid 11
		acl id 11, type ipv4_tuple, action deny, num rules 1
		--
		prio 21, proto any, src any, dst 10.1.1.0/24, sport any, dport 81

### sh vif

		sh vif

Shows ACL interface.
 
Example:

		h5 ~ # rcli sh vif
		vif v5, id 3
		  port 0, vlan 0.5, encapsulation dot1q
		  mac address 00:1B:21:A3:0C:88
		  NPF index 12
		  CAR ingress not set
		      egress not set
		  ACL ingress prio 30 acl 10, prio 40 acl 11
		      egress not set

## PPPoE subscribers

### sh pppoe subsc

	sh pppoe subsc

Output all connected/online pppoe subscribers

Example:

	h5 ~ # $rvrf rcli sh pppoe subsc
	vif_id  mac     session_id      ip addr mtu     ingress cir     egress cir      tx_pkts rx_pkts
	9       60:A4:4C:41:0A:24       6       10.11.1.1       1480    50      55      0       0

### pppoe disconnect

	pppoe disconnect <pppoe_vif_id>

Disconnect pppoe subscriber VIF with id <pppoe_vif_id>

### Enable PPPoE on VIF

See "vif add" for all details.

Example:

	vif add name v3 port 0 type dot1q cvid 3 flags flow_acct,pppoe_on,npf_on

Using pppoe_on flag enables PPPoE protocol at an interface.

### pppoe ac_cookie key

	pppoe ac_cookie key "key_data"

Sets ac_cookie key value.

Example:

	pppoe ac_cookie key "13071232717"

### pppoe ac_name

	pppoe ac_name "ac_name"

Sets PPPoE AC name.

Example:

	pppoe ac_name "trouter1"

### pppoe service name

	pppoe service name "service_name"

Sets PPPoE service name.

Example:

	pppoe service name "*"

### ppp dns primary

	ppp dns primary <ip_address>

Sets ip address of the primary dns server for ppp subscribers.

Example:

	ppp dns primary 8.8.8.8

### ppp dns secondary

	ppp dns secondary <ip_address>

Sets ip address of the secondary dns server for ppp subscribers.

Example:

	ppp dns primary 8.8.4.4

### ppp ipcp server ip

	ppp ipcp server ip <ip_address>

Sets ip address the_router side ot ppp p-t-p tunnels

Example:

	ppp ipcp server ip 10.10.1.1

## PPPoE subsribers sysctl variables

### pppoe_max_subsc

Maximum number of concurrent pppoe subscribers.

Variable can be used only in the startup configuration file section.

### pppoe_inactive_ttl

Time, seconds. Pppoe subscriber will be disconnected
if there are no packets during this period of time.

### tcp_mss_fix
1 - on, 0 - off. Enables or disables using TCP MSS fix for pppoe traffic.

### ppp_max_terminate
Maximum number of PPP FSM (LCP or NCP(IPCP)) terminate packets that may be sent.

### ppp_max_configure
Maximum number of PPP FSM (LCP or NCP(IPCP)) configure packets that may be sent.

### pppoe_sub_uniq_check
1 - on, 0 - off. Check that each pppoe subscriber has a uniq pair: Host-Uniq TAG and MAC address.
If a new pppoe discover request containging already existing pair of the values is received
it will be dropped. The pppoe_sub_uniq_check variable can be used only in the startup configuration file section.

## RADIUS and CoA

### radius_client add server

	radius_client add server <ip address> [port <port number>]

Add RADIUS server to the list of servers. RADIUS requests
will be sent to servers in the round-robin way. Maximum numbers
of servers in the list is 8. Default port number is 1812.

Example:

	radius_client add server 192.168.5.2 port 1612


### radius_client add src ip 

	radius_client add src ip <ip address>

Add ip address to the list of source ip addresses that will be used
by the TheRouter's RADIUS client to send RADIUS requests. A source
ip address must be assigned to a VIF.

Example:

	radius_client add src ip 192.168.5.111



### radius_client set secret

	radius_client set secret "secret"

Set the RADIUS client secret.

Example:

	radius_client set secret "1234abcd"
	


### coa server set secret

	coa server set secret "secret"

Set the RADIUS CoA server secret.

Example:

	coa server set secret "abcd1234"