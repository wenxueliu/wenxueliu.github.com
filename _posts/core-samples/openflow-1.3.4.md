##Network 

[STP 802.1D](http://en.wikipedia.org/wiki/Spanning_Tree_Protocol)
[Copper cable]
[Fiber cable]
[SCTP]
[MPLS]
[PBB]
[Neighbor Discovery]
[CRC]
link aggregation group (LAG)

##V1.0

ingress port : 数据包进入交换机的端口
virtual ports: OFPP_IN_PORT, OFPP_TABLE, OFPP_NORMAL, OFPP_FLOOD, and OFPP_ALL
physical ports
datapath : 交换机的学名
CRC

###Flow Tables

* Header Fields
* Counter
* Actions

###Switch

* Openflow-enable
* Openflow-only

###Action

####Physicial Port

####Virtual Port
* ALL
* CONTROLLER
* LOCAL
* TABLE
* IN_PORT

* Drop

-------------------

* NORMAL
* FLOOD

* Enqueue

* Modify-Field

####Match


###Message Type

* controller-to-switch : manage or inspect state of switch. may or may not a response from switch
* asynchronous : initiated by switch, update controller of network event and changes to switch state
* symmetric : initiated by switch or controller. send without solicitation

####Controller-to-switch

* Features : STP support  	5.3.1
* Configuration 		5.3.2
* Modify-State  		5.3.3	flow, port
* Read-State			5.3.5   desc,flow,table,port,queue,vendor
* Send-Packet			5.3.6	packet out
* Barrier			5.3.7	dependent meet or received notification for completed operation

PS: flow entry match fields: out_port  priority  ofp_mathch

* Queue configuration Message  exinclude of protocol

####asynchronous

* packet-in : without a match flow entry or send-to-controller action 		5.4.1
* flow-removed : flow modify, flow delete, flow expires				5.4.2
* port-status : 802.1d change port-status or user bring down port-status	5.4.3
* Error

####Symmetric

* Hello
* Echo
* Vendor


##V1.3

问题：
1 the actions 与 actions buckets 的关系？
2 the different between group and group entry
3 Metadata怎么理解
4 the different between action set and action list and actions
5 openflow pipeline and normal pipeline
7 MPLS PBB BFD
6 switch how to decide use normal pipeline or openflow piple when packet into switch
9 Copy TTL outwards and Copy TTL inwards
10 ofp_port_state 由谁来改变
11 the relation between oxm_class, oxm_field, and oxm_length, oxm_hasmask
12 what does bands meaning in opflow meter mod

The OpenFlow pipeline processing defines how packets interact with those flow tables


##Glossary

###group

a list of action buckets and some means of choosing one or more of those buckets to apply on
a per-packet basis.

###Tag

a header that can be inserted or removed from a packet via push and pop actions.

###Outermost Tag

 the tag that appears closest to the beginning of a packet

###Meter

##LAG[lLink Aggregation]


##Port

OpenFlow ports are the network interfaces for passing packets between OpenFlow processing and the rest
of the network. The set of OpenFlow ports may not be identical to the set of network interfaces provided 
by the switch hardware,

##Ports [A2.1 pg.36]

###Standard ports

定义为 Physical port, Logical port 和 Reserved port 中的 LOCAL 端口。有 port counters。

###Physical ports

不一定与物理交换机的端口一一对应

###Logical ports

可以对应与多个 Physical 端口。与 Physical 端口唯一的区别是包含一个 Tunnel-ID。

###Reserved ports

####Required :

	ALL(只出) : 全部的端口，除了包进入的端口和设置了 OFPPC_NO_FWD 的端口
	CONTROLLER (可入可出) : 交换机和控制器连接的端口
	TABLE(只出) : 用于 packet out 消息，代表 openflow pipeline 的开始。有疑问？
	IN_PORT(可出可入) : 包进入交换机的端口，只有在包转发到输入端口的情况下才可以作为输出端口
	ANY(只入): 当没有具体指定端口的时候，比如端口使用了通配符。

####Optional

	LOCAL(可入可出) : 用于其他实体(entities)与 switch 通信。
	NORMAL(只出) : 用于 openflow pipeline 到 non-openflow pipeline 的转换
	FLOOD(只出) : 用于 flood，包到所有 Standard 端口，除了包进入端口和设置了 OFPPS_BLOCKED 的端口

PS: 这里端口的方向，以交换机为参考。


##Flow Tables
###Pipeline process
###Flow Entry
###Matching
###Table-miss
###Flow Removal

Every flow table must support a table-miss flow entry to process table misses. but the table-miss entry need not existe.
The table-miss flow entry may not have the same capability as regular flow entry

match fields 和 priority 唯一标记了一条 flow entry

wildcards all fields(all fields ommitted) and has priority 0 is called table-miss flow entry

##Group Table

###group identifier

###group type

####Required
* all
* indirect

####Optional
* select
* fast failover


###counters

###action buckets

##Meter Table

the meter measures and controls the rate of the aggregate of all flow entries to
which it is attached. Multiple meters can be used in the same table, but in an
exclusive way (disjoint set of flow entries). Multiple meters can be used on the
same set of packets by using them in successive flow tables.

Meters are attached directly to flow entries

##Openflow Channel

###Message Type

* Controller-to-Switch
* Asynchronous
* Symmteric

###Message Handling

1 网络堵塞
2 连接突然断开
3 发送到无效的端口
4 qos 策略: 速率限制，防止ddos攻击

执行顺序：通过 Barrier 消息，否则顺序不能保证。

###Channel Connections

####Connection setup

S:Hello
C:Hello
S:Reply Hello
C:Reply Hello
C:HandShake success or OFPT_ERROR[OFPET_HELLO_FAILED,OFPHFC_COMPATIBLE]

####Connection Interruption

* fail secure mode
* fail standalone mode(Openflow-Hybrid)

####Multiple Controller

* HA 策略

    OFPCR_ROLE_EQUAL
    OFPCR_ROLE_SLAVE
    OFPCR_ROLE_MASTER

####Auxiliary Connection

* Datapath id
* Auxiliary

##controller

##openflow protocol

##switch

* OpenFlow-only : 不支持 NORMAL 和 FLOOD 
* OpenFlow-hybrid : 也许支持 NORMAL 和 FLOOD

    group tables
        group entries: identified by group identifier
            group identifier
            group type
                required: indirect, all
                optional: select, fast failover
            counters
            actions buckets
    pipline
        flow tables
            flow entries
                match fields
                priority
                counters
                instructions : 
		    actions 
		    pipeline processing
                    type :  Write-Actions actions
                            Write-Metadata metadata /mask
                            Goto-Table next-table-id
                            --------------------
                            Meter meter_id
                            Apply-Actions action
                            Clear Actions;
                timeouts
                cookie
                flags

meter table
    meter entry
        meter identifier
        meter brands: identified by rate
            brand type: drop, dscp remark
            rate
            burst
            counters
            type specific arguments
        counters


                         |       |-- queue_1
          O              |  port     ...
          P              |       |-- queue_n
          E     |
          N   / | switch    ....
          F  /  |
          L /            |       |-- queue_1
          O              |  port     ...
          W              |       |-- queue_n
        /       |
control  -----  |  ....
                |
        \ P              |       |-- queue_1
          R              |  port     ...
          O \            |       |-- queue_n
          T  \  |
          O   \ | switch    ....
          C     |
          O              |       |-- queue_1
          L              |  port     ...
                         |       |-- queue_n

flow expiry

action bucket

###action

* Output port_no : packet to port number
* Grout group_id: packet to which group
* Drop : empty action bucket or instruction set, without output action or grout action
* Set Queue queue_id : packet from port to which queue
* Push-Tag/Pop-Tag ethertype : VLAN ID
* Set-Field field type value : modify the respective packet header
* Change-TTL ttl

###list of action

* Apply-Actions instruction
* Packet-out message

The actions of a list of actions are executed in the order specified by the
list, and are applied immediately to the packet.


###action set

flow table <--> action set <--> flow table

An action set is associated with each packet. This set is empty by default.A
flow entry can modify the action set using a Write-Action instruction or a
Clear-Action instruction associated with a particular match. The action set is
carried between flow tables. When the instruction set of a flow entry does not
contain a Goto-Table instruction, pipeline processing stops and the actions in
the action set of the packet are executed.

When an action of a specific type is added in the action set, if an action of
the same type exist, it is overwritten by the later action.


The actions in an action set are applied in the order specified below,
regardless of the order that they were added to the set.

* copy TTL inwards
* pop
* push-MPLS
* push-PBB
* push-VLAN
* copy TTL outwards
* decrement TTL
* set
* qos
* group
* output

idle_timeout
hard_timeout

table match
table miss

output port
ingress port

physical ports
logical ports
reserved ports : ALL CONTROLLER TABLE IN_PORTS ANY LOCAL NORMAL FLOOD

OpenFlow-only
OpenFlow-hybrid

when a port is deleted it is left to the controller to clean up any flow entries
or group entries referencing that port if needed.

pipeline processing can only go forward and not backward

headers can not be transparently removed, added or changed between tables,
unless explicitly specified by OpenFlow processing.

the match fields and priority taken together identify a unique flow entry in a
specific flow table.

The flow entry that wildcards all fields (all fields omitted) and has priority
equal to 0 is called the table-miss flow entry

The table-miss flow entry must support at least sending packets to the
controller using the CONTROLLER reserved port (see 4.5) and dropping packets
using the Clear-Actions instruction

If the table-miss flow entry does not exist, by default packets unmatched by
flow entries are dropped (discarded).

Flow entries are removed from flow tables in two ways, either at the request of
the controller or via the switch flow expiry mechanism.

A non-zero hard_timeout field causes the flow entry to be removed after the
given number of seconds, regardless of how many packets it has matched.

Each meter may have one or more meter bands

Each band specifies the rate at which the band applies and the way packets
should be processed

In practice, the only constraints are that the Meter instruction is executed
before the Apply-Actions instruction, that the Clear-Actions instruction is
executed before the Write-Actions instruction, and that Goto-Table is executed
last.

A switch must reject a flow entry if it is unable to execute the instructions or
part of the instructions associated with the flow entry. In this case, the
switch must return the error message associated with the issue

OpenFlow channel is usually encrypted using TLS, but may be run directly over
TCP.

For out-of-band connections, the switch must make sure that traffic to and from
the OpenFlow channel is not run through the OpenFlow pipeline. For in-band
connections, the switch must set up the proper set of flow entries for the
connection in the OpenFlow pipeline.

###OpenFlow Channel



datapath
Control Channel

message delivery
message processing
message ordering :  barier message

##OpenFlow Protocol

###message type

####controller-to-switch

initiated by the controller and used to directly manage or inspect the state of
the switch

* Features
* Configuration
* Modify-State
* Read-State
* Packet-out
* Barrier
* Role-Request
* Asynchronous-Configuration

####asynchronous

initiated by the switch and used to update the controller of network events and
changes to the switch state.

* Packet-in: TTL checking, table-miss flow entry
* Flow-Removed:
* Port-status
* Error

####symmetric

initiated by either the switch or the controller and sent without solicitation

* Hello
* Echo
* Experimenter

DiffServ

###Multiple controller


####Role

* OFPCR_ROLE_EQUAL
* OFPCR_ROLE_SLAVE
* OFPCR_ROLE_SLAVE.

###without specification

The controllers coordinate the management of the switch amongst themselves via
mechanisms outside the scope of the present specification

####connection

#####catalog

* auxiliary connections
* main connection

what is different between auxiliary connection and main connection?

#####identify

* Datapath ID
* Auxiliary ID


Port NUM TO Queue ID : multi to multi

pipline processing after packet processing

###Flow Match Structures
* Flow Match Header
* Flow Match Field Structures
    oxm_class: ONF member class, ONF reserved class
    oxm_field
    oxm_hasmask
    oxm_length

TLV(type-length-value)
OXM(OpenFlow Extensible Match)


###Flow Instruction Structures

    enum ofp_instruction_type {
        /* Setup the next table in the lookup
        * pipeline
        */
        OFPIT_GOTO_TABLE = 1,

        /* Setup the metadata field for use later in
        * pipeline
        */
        OFPIT_WRITE_METADATA = 2,

        /* Write the action(s) onto the datapath action
        * set
        */
        OFPIT_WRITE_ACTIONS = 3,

        /* Applies the action(s) immediately */
        OFPIT_APPLY_ACTIONS = 4,

        /* Clears all actions from the datapath
        * action set
        */
        OFPIT_CLEAR_ACTIONS = 5,

        /* Apply meter (rate limiter) */
        OFPIT_METER = 6,

        /* Experimenter instruction */
        OFPIT_EXPERIMENTER = 0xFFFF
    }


###Action Structures

Actions are used in flow entry instructions and in group buckets.

    enum ofp_action_type {

        OFPAT_OUTPUT = 0, /* Output to switch port. */

        /* Copy TTL "outwards" --
        * from next-to-outermost to outermost
        */
        OFPAT_COPY_TTL_OUT = 11,

        /* Copy TTL "inwards" -- from outermost to
        * next-to-outermost
        */
        OFPAT_COPY_TTL_IN = 12,

        OFPAT_SET_MPLS_TTL = 15, /* MPLS TTL */
        OFPAT_DEC_MPLS_TTL = 16, /* Decrement MPLS TTL */
        OFPAT_PUSH_VLAN    = 17, /* Push a new VLAN tag */
        OFPAT_POP_VLAN     = 18, /* Pop the outer VLAN tag */
        OFPAT_PUSH_MPLS    = 19, /* Push a new MPLS tag */
        OFPAT_POP_MPLS     = 20, /* Pop the outer MPLS tag */
        OFPAT_SET_QUEUE    = 21, /* Set queue id when outputting to a port */
        OFPAT_GROUP        = 22, /* Apply group. */
        OFPAT_SET_NW_TTL   = 23, /* IP TTL. */
        OFPAT_DEC_NW_TTL   = 24, /* Decrement IP TTL. */
        OFPAT_SET_FIELD    = 25, /* Set a header field using OXM TLV format.*/
        OFPAT_PUSH_PBB     = 26, /* Push a new PBB service tag (I-TAG) */
        OFPAT_POP_PBB      = 27, /* Pop the outer PBB service tag (I-TAG) */
        OFPAT_EXPERIMENTER = 0xffff
    }


##Controller-to-Switch Messages
###Handshake

    Controller    -------->   Switch
OFPT_FEATURES_REQUEST   OFPT_FEATURES_REPLY

####OFPT_FEATURES_REPLY

    /* Switch features. */
    struct ofp_switch_features {
        struct ofp_header header;

        /* Datapath unique ID. The lower 48-bits are for
        *  a MAC address, while the upper 16-bits are
        * implementer-defined.
        */
        uint64_t datapath_id;

        uint32_t n_buffers; /* Max packets buffered at once. */
        uint8_t n_tables; /* Number of tables supported by datapath. */
        uint8_t auxiliary_id; /* Identify auxiliary connections */
        uint8_t pad[2]; /* Align to 64-bits. */

        /* Features. */
        uint32_t capabilities; /* Bitmap of support "ofp_capabilities". */
        uint32_t reserved;

    }


    /* Capabilities supported by the datapath. */
    enum ofp_capabilities {
        OFPC_FLOW_STATS     = 1 << 0, /* Flow statistics. */
        OFPC_TABLE_STATS    = 1 << 1, /* Table statistics. */
        OFPC_PORT_STATS     = 1 << 2, /* Port statistics. */
        OFPC_GROUP_STATS    = 1 << 3, /* Group statistics. */
        OFPC_IP_REASM       = 1 << 5, /* Can reassemble IP fragments. */
        OFPC_QUEUE_STATS    = 1 << 6, /* Queue statistics. */
        OFPC_PORT_BLOCKED   = 1 << 8  /* Switch will block looping ports. */
        };


###Switch Configuration

       Controller    -------->   Switch
    OFPT_SET_CONFIG             NO REPLAY
OFPT_GET_CONFIG_REQUEST     OFPT_GET_CONFIG_REPLY


####OFPT_GET_CONFIG_REPLY

    /* Switch configuration. */
    struct ofp_switch_config {
        struct ofp_header header;
        uint16_t flags;             /* Bitmap of OFPC_* flags. */

        /* Max bytes of packet that datapath
        *  should send to the controller. See
        *  ofp_controller_max_len for valid values.
        */
        uint16_t miss_send_len;
    };
    OFP_ASSERT(sizeof(struct ofp_switch_config) == 12);

####OFPT_SET_CONFIG

    enum ofp_config_flags {
        /* Handling of IP fragments. */
        OFPC_FRAG_NORMAL    = 0,      /* No special handling for fragments. Mandatory */
        OFPC_FRAG_DROP      = 1 << 0, /* Drop fragments. Optional */
        OFPC_FRAG_REASM     = 1 << 1, /* Reassemble (only if OFPC_IP_REASM set). Optional*/
        OFPC_FRAG_MASK      = 3,
    };

###Flow Table Configuration

    /* Table numbering. Tables can use any number up to OFPT_MAX. */
    enum ofp_table {
        /* Last usable table number. */
        OFPTT_MAX   = 0xfe,
        /* Fake tables. */
        OFPTT_ALL   = 0xff
        /* Wildcard table used for table config,
        *  flow stats and flow deletes. 
        */
    };

####OFPT_TABLE_MOD

    /* Configure/Modify behavior of a flow table */
    struct ofp_table_mod {
            struct ofp_header header;
            uint8_t table_id;   /* ID of the table, OFPTT_ALL indicates all tables */
            uint8_t pad[3];     /* Pad to 32 bits */
            uint32_t config;    /* Bitmap of OFPTC_* flags */
    };
    OFP_ASSERT(sizeof(struct ofp_table_mod) == 16);

    /* Flags to configure the table. Reserved for future use. */
    enum ofp_table_config {
            OFPTC_DEPRECATED_MASK   = 3, /* Deprecated bits */
    };

###Modify State Messages

####Modify Flow Entry Message

       Controller    -------->   Switch
    OFPT_FLOW_MOD

    /* Flow setup and teardown (controller -> datapath). */
    struct ofp_flow_mod {
        struct ofp_header header;
        uint64_t cookie;            /* Opaque controller-issued identifier. */

        /* Mask used to restrict the cookie bits
        * that must match when the command is
        * OFPFC_MODIFY* or OFPFC_DELETE*. A value
        * of 0 indicates no restriction.

        Useless:
            OFPFC_ADD
        */
        uint64_t cookie_mask;

        /* Flow actions. */

        /* ID of the table to put the flow in.
        * For OFPFC_DELETE_* commands, OFPTT_ALL
        * can also be used to delete matching
        * flows from all tables.
        */
        uint8_t table_id;

        uint8_t command;            /* One of OFPFC_*. */

        /*
        Useless:
            OFPFC_MODIFY
            OFPFC_MODIFY_STRICT
         */
        uint16_t idle_timeout;      /* Idle time before discarding (seconds). */
        uint16_t hard_timeout;      /* Max time before discarding (seconds). */

        /*
        Useful:
            OFPFC_ADD
            OFPFC_MODIFY_STRICT
            OFPFC_DELETE_STRICT
        Useless:
            OFPFC_DELETE
            OFPFC_MODIFY
         */
        uint16_t priority;          /* Priority level of flow entry. */

        /* Buffered packet to apply to when switch send packet_in message to
        * controller,
        * when no buffered packet is associated with the flow mod OFP_NO_BUFFER
        * is set. Not meaningful for OFPFC_DELETE*.
        */
        uint32_t buffer_id;

        /* For OFPFC_DELETE* commands, require
        * matching entries to include this as an
        * output port. A value of OFPP_ANY
        * indicates no restriction.
        Useless:
            OFPFC_ADD
            OFPFC_MODIFY_STRICT
            OFPFC_MODIFY
        */
        uint32_t out_port;

        /* For OFPFC_DELETE* commands, require
        * matching entries to include this as an
        * output group. A value of OFPG_ANY
        * indicates no restriction.
        Useless:
            OFPFC_ADD
            OFPFC_MODIFY_STRICT
            OFPFC_MODIFY
        */
        uint32_t out_group;

        uint16_t flags;             /* Bitmap of OFPFF_* flags. */
        uint8_t pad[2];
        struct ofp_match match;     /* Fields to match. Variable size. */

        /* Instruction set - 0 or more.
        * The length of the instruction
        * set is inferred from the
        * length field in the header.
        */
        /* The variable size and padded match is always followed by instructions. */
        //struct ofp_instruction instructions[0];
        OFP_ASSERT(sizeof(struct ofp_flow_mod) == 56);
    }

    enum ofp_flow_mod_command{
        OFPFC_ADD           =   0, /* New flow. */
        OFPFC_MODIFY        =   1, /* Modify all matching flows. */
        OFPFC_MODIFY_STRICT =   2, /* Modify entry strictly matching wildcards and priority. */
        OFPFC_DELETE        =   3, /* Delete all matching flows. */
        OFPFC_DELETE_STRICT =   4, /* Delete entry strictly matching wildcards and priority. */
    }


    enum ofp_flow_mod_flags {
        /*
         *  Send flow removed message when flow
         *  expires or is deleted.
        */
        OFPFF_SEND_FLOW_REM = 1 << 0,

        OFPFF_CHECK_OVERLAP = 1 << 1,       /* Check for overlapping entries first. */
        OFPFF_RESET_COUNTS  = 1 << 2,       /* Reset flow packet and byte counts. */
        OFPFF_NO_PKT_COUNTS = 1 << 3,       /* Don’t keep track of packet count. */
        OFPFF_NO_BYT_COUNTS = 1 << 4,       /* Don’t keep track of byte count. */
    };

####Modify Group Entry Message

       Controller    -------->   Switch
    OFPT_GROUP_MOD

    /* Group setup and teardown (controller -> datapath). */
    struct ofp_group_mod {
        struct ofp_header header;
        uint16_t command;               /* One of OFPGC_*. */
        uint8_t type;                   /* One of OFPGT_*. */
        uint8_t pad;                    /* Pad to 64 bits. */
        uint32_t group_id;              /* Group identifier. */

        /* The length of the bucket array is inferred
            * from the length field in the header.
            */
        struct ofp_bucket buckets[0];
    };
    OFP_ASSERT(sizeof(struct ofp_group_mod) == 16);

    /* Group commands */
    enum ofp_group_mod_command {
        OFPGC_ADD    = 0,       /* New group. */
        OFPGC_MODIFY = 1,       /* Modify all matching groups. */
        OFPGC_DELETE = 2,       /* Delete all matching groups. */
    };


    /* Group types. Values in the range [128, 255] are reserved for
     * experimental * use.
    */
    enum ofp_group_type {
        OFPGT_ALL       = 0,    /* All (multicast/broadcast) group. */
        OFPGT_SELECT    = 1,    /* Select group. */
        OFPGT_INDIRECT  = 2,    /* Indirect group. */
        OFPGT_FF        = 3,    /* Fast failover group. */
    };

    /* Group numbering. Groups can use any number up to OFPG_MAX. */
    enum ofp_group {
        /* Last usable group number. */
        OFPG_MAX    = 0xffffff00,

        /* Fake groups. */

        OFPG_ALL    = 0xfffffffc,   /* Represents all groups for group delete commands. */
        OFPG_ANY    = 0xffffffff    /* Special wildcard: no group specified. */
    };

    /* Bucket for use in groups. */
    struct ofp_bucket {
        /* Length of the bucket in bytes, including
        * this header and any padding to make it
        * 64-bit aligned.
        */
        uint16_t len;

        /* Relative weight of bucket. Only
        * defined for select groups.
        */
        uint16_t weight;

        /* Port whose state affects whether this
        * bucket is live. Only required for fast
        * failover groups.
        */
        uint32_t watch_port;

        /* Group whose state affects whether this
        * bucket is live. Only required for fast
        * failover groups.
        */
        uint32_t watch_group;
        uint8_t pad[4];

        /* 0 or more actions associated with
        * the bucket - The action list length
        * is inferred from the length
        * of the bucket.
        */
        struct ofp_action_header actions[0]; 
    };
    OFP_ASSERT(sizeof(struct ofp_bucket) == 16);

DSCP
PCP



