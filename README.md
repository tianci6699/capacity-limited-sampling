# capacity-limited-sampling

There is an implementation of OVS packet sampling mechanism with limited flow sampling capacity. 
To use it, you should firstly install our software switch, which is based on OpenFlow 1.3 compatible user-space software switch, https://github.com/CPqD/ofsoftswitch13.

These instructions have been tested on Ubuntu 22.04. Other distributions or versions may need different steps.

1 Installation instructions, the switch makes use of the NetBee library to parse packets, so we need to install it first.

1.1 Install the following packages:
$ sudo apt-get install cmake libpcap-dev libxerces-c-dev libpcre3 libpcre3-dev flex bison pkg-config autoconf libtool libboost-dev

1.2 Clone and build netbee
$ git clone https://github.com/netgroup-polito/netbee.git
$ cd netbee/src
$ cmake .
$ make

1.3 Add the shared libraries built in /nbeesrc/bin/ to your /usr/local/lib directory and put the contens of folder nbeesrc/include in the /usr/include
$ sudo cp ../bin/libn*.so /usr/local/lib
$ sudo ldconfig
$ sudo cp -R ../include/* /usr/include/

2 Clone and build ofsoftswitch13
$ git clone https://github.com/netgroup-polito/netbee.git
$ ./boot.sh
$ ./configure
$ make
$ sudo make install

3 Running instructions, you can send requests to the switch using the dpctl utility. We configure the flow sampling rule from the console for the sampling switch through dpctl. This sampling rule specifies the sampling operation on which switch for which flows at what sampling rate and sampling capacity.

3.1 Ethernet header. The fields to match in the Ethernet header are the source and destination MAC and the Ethernet type. The Ethernet type is should be a very common field, as it is a pre-requisite for upper layer fields. In the first example, we will install a sampling rule for a simple flow from 00:00:00:00:00:01 to 00:00:00:00:00:02 without Ethernet type.

dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_src=00:00:00:00:00:01,eth_dst=00:00:00:00:00:02
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_dst="00:00:00:00:00:02", eth_src="00:00:00:00:00:01"}}

OK.

Flow-rate-mod is the most variable command of dpctl, which can configure the flow sampling rule. 
table field represents the flow table number, with the default value of 0. rate field indicates the sampling rate, and bound field represents the upper limit of the sampling capacity for one flow or multiple flows, that is, the maximum number of packets sampled per second. The matching condition eth_src=00:00:00:00:00:01,eth_dst=00:00:00:00:00:02 can specify one or more traffic flows. You can also configure a sampling rule that will sample only IPv4 packets between two hosts, adding the ethernet type 0x800 to the match fields of our previous example.

dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_type=0x0800,eth_src=00:00:00:00:00:01,eth_dst=00:00:00:00:00:02
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_dst="00:00:00:00:00:02", eth_src="00:00:00:00:00:01", eth_type="0x800"}}

OK.

In the next example, the sampling rule only specifies the Ethernet destination address of the flow.
dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_dst=00:00:00:00:00:02
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_dst="00:00:00:00:00:02"}}

OK.
dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_dst=00:00:00:00:00:01
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_dst="00:00:00:00:00:01"}}

OK.

3.2 IPv4 header. There are five fields to match in the IPv4 header. Beyond the source and destination address it's possible to match the ip proto and the TCP source port and destiny port fields. The pre-requisite to match is the ethernet type equals to 0x800. Our example shows a flow with IP source and destination addresses.

dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10  eth_type=0x0800,ip_src=192.168.0.1,ip_dst=172.40.56.101
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_type="0x800", ipv4_src="192.168.0.1", ipv4_dst="172.40.56.101"}}

OK.

3.3 TCP and UDP. OpenFlow supports TCP and UDP source and destination ports. Due to the similarity of flows containing these fields, we are covering both in just one topic. The TCP source port and destiny port fields must add two preconditions simultaneously, the ethernet type 0x800 and the ip proto field with a value of 6.

dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_type=0x0800,ip_proto=6,tcp_src=6666,tcp_dst=9999
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_type="0x800", ip_proto="6", tcp_src="6666", tcp_dst="9999"}}

OK.

Similarly to TCP, to install a rule with UDP source and destination, you should have the field ip_proto equals to 17.
dpctl unix:/tmp/s1 flow-rate-mod table=0,rate=5,bound=10 eth_type=0x0800,ip_proto=17,udp_src=6666,udp_dst=9999
SENDING (xid=0xF0FF00F0):
flow_rate_mod{table="0", cookie="0x0", mask="0x0"", rate="5", bound="10", match=oxm{eth_type="0x800", ip_proto="17", udp_src="6666", udp_dst="9999"}}

OK.
