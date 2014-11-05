
controller

openflow protocol

switch
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
                instructions
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

DSCP
PCP
