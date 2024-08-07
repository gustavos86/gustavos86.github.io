---
title: How to navigate Logical Systems in Junos OS CLI
date: 2024-07-13 11:30:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper, junos os]     # TAG names should always be lowercase
---

## 1. Appending the logical system to a command

This will display the information about that specific logical system. The Logical System needs to be explicitly stated in the command. For example, the OSPF neighborship for a specific Logical System.

```
lab@mxD> show ospf neighbor logical-system R3-1 
Address          Interface              State     ID               Pri  Dead
172.22.125.2     ge-0/0/0.0             Full      20.20.1.1        128    33

lab@mxD>
```

## 2. Use set cli logical-system to move across Logical Systems

The advangate of this approach is that all the commands run will display information about a specific Logical System implicitly.

```
lab@mxD> set cli logical-system ?
Possible completions:
  R3-1                 
  R3-2                 
  tenant               Name of tenant
lab@mxD> set cli logical-system
```

```
lab@mxD> set cli logical-system R3-1 
Logical system: R3-1

lab@mxD:R3-1> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.125.2     ge-0/0/0.0             Full      20.20.1.1        128    36

lab@mxD:R3-1> 
```

Use the same command to "move" to a different Logical System.

```
lab@mxD:R3-1> set cli logical-system R3-2 
Logical system: R3-2

lab@mxD:R3-2> 
```

## 3. Exit all Logical Systems

To run all commands out of any Logical System again.

```
lab@mxD:R3-2> clear cli logical-system 
Cleared default logical system

lab@mxD>
```