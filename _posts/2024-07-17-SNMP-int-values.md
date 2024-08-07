---
title: SNMP integer values for some OIDs
date: 2024-07-17 12:30:00 -0700
categories: [SNMP]
tags: [snmp]     # TAG names should always be lowercase
---

SNMP reports the values of OIDs as integers and it is up to the application show the meaning of those values to the users.
If the application does not include the meaning of these integer, it becomes to the user cumbersome to look up for the meaning of these on other sources of information.

Some examples I have come across:

```
ospfNbrState OBJECT-TYPE
    SYNTAX INTEGER {
        down (1),
        attempt (2),
        init (3),
        twoWay (4),
        exchangeStart (5),
        exchange (6),
        loading (7),
        full (8)
    }
```

src: [https://oidref.com/1.3.6.1.2.1.14.10.1.6](https://oidref.com/1.3.6.1.2.1.14.10.1.6)

```
bgpPeerState OBJECT-TYPE
	SYNTAX INTEGER {
		idle(1),
		connect(2),
		active(3),
		opensent(4),
		openconfirm(5),
		established(6)
	}
```

src: [https://oidref.com/1.3.6.1.2.1.15.3.1.2](https://oidref.com/1.3.6.1.2.1.15.3.1.2)

```
ifOperStatus OBJECT-TYPE
    SYNTAX INTEGER {
        up(1),
        down(2),
        testing(3),
        unknown(4),
        dormant(5),
        notPresent(6),
        lowerLayerDown(7)
    }
```

src: [https://oidref.com/1.3.6.1.2.1.2.2.1.8](https://oidref.com/1.3.6.1.2.1.2.2.1.8)
