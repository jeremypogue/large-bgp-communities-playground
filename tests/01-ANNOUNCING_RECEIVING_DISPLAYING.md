# Announcing, receiving and displaying large communities

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538

They all announce the following prefixes:

- 203.0.113.x1, 1 large community (ASN:1:1)
- 203.0.113.x2, 2 large communities (ASN:1:1 ASN:1:2)
- 203.0.113.x3, 3 large communities (ASN:1:1 ASN:1:2 ASN:1:3)
- 203.0.113.x4, 1 duplicate large community (ASN:1:1 ASN:1:1)
- 203.0.113.x5, large communities with zeroes (ASN:0:1 ASN:1:0)
- 203.0.113.x6, large communities with reserved values (65535:1:1 4294967295:4294967295:4294967295)

x = 1 for ExaBGP, 2 for GoBGP, 3 for BIRD.

ExaBGP and BIRD read the prefixes to announce from their [configuration files](01/); GoBGP announces them via CLI:

```
gobgp global rib add -a ipv4 203.0.113.21/32 large-community 65537:1:1
gobgp global rib add -a ipv4 203.0.113.22/32 large-community 65537:1:1,65537:1:2
gobgp global rib add -a ipv4 203.0.113.23/32 large-community 65537:1:1,65537:1:2,65537:1:3
gobgp global rib add -a ipv4 203.0.113.24/32 large-community 65537:1:1,65537:1:1
gobgp global rib add -a ipv4 203.0.113.25/32 large-community 65537:0:1,65537:1:0
gobgp global rib add -a ipv4 203.0.113.26/32 large-community 65535:1:1,4294967295:4294967295:4294967295
```

## Results

### ExaBGP announcing

GoBGP:

```
root@gpbgp:/go# gobgp neighbor 192.0.2.2 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.11/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65536:1:1]}]
    203.0.113.12/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65536:1:1, 65536:1:2]}]
    203.0.113.13/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65536:1:1, 65536:1:2, 65536:1:3]}]
    203.0.113.14/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65536:1:1, 65536:1:1]}]
    203.0.113.15/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65536:0:1, 65536:1:0]}]
    203.0.113.16/32     192.0.2.2            65536                00:00:13   [{Origin: i} {LargeCommunity: [ 65535:1:1, 4294967295:4294967295:4294967295]}]
```

:white_check_mark: ExaBGP, send correctly formatted large communities

:white_check_mark: ExaBGP, send 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: ExaBGP, send reserved values (203.0.113.16/32)

:white_check_mark: ExaBGP, duplicate large communities not transmitted (203.0.113.14/32)

:white_check_mark: GoBGP, receive and display large communities with routes

:white_check_mark: GoBGP, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: GoBGP, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: GoBGP, receive duplicate communities (203.0.113.14/32)

:white_check_mark: GoBGP, display reserved values (203.0.113.16/32)

BIRD:

```
203.0.113.14/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1)
203.0.113.15/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 0, 1) (65536, 1, 0)
203.0.113.12/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1) (65536, 1, 2)
203.0.113.13/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1) (65536, 1, 2) (65536, 1, 3)
203.0.113.11/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1)
203.0.113.16/32    unreachable [ExaBGP 16:26:27 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65535, 1, 1) (4294967295, 4294967295, 4294967295)
```

:white_check_mark: BIRD, receive and display large communities with routes

:white_check_mark: BIRD, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: BIRD, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: BIRD, receive duplicate communities (203.0.113.14/32)

:white_check_mark: BIRD, display reserved values (203.0.113.16/32)

### GoBGP announcing

ExaBGP JSON dump:

```
{ "exabgp": "3.5.0", "time": 1475607517.29, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65537, 1 , 2 ], [ 65537, 1 , 3 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.23/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475607517.3, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 2, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.24/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475607517.3, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 3, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 0 , 1 ], [ 65537, 1 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.25/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475607517.3, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 4, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65535, 1 , 1 ], [ 4294967295, 4294967295 , 4294967295 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.26/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475607517.3, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 5, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65537, 1 , 2 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.22/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475607517.3, "host" : "exabgp", "pid" : 264, "ppid" : 1, "counter": 6, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.21/32" } ] } } } } } }
```

:white_check_mark: GoBGP, send correctly formatted large communities

:white_check_mark: GoBGP, send 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: GoBGP, send reserved values (203.0.113.26/32)

:x: GoBGP, duplicate large communities not transmitted (203.0.113.24/32)

:white_check_mark: ExaBGP, receive and display large communities with routes

:white_check_mark: ExaBGP, receive 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: ExaBGP, textual representation, integers not omitted, even when zero (203.0.113.25/32)

:white_check_mark: ExaBGP, receive duplicate communities (203.0.113.24/32)

:white_check_mark: ExaBGP, display reserved values (203.0.113.26/32)

BIRD:

```
bird> show route all protocol GoBGP
203.0.113.26/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65535, 1, 1) (4294967295, 4294967295, 4294967295)
203.0.113.24/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 1)
203.0.113.25/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 0, 1) (65537, 1, 0)
203.0.113.22/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 2)
203.0.113.23/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 2) (65537, 1, 3)
203.0.113.21/32    unreachable [GoBGP 18:14:17 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1)
```

:white_check_mark: BIRD, receive and display large communities with routes

:white_check_mark: BIRD, receive 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: BIRD, textual representation, integers not omitted, even when zero (203.0.113.25/32)

:white_check_mark: BIRD, receive duplicate communities (203.0.113.24/32)

:white_check_mark: BIRD, display reserved values (203.0.113.26/32)

### BIRD announcing

GoBGP:

```
root@gobgp:/go# gobgp neighbor 192.0.2.4 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.31/32     192.0.2.4            65538                00:03:03   [{Origin: i} {LargeCommunity: [ 65538:1:1]}]
    203.0.113.32/32     192.0.2.4            65538                00:03:03   [{Origin: i} {LargeCommunity: [ 65538:1:1, 65538:1:2]}]
    203.0.113.33/32     192.0.2.4            65538                00:00:31   [{Origin: i} {LargeCommunity: [ 65538:1:1, 65538:1:2, 65538:1:3]}]
    203.0.113.34/32     192.0.2.4            65538                00:00:31   [{Origin: i} {LargeCommunity: [ 65538:1:1]}]
    203.0.113.35/32     192.0.2.4            65538                00:00:31   [{Origin: i} {LargeCommunity: [ 65538:0:1, 65538:1:0]}]
    203.0.113.36/32     192.0.2.4            65538                00:00:31   [{Origin: i} {LargeCommunity: [ 65535:1:1, 4294967295:4294967295:4294967295]}]
```

:white_check_mark: BIRD, send correctly formatted large communities

:white_check_mark: BIRD, send 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: BIRD, send reserved values (203.0.113.36/32)

:white_check_mark: BIRD, duplicate large communities not transmitted (203.0.113.34/32)

:white_check_mark: GoBGP, receive and display large communities with routes

:white_check_mark: GoBGP, receive 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: GoBGP, textual representation, integers not omitted, even when zero (203.0.113.35/32)

:white_check_mark: GoBGP, display reserved values (203.0.113.36/32)

ExaBGP:

```
{ "exabgp": "3.5.0", "time": 1475609673.22, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65535, 1 , 1 ], [ 4294967295, 4294967295 , 4294967295 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.36/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475609673.23, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 2, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.34/32" }, { "nlri": "203.0.113.31/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475609673.24, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 3, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 0 , 1 ], [ 65538, 1 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.35/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475609673.24, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 4, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ], [ 65538, 1 , 2 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.32/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475609673.24, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 5, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ], [ 65538, 1 , 2 ], [ 65538, 1 , 3 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.33/32" } ] } } } } } }
```

:white_check_mark: ExaBGP, receive and display large communities with routes

:white_check_mark: ExaBGP, receive 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: ExaBGP, textual representation, integers not omitted, even when zero (203.0.113.35/32)

:white_check_mark: ExaBGP, display reserved values (203.0.113.26/32)

Nota bene: two NLRIs in the same JSON dump for 203.0.113.31/32 (single 65538:1:1 large community) and 203.0.113.34/32 (configured with a double 65538:1:1 large community but sent deduplicated by BIRD).