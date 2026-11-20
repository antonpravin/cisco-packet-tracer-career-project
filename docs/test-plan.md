# Test Plan

## Preconditions

- Save the initial topology before testing.
- Record each pass/fail result and packet screenshot.
- Use a separate packet for every test scenario.
- Confirm all device clocks and hostname labels.

## TC-01 DHCP — Employee

1. Connect a PC to an employee access port.
2. Set the PC interface to DHCP.
3. Verify `10.10.10.100–200` is assigned.
4. Verify gateway `10.10.10.254` and DNS `10.10.40.10`.

## TC-02 Inter-VLAN Routing

From an employee PC:

```text
ping 10.10.40.10
ping 10.10.20.254
ping 10.10.99.254
```

Record whether the expected forwarding policy is allowed or denied.

## TC-03 Guest Isolation

From a guest PC:

- DNS lookup: expected pass
- Employee VLAN ping: expected fail
- Management VLAN ping: expected fail
- Server HTTP portal: expected fail unless explicit policy permits it

## TC-04 HSRP Failover

1. Confirm Core1 is Active and Core2 is Standby.
2. Start a continuous ping to `10.10.10.254`.
3. Shut down Core1.
4. Verify the virtual gateway remains reachable.
5. Restore Core1 and verify preemption.

## TC-05 STP and EtherChannel

1. Run `show etherchannel summary`.
2. Confirm Port-channel is up.
3. Run `show spanning-tree summary`.
4. Shut one member link.
5. Verify the remaining member link handles traffic.

## TC-06 Administration

1. SSH to `10.10.99.252` from VLAN 99.
2. Attempt Telnet.
3. Confirm only SSH is permitted.
4. Attempt management access from VLAN 10 and confirm denial.

## TC-07 NAT/PAT

1. Connect a browser to the external web server.
2. Inspect NAT translations on the edge router.
3. Confirm private source addresses are translated to `203.0.113.2`.

## TC-08 Server and DNS

1. Set employee DNS to `10.10.40.10`.
2. Browse `www.corp.local`.
3. Test HTTP and DNS in simulation mode.
4. Record source, destination, and final ACL decision.

## Evidence to Capture

- `show running-config`
- `show ip route`
- `show standby brief`
- `show etherchannel summary`
- `show interfaces trunk`
- `show spanning-tree vlan 10`
- `show ip interface brief`
- `show ip access-lists`
- Ping success before and after failover
