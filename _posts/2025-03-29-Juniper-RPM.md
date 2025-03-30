---
title: Juniper Real-Time Performance Monitoring (RPM)
date: 2025-03-29 21:00:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

## Real-Time Performance Monitoring (RPM)

The **Real-Time Performance Monitoring (RPM)** feature in Junos OS is similar to **IP SLA** in Cisco IOS.
The following configuration makes the device send ICMP echo request packets to a public IP address for monitoring purposes every 3 seconds. The lack of ICMP echo responses received triggers a Junos OS **Event Policy** configuration that deactivates the Default Static Route **0.0.0.0/0**.
The same way, once ICMP echo responses successfully resume from the Internet within a short time frame, the Default Static Route gets activated again.

```bash
set routing-options static route 0.0.0.0/0  next-hop <ISP_IP_address>
set routing-options static route 8.8.8.8/32 next-hop <ISP_IP_address>

set system syslog file DAEMON-INFO-LOG daemon info
set system syslog file CHANGE-LOG change-log any

set services rpm probe PROBE-INTERNET test PROBE-TEST1 probe-type icmp-ping
set services rpm probe PROBE-INTERNET test PROBE-TEST1 target address 8.8.8.8
set services rpm probe PROBE-INTERNET test PROBE-TEST1 test-interval 3
set services rpm probe PROBE-INTERNET test PROBE-TEST1 source-address <ISP_IP_address>
set services rpm probe PROBE-INTERNET test PROBE-TEST1 history-size 512
set services rpm probe PROBE-INTERNET test PROBE-TEST1 thresholds successive-loss 5

set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE events ping_test_failed
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE events ping_probe_failed
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE within 30 trigger until
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE within 30 trigger 4
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE within 25 trigger on
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE within 25 trigger 3
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE attributes-match ping_test_failed.test-owner matches PROBE-INTERNET
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE attributes-match ping_test_failed.test-name matches PROBE-TEST1
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE then change-configuration commands "deactivate routing-options static route 0.0.0.0/0"
set event-options policy DISABLE-ON-PROBE-INTERNET-FAILURE then change-configuration commit-options log "updating configuration from event policy DISABLE-ON-PROBE-INTERNET-FAILURE"
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS events ping_test_completed

set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS within 20 trigger on
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS within 20 trigger 3
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS within 25 trigger until
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS within 25 trigger 4
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS attributes-match ping_test_completed.test-owner matches PROBE-INTERNET
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS attributes-match ping_test_completed.test-name matches PROBE-TEST1
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS then change-configuration commands "activate routing-options static route 0.0.0.0/0"
set event-options policy ENABLE-ON-PROBE-INTERNET-SUCCESS then change-configuration commit-options log "updating configuration from event policy ENABLE-ON-PROBE-INTERNET-SUCCESS"
```

## Failover testing

Failover can be tested by removing and re-adding the /32 Static Route.

When executing the "simulate WAN outage" configuration lines, we should see the Default Static Route being removed from the Routing Table after a few seconds.
Likewise, once the configuration lines "recover from WAN outage" are run, the Default Static Route should be restored.

```bash
# simulate WAN outage
delete routing-options static route 8.8.8.8/32 next-hop <ISP_IP_address>
set    routing-options static route 8.8.8.8/32 discard
commit
```

```bash
# recover from WAN outage
delete routing-options static route 8.8.8.8/32 discard
set    routing-options static route 8.8.8.8/32 next-hop <ISP_IP_address>
commit
```

## Troubleshooting commands

Display the Default Static Route is on the Routing Table if available

```bash
show route 0.0.0.0/0 exact
```

Monitor for the success or failure of a receiving ICMP echo reply. Up to 512 attempts can be stored in the history which is the max value allowed in the configuration

```bash
show services rpm history-results owner PROBE-INTERNET
```

Stored in a log file, a long history of successful and failed attempts to get ICMP echo replies. **IMPORTANT** log lines are abbreviated in the logs

```bash
show log DAEMON-INFO-LOG | match PING_TEST
```

Stored in a log file, this display the root account deactivating or activating the Default Static route when the event policy is triggered

```bash
show log CHANGE-LOG | no-more | match root
```

Similar to the previous one, we should see the message "updating configuration from event policy DISABLE-ON-PROBE-INTERNET-FAILURE" or "updating configuration from event policy ENABLE-ON-PROBE-INTERNET-SUCCESS" based on the configuration

```bash
show log messages | match "while executing policy"
```

Shows the configuration of RPM and the Event Policy in Junos OS.

```bash
show configuration | display set | match PROBE-INTERNET
```

## References

- [[SRX] Example - RPM with event-options for route failover](https://supportportal.juniper.net/s/article/SRX-Example-RPM-with-event-options-for-route-failover?language=en_US)

- [Automation Scripting User Guide - Example: Changing the Interface Configuration in Response to an Event](https://www.juniper.net/documentation/us/en/software/junos/automation-scripting/topics/example/junos-script-automation-event-policy-change-configuration.html)
