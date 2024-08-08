# Access List, Prefix List, and Time-Based ACL Commands

## Access List Commands

### Overview

Access Control Lists (ACLs) filter network traffic by permitting or denying packets based on criteria such as source and destination IP addresses, protocols, and port numbers. They are essential for controlling access to network resources and implementing security policies.

### Standard ACLs

Standard ACLs filter packets based only on source IP addresses.

#### Configuration Commands

```plaintext
access-list <ACL_NUMBER> permit|deny <SOURCE_IP> <WILDCARD_MASK>
```

- **<ACL_NUMBER>**: An integer from 1 to 99 for standard ACLs.
- **permit|deny**: Specifies whether to allow or block the traffic.
- **<SOURCE_IP>**: The source IP address to match.
- **<WILDCARD_MASK>**: The inverse of the subnet mask used to specify a range.

#### Example

```plaintext
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny 192.168.2.0 0.0.0.255
access-list 10 permit any
```

### Extended ACLs

Extended ACLs filter packets based on source and destination IP addresses, protocols, and port numbers.

#### Configuration Commands

```plaintext
access-list <ACL_NUMBER> permit|deny <PROTOCOL> <SOURCE_IP> <SOURCE_WILDCARD> <DESTINATION_IP> <DESTINATION_WILDCARD> [eq|lt|gt|neq] <PORT>
```

- **<ACL_NUMBER>**: An integer from 100 to 199 for extended ACLs.
- **<PROTOCOL>**: Protocol to match (e.g., ip, tcp, udp, icmp).
- **<SOURCE_IP>** and **<DESTINATION_IP>**: Source and destination IP addresses.
- **<SOURCE_WILDCARD>** and **<DESTINATION_WILDCARD>**: Wildcard masks for source and destination IPs.
- **[eq|lt|gt|neq] <PORT>**: Optional port criteria for TCP/UDP.

#### Example

```plaintext
access-list 101 permit tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 80
access-list 101 deny ip 192.168.3.0 0.0.0.255 any
access-list 101 permit any
```

### Named ACLs

Named ACLs provide more flexibility and readability by allowing you to use names instead of numbers.

#### Configuration Commands

```plaintext
ip access-list standard <ACL_NAME>
 permit|deny <SOURCE_IP> <WILDCARD_MASK>

ip access-list extended <ACL_NAME>
 permit|deny <PROTOCOL> <SOURCE_IP> <SOURCE_WILDCARD> <DESTINATION_IP> <DESTINATION_WILDCARD> [eq|lt|gt|neq] <PORT>
```

#### Example

```plaintext
ip access-list standard MY_ACL
 permit 192.168.1.0 0.0.0.255
 deny 192.168.2.0 0.0.0.255
 permit any

ip access-list extended MY_EXT_ACL
 permit tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 80
 deny ip 192.168.3.0 0.0.0.255 any
 permit any
```

### Deny All

To explicitly deny all other traffic not previously permitted by the ACL, use the `deny` statement at the end of the ACL. This is useful to ensure that only the traffic specified by the permit statements is allowed.

#### Example

```plaintext
access-list 100 permit tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 80
access-list 100 deny ip any any
```

In this example, all traffic not specifically permitted is denied.

## Prefix List Commands

### Overview

Prefix lists filter routes based on IP address prefixes. They are especially useful in routing protocols like BGP and OSPF for more precise route filtering.

### Configuration Commands

#### Basic Prefix List

```plaintext
ip prefix-list <PREFIX_LIST_NAME> permit|deny <PREFIX> [<PREFIX_LENGTH>]
```

- **<PREFIX_LIST_NAME>**: Name of the prefix list.
- **permit|deny**: Specifies whether to allow or block the route.
- **<PREFIX>**: The IP address prefix to match.
- **<PREFIX_LENGTH>**: The length of the prefix in bits (optional).

#### Example

```plaintext
ip prefix-list MY_PREFIX_LIST permit 192.168.1.0/24
ip prefix-list MY_PREFIX_LIST deny 192.168.2.0/24
```

### Advanced Prefix List Examples

#### Matching Specific Prefix Lengths

To match routes with specific prefix lengths:

```plaintext
ip prefix-list MY_SPECIFIC_PREFIX_LIST permit 192.168.0.0/24
ip prefix-list MY_SPECIFIC_PREFIX_LIST permit 192.168.1.0/25
```

#### Using Ge and Le for Aggregation

To allow any prefix length from /24 to /32:

```plaintext
ip prefix-list AGGREGATE_LIST permit 192.168.0.0/24 ge 24 le 32
```

#### Prefix List with Multiple Conditions

To match a specific range and exclude certain subnets:

```plaintext
ip prefix-list MY_PREFIX_LIST permit 10.0.0.0/8 le 24
ip prefix-list MY_PREFIX_LIST deny 10.1.0.0/16
```

#### Prefix List for Route Aggregation

To aggregate multiple subnets into a larger prefix:

```plaintext
ip prefix-list AGGREGATE_LIST permit 192.168.0.0/16
```

### Applying Prefix Lists

Prefix lists are commonly used in routing protocols to filter routes.

#### Example with BGP

To filter incoming BGP routes using a prefix list:

```plaintext
router bgp <AS_NUMBER>
 neighbor <NEIGHBOR_IP> prefix-list MY_PREFIX_LIST in
```

#### Example with OSPF

To filter OSPF external routes using a prefix list:

```plaintext
router ospf <PROCESS_ID>
 distribute-list prefix MY_PREFIX_LIST in
```

### Combining Prefix Lists and ACLs

Prefix lists can be used alongside ACLs for more granular control over routing and traffic filtering.

#### Example

```plaintext
access-list 101 permit ip 192.168.1.0 0.0.0.255 any
ip prefix-list MY_PREFIX_LIST permit 192.168.1.0/24

router bgp <AS_NUMBER>
 neighbor <NEIGHBOR_IP> prefix-list MY_PREFIX_LIST in
```

## Time-Based ACL Commands

### Overview

Time-based ACLs apply access control policies based on the time of day or day of the week, offering more flexible traffic management.

### Configuration Commands

#### Time Range Configuration

```plaintext
time-range <TIME_RANGE_NAME>
 periodic <DAYS> <START_TIME> <END_TIME>
```

- **<TIME_RANGE_NAME>**: The name of the time range.
- **<DAYS>**: Days of the week (e.g., Monday, Tuesday).
- **<START_TIME>** and **<END_TIME>**: Time in HH:MM format.

#### Example

```plaintext
time-range BUSINESS_HOURS
 periodic weekdays 09:00 to 17:00
```

#### Applying Time-Based ACL

```plaintext
ip access-list extended TIME_BASED_ACL
 permit tcp 192.168.1.0 0.0.0.255 any time-range BUSINESS_HOURS
```

### Combining Time-Based ACLs with Other ACLs

You can combine time-based ACLs with other ACL types for more specific control.

#### Example

```plaintext
ip access-list extended TIME_BASED_ACL
 permit tcp 192.168.1.0 0.0.0.255 any time-range BUSINESS_HOURS
 deny ip 192.168.2.0 0.0.0.255 any
```

## Additional Useful Commands and Concepts

### Route Maps

Route maps control route redistribution and policy-based routing by matching and manipulating routing information.

#### Configuration Commands

```plaintext
route-map <MAP_NAME> permit|deny <SEQUENCE_NUMBER>
 match <CRITERIA>
 set <ACTION>
```

- **<MAP_NAME>**: Name of the route map.
- **permit|deny**: Specifies whether to permit or deny the route.
- **<SEQUENCE_NUMBER>**: Sequence number of the route map entry.
- **match <CRITERIA>**: Criteria to match routes (e.g., IP address, AS path).
- **set <ACTION>**: Action to apply to matched routes (e.g., metric, next-hop).

#### Example

```plaintext
route-map FILTER_ROUTES permit 10
 match ip address 101
 set metric 10

route-map FILTER_ROUTES deny 20
 match ip address 102
```

### Policy-Based Routing (PBR)

Policy-Based Routing makes routing decisions based on policies other than the destination IP address.

#### Configuration Commands

```plaintext
route-map <MAP_NAME> permit|deny <SEQUENCE_NUMBER>
 match ip address <ACL_NUMBER>
 set ip next-hop <NEXT_HOP_IP>

interface <INTERFACE_NAME>
 ip policy route-map <MAP_NAME>
```

#### Example



```plaintext
ip access-list extended PBR_ACL
 permit ip 192.168.1.0 0.0.0.255 any

route-map PBR_MAP permit 10
 match ip address PBR_ACL
 set ip next-hop 10.0.0.1

interface GigabitEthernet0/0
 ip policy route-map PBR_MAP
```

