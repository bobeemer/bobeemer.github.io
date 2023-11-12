
layout: post
title: 工具手册
category: 手册
tags: 手册
keywords: 手册

# Table of Contents

1.  [Usage: nmap [Scan Type(s)] [Options] {target specification}](#orgc46aea3)
2.  [TARGET SPECIFICATION:](#org5118d8d)
3.  [HOST DISCOVERY:](#orgf6db0fa)
4.  [SCAN TECHNIQUES:](#orgec8c35d)
5.  [PORT SPECIFICATION AND SCAN ORDER:](#orge1454d3)
6.  [SERVICE/VERSION DETECTION:](#org10159e5)
7.  [SCRIPT SCAN:](#org15da4ed)
8.  [OS DETECTION:](#org4c4c414)
9.  [TIMING AND PERFORMANCE:](#org84c8240)
10. [FIREWALL/IDS EVASION AND SPOOFING:](#orgd5ad925)
11. [OUTPUT:](#org78e9136)
12. [MISC:](#org32c2993)
13. [EXAMPLES:](#org803a30a)

\#+Nmap 6.40 ( <http://nmap.org> )


<a id="orgc46aea3"></a>

# Usage: nmap [Scan Type(s)] [Options] {target specification}


<a id="org5118d8d"></a>

# TARGET SPECIFICATION:

Can pass hostnames, IP addresses, networks, etc.
Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0.0-255.1-254
-iL <inputfilename>: Input from list of hosts/networks
-iR <num hosts>: Choose random targets
&#x2013;exclude <host1[,host2][,host3],&#x2026;>: Exclude hosts/networks
&#x2013;excludefile <exclude<sub>file</sub>>: Exclude list from file


<a id="orgf6db0fa"></a>

# HOST DISCOVERY:

-sL: List Scan - simply list targets to scan
-sn: Ping Scan - disable port scan
-Pn: Treat all hosts as online &#x2013; skip host discovery
-PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
-PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
-PO[protocol list]: IP Protocol Ping
-n/-R: Never do DNS resolution/Always resolve [default: sometimes]
&#x2013;dns-servers <serv1[,serv2],&#x2026;>: Specify custom DNS servers
&#x2013;system-dns: Use OS's DNS resolver
&#x2013;traceroute: Trace hop path to each host


<a id="orgec8c35d"></a>

# SCAN TECHNIQUES:

-sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
-sU: UDP Scan
-sN/sF/sX: TCP Null, FIN, and Xmas scans
&#x2013;scanflags <flags>: Customize TCP scan flags
-sI <zombie host[:probeport]>: Idle scan
-sY/sZ: SCTP INIT/COOKIE-ECHO scans
-sO: IP protocol scan
-b <FTP relay host>: FTP bounce scan


<a id="orge1454d3"></a>

# PORT SPECIFICATION AND SCAN ORDER:

-p <port ranges>: Only scan specified ports
  Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9
-F: Fast mode - Scan fewer ports than the default scan
-r: Scan ports consecutively - don't randomize
&#x2013;top-ports <number>: Scan <number> most common ports
&#x2013;port-ratio <ratio>: Scan ports more common than <ratio>


<a id="org10159e5"></a>

# SERVICE/VERSION DETECTION:

-sV: Probe open ports to determine service/version info
&#x2013;version-intensity <level>: Set from 0 (light) to 9 (try all probes)
&#x2013;version-light: Limit to most likely probes (intensity 2)
&#x2013;version-all: Try every single probe (intensity 9)
&#x2013;version-trace: Show detailed version scan activity (for debugging)


<a id="org15da4ed"></a>

# SCRIPT SCAN:

-sC: equivalent to &#x2013;script=default
&#x2013;script=<Lua scripts>: <Lua scripts> is a comma separated list of 
         directories, script-files or script-categories
&#x2013;script-args=<n1=v1,[n2=v2,&#x2026;]>: provide arguments to scripts
&#x2013;script-args-file=filename: provide NSE script args in a file
&#x2013;script-trace: Show all data sent and received
&#x2013;script-updatedb: Update the script database.
&#x2013;script-help=<Lua scripts>: Show help about scripts.
         <Lua scripts> is a comma separted list of script-files or
         script-categories.


<a id="org4c4c414"></a>

# OS DETECTION:

-O: Enable OS detection
&#x2013;osscan-limit: Limit OS detection to promising targets
&#x2013;osscan-guess: Guess OS more aggressively


<a id="org84c8240"></a>

# TIMING AND PERFORMANCE:

Options which take <time> are in seconds, or append 'ms' (milliseconds),
's' (seconds), 'm' (minutes), or 'h' (hours) to the value (e.g. 30m).
-T<0-5>: Set timing template (higher is faster)
&#x2013;min-hostgroup/max-hostgroup <size>: Parallel host scan group sizes
&#x2013;min-parallelism/max-parallelism <numprobes>: Probe parallelization
&#x2013;min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>: Specifies
    probe round trip time.
&#x2013;max-retries <tries>: Caps number of port scan probe retransmissions.
&#x2013;host-timeout <time>: Give up on target after this long
&#x2013;scan-delay/&#x2013;max-scan-delay <time>: Adjust delay between probes
&#x2013;min-rate <number>: Send packets no slower than <number> per second
&#x2013;max-rate <number>: Send packets no faster than <number> per second


<a id="orgd5ad925"></a>

# FIREWALL/IDS EVASION AND SPOOFING:

-f; &#x2013;mtu <val>: fragment packets (optionally w/given MTU)
-D <decoy1,decoy2[,ME],&#x2026;>: Cloak a scan with decoys
-S <IP<sub>Address</sub>>: Spoof source address
-e <iface>: Use specified interface
-g/&#x2013;source-port <portnum>: Use given port number
&#x2013;data-length <num>: Append random data to sent packets
&#x2013;ip-options <options>: Send packets with specified ip options
&#x2013;ttl <val>: Set IP time-to-live field
&#x2013;spoof-mac <mac address/prefix/vendor name>: Spoof your MAC address
&#x2013;badsum: Send packets with a bogus TCP/UDP/SCTP checksum


<a id="org78e9136"></a>

# OUTPUT:

-oN/-oX/-oS/-oG <file>: Output scan in normal, XML, s|<rIpt kIddi3,
   and Grepable format, respectively, to the given filename.
-oA <basename>: Output in the three major formats at once
-v: Increase verbosity level (use -vv or more for greater effect)
-d: Increase debugging level (use -dd or more for greater effect)
&#x2013;reason: Display the reason a port is in a particular state
&#x2013;open: Only show open (or possibly open) ports
&#x2013;packet-trace: Show all packets sent and received
&#x2013;iflist: Print host interfaces and routes (for debugging)
&#x2013;log-errors: Log errors/warnings to the normal-format output file
&#x2013;append-output: Append to rather than clobber specified output files
&#x2013;resume <filename>: Resume an aborted scan
&#x2013;stylesheet <path/URL>: XSL stylesheet to transform XML output to HTML
&#x2013;webxml: Reference stylesheet from Nmap.Org for more portable XML
&#x2013;no-stylesheet: Prevent associating of XSL stylesheet w/XML output


<a id="org32c2993"></a>

# MISC:

-6: Enable IPv6 scanning
-A: Enable OS detection, version detection, script scanning, and traceroute
&#x2013;datadir <dirname>: Specify custom Nmap data file location
&#x2013;send-eth/&#x2013;send-ip: Send using raw ethernet frames or IP packets
&#x2013;privileged: Assume that the user is fully privileged
&#x2013;unprivileged: Assume the user lacks raw socket privileges
-V: Print version number
-h: Print this help summary page.


<a id="org803a30a"></a>

# EXAMPLES:

  nmap -v -A scanme.nmap.org
  nmap -v -sn 192.168.0.0/16 10.0.0.0/8
  nmap -v -iR 10000 -Pn -p 80
SEE THE MAN PAGE (<http://nmap.org/book/man.html>) FOR MORE OPTIONS AND EXAMPLES

