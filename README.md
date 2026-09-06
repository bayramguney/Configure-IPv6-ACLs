# Configure-IPv6-ACLs

# Packet Tracer - Configure IPv6 ACLs

## Overview

This Cisco Packet Tracer lab demonstrates how to configure, apply, and verify **IPv6 Access Control Lists (ACLs)** on routers.

The lab uses two ACLs:

* `BLOCK_HTTP` — blocks HTTP and HTTPS traffic from reaching Server3 while allowing other IPv6 traffic.
* `BLOCK_ICMP` — blocks ICMP traffic destined for Server3 while allowing other IPv6 traffic.

The purpose is to simulate mitigating **DoS/DDoS attacks** against a server by filtering specific types of network traffic.

---

## Topology

The lab includes:

* **PC1**
* **PC2**
* **Server3**
* **R1**
* **R3**

### IPv6 Addressing

| Device  | Interface | IPv6 Address/Prefix    | Default Gateway |
| ------- | --------- | ---------------------- | --------------- |
| PC1     | NIC       | `2001:db8:1:10::10/64` | `fe80::1`       |
| PC2     | NIC       | `2001:db8:1:11::11/64` | `fe80::1`       |
| Server3 | NIC       | `2001:db8:1:30::30/64` | `fe80::30`      |

---

## Objectives

### Part 1: Configure, Apply, and Verify an IPv6 ACL

Configure an IPv6 ACL on **R1** to block HTTP and HTTPS traffic from reaching Server3.

### Part 2: Configure, Apply, and Verify a Second IPv6 ACL

Configure an IPv6 ACL on **R3** to block ICMP traffic destined for Server3.

---

# Part 1 - BLOCK_HTTP ACL

## Scenario

A computer on the `2001:db8:1:11::/64` network is repeatedly refreshing a web page and causing a potential DoS attack against Server3.

Until the client can be identified and cleaned, HTTP and HTTPS access to Server3 must be blocked from that network.

## Step 1 - Create the ACL

On **R1**, create an IPv6 ACL named `BLOCK_HTTP`.

```cisco
R1(config)# ipv6 access-list BLOCK_HTTP
R1(config-ipv6-acl)# deny tcp any host 2001:db8:1:30::30 eq www
R1(config-ipv6-acl)# deny tcp any host 2001:db8:1:30::30 eq 443
R1(config-ipv6-acl)# permit ipv6 any any
R1(config-ipv6-acl)# exit
```

The ACL:

1. Blocks HTTP traffic (`TCP/80`) to Server3.
2. Blocks HTTPS traffic (`TCP/443`) to Server3.
3. Allows all other IPv6 traffic.

> **Note:** IPv6 ACLs use an implicit deny at the end, so the explicit `permit ipv6 any any` is required to allow all other IPv6 traffic.

## Step 2 - Apply the ACL

Apply `BLOCK_HTTP` to the interface on **R1** that is closest to the source of the traffic being blocked.

```cisco
R1(config)# interface <source-facing-interface>
R1(config-if)# ipv6 traffic-filter BLOCK_HTTP in
R1(config-if)# exit
```

Replace `<source-facing-interface>` with the appropriate R1 interface in the Packet Tracer topology.

## Step 3 - Verify the ACL

### PC1 - HTTP/HTTPS

From **PC1**, open a web browser and access:

```text
http://2001:db8:1:30::30
```

or:

```text
https://2001:db8:1:30::30
```

**Expected result:** The website should appear.

### PC2 - HTTP/HTTPS

From **PC2**, access Server3 using HTTP or HTTPS.

**Expected result:** The website should be blocked.

### PC2 - Ping

From PC2, ping Server3:

```text
ping 2001:db8:1:30::30
```

**Expected result:** The ping should be successful.

This confirms that the ACL blocks only HTTP/HTTPS traffic and allows other IPv6 traffic.

---

# Part 2 - BLOCK_ICMP ACL

## Scenario

The server is now receiving ping requests from many different IPv6 addresses as part of a potential DDoS attack.

ICMP traffic destined for Server3 must therefore be blocked.

## Step 1 - Create the ACL

On **R3**, create an IPv6 ACL named `BLOCK_ICMP`.

```cisco
R3(config)# ipv6 access-list BLOCK_ICMP
R3(config-ipv6-acl)# deny icmp any any
R3(config-ipv6-acl)# permit ipv6 any any
R3(config-ipv6-acl)# exit
```

The ACL:

1. Blocks ICMP traffic.
2. Allows all other IPv6 traffic.

## Step 2 - Apply the ACL

Because ICMP traffic can originate from any source, apply the ACL as close as possible to the destination.

Apply it to the interface on **R3** connected toward Server3:

```cisco
R3(config)# interface <server-facing-interface>
R3(config-if)# ipv6 traffic-filter BLOCK_ICMP out
R3(config-if)# exit
```

Replace `<server-facing-interface>` with the appropriate R3 interface in the Packet Tracer topology.

## Step 3 - Verify the ACL

### PC2 - Ping

From PC2:

```text
ping 2001:db8:1:30::30
```

**Expected result:** The ping should fail.

### PC1 - Ping

From PC1:

```text
ping 2001:db8:1:30::30
```

**Expected result:** The ping should fail.

### PC1 - Web Access

From PC1, open:

```text
http://2001:db8:1:30::30
```

or:

```text
https://2001:db8:1:30::30
```

**Expected result:** The website should still display.

This confirms that `BLOCK_ICMP` blocks ICMP while allowing TCP-based web traffic.

---

# Verification Commands

Use the following commands on the routers to verify the ACL configuration and counters.

### Display IPv6 ACLs

```cisco
show ipv6 access-list
```

### Display the running configuration

```cisco
show running-config
```

### Check IPv6 interfaces

```cisco
show ipv6 interface
```

The ACL hit counters can help confirm that traffic is being matched by the expected ACL entries.

---

# Expected Results

| Test                         | Expected Result |
| ---------------------------- | --------------- |
| PC1 → Server3 HTTP           | Allowed         |
| PC1 → Server3 HTTPS          | Allowed         |
| PC2 → Server3 HTTP           | Blocked         |
| PC2 → Server3 HTTPS          | Blocked         |
| PC2 → Server3 Ping (Part 1)  | Allowed         |
| PC1 → Server3 Ping (Part 2)  | Blocked         |
| PC2 → Server3 Ping (Part 2)  | Blocked         |
| PC1 → Server3 HTTP (Part 2)  | Allowed         |
| PC1 → Server3 HTTPS (Part 2) | Allowed         |

---

## Key Concepts

### IPv6 ACL Syntax

IPv6 ACLs are created using:

```cisco
ipv6 access-list <ACL_NAME>
```

Individual filtering rules are then added inside the ACL.

### Applying an IPv6 ACL

IPv6 ACLs are applied to interfaces using:

```cisco
ipv6 traffic-filter <ACL_NAME> in
```

or:

```cisco
ipv6 traffic-filter <ACL_NAME> out
```

The direction should be selected based on where the unwanted traffic should be filtered.

### Implicit Deny

IPv6 ACLs have an implicit deny at the end. Therefore, an ACL intended to block specific traffic while allowing everything else should normally include:

```cisco
permit ipv6 any any
```

---

## Summary

This lab demonstrates how IPv6 ACLs can be used to mitigate different types of unwanted traffic:

* **`BLOCK_HTTP` on R1** blocks HTTP and HTTPS access to Server3 while allowing other IPv6 traffic.
* **`BLOCK_ICMP` on R3** blocks ICMP traffic close to Server3 while allowing web traffic and other IPv6 traffic.

The lab reinforces the importance of selecting the correct **ACL location and direction** based on the source, destination, and type of traffic being filtered.

---

## Source

Cisco Networking Academy / Cisco Packet Tracer

**Lab:** Packet Tracer - Configure IPv6 ACLs

© 2017–2022 Cisco and/or its affiliates. All rights reserved. Cisco Public.
