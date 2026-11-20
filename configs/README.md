# Configuration Exports

Place complete text exports from each Packet Tracer device in this directory:

```text
configs/
├── CORE-1.txt
├── CORE-2.txt
├── ACCESS-1.txt
├── ACCESS-2.txt
└── EDGE-R1.txt
```

Export by opening each CLI and copying the output of:

```text
enable
show running-config
show startup-config
show ip interface brief
show etherchannel summary
show standby brief
show spanning-tree summary
show ip route
show ip access-lists
```

Do not store passwords or secrets in a public repository. Replace credentials with placeholders before committing exports.
