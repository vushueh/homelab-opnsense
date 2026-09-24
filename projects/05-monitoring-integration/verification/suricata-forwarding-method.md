# Q028 Suricata regression and flow telemetry

The existing Q027 EVE syslog route and custom Wazuh decoder/rule were retained. The Q028 resume sent two inert matching ICMP echoes and two nonmatching echoes from a disposable client confined to the lab source VLAN. Both pairs received two replies. The manager recorded eight SID 2062640 observations for the positive flow and none in the negative window. The eight observations cover two interfaces and both echo directions; they are not eight attacks. Indexed confirmation remains a separate acceptance gate.

The three approved TCP connects used ports 22, 80 and 21, closing without authentication or application commands. Every connection succeeded. All three exact tuples appeared in a fresh dedicated OPNsense collector file, with interface IDs, packet and byte counts: ten exported records, 24 packets and 1,192 bytes in that filtered capture. Multiple capture observations and flow splits prevent interpreting this as unique wire volume.

The temporary client's first DHCP attempt timed out and was fully cleaned up. The successful test used an address outside the verified DHCP pool after duplicate-address probing, then confirmed the gateway's ARP response. Every run verified original host routes, rules, bridge membership/VLANs, MAC/MTU, DNS and hostname after cleanup. No VM, persistent network configuration or firewall policy was modified.

Private exact tuples and command output belong to the management evidence record. NetFlow is queried on its dedicated collector; it was not integrated into the Wazuh dashboard.
