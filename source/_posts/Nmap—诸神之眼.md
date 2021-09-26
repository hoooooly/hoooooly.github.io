---
title: Nmap—诸神之眼
tags:
  - 信息探测
  - nmap
comments: true
typora-root-url: Nmap-诸神之眼
date: 2021-09-26 19:48:57
categories: 安全
index_img: 2021/09/26/Nmap—诸神之眼/sitelogo.png
banner_img:
---

# 概述
Nmap概述网络映射器（Network Mapper，Nmap）是一个免费开源的网络扫描和嗅探工具，可以用来扫描计算机上开放的端口，确定哪些服务运行在哪些端口，并且推断出计算机运行的操作系统。利用该工具，可以评估网络系统的安全性

## 功能架构
Nmap主要有4项扫描功能，分别是**主机发现**、**端口扫描**、**版本侦测**和**操作系统侦测**。

![](2021-09-26_110727.png)

Nmap各功能之间的依赖关系。

（1）确定目标，进行主机发现，找出活动的主机，然后确定活动主机上的端口状况。
（2）根据端口进行扫描。
（3）确定端口上具体运行的应用程序与版本信息。
（4）对版本信息侦测后，再对操作系统进行侦测。

在这4项基本功能的基础上，Nmap还提供了防火墙与入侵检测系统（IntrusionDetection System，IDS）的规避技巧，可以综合应用到4个基本功能的各个阶段。另外，Nmap还提供了强大的NSE（Nmap Scripting Language）脚本引擎功能，可以对基本功能进行补充和扩展，提供漏洞扫描等功能。

## 工作原理
map通过发包和分析响应包来判断目标主机的状态。为了获取有价值的信息，Nmap会发送特定的包（Probe），并分析响应包的特征信息（指纹）。

### 1. 探针

探针（Probe）是基于协议功能和特性，使用特定端口和数据载荷所构建的数据包。

### 2. 指纹信息

指纹信息就是目标主机响应包的特征信息，如ARP应答报文、TCP标志位、ICMP应答报文等。Nmap根据这些指纹信息即可判断主机的状态和端口状态等。

## Nmap的扫描类型

Ping扫描：用于发现主机，以探测网络中活动的主机。
TCP SYN扫描：Nmap默认的端口扫描方式，用来探测目标开放的TCP端口。
操作系统识别：用于识别操作系统的指纹信息，如操作系统类型和服务版本等。
端口扫描：用于探测目标主机中开放的端口。
UDP扫描：使用UDP实施扫描，如UDP Ping扫描和UDP端口扫描等。
隐蔽扫描：主要用于规避防火墙，如TCP NULL扫描、TCP FIN扫描、TCP Xmas扫描和FTP转发扫描等。

## 安装

网上有很多关于Nmap安装的教程，不做介绍。

## 网络环境
### 查看网路接口
多主机都具备多个网络接口。为了明确扫描时所使用的接口，Nmap工具提供了选项--iflist，用来查看主机的网络接口和路由信息。

```shell
nmap --iflist
```

### 网络配置

当确定了当前主机中的网络接口后，即可使用网络调试命令获取网络配置信息。在Windows系统中可以使用ipconfig命令查看网络配置；在Linux系统中可以使用ifconfig命令查看网络配置。

### IPv4和IPv6网络

IP地址可以分为IPv4和IPv6两大类，网络也可以分为IPv4网络和IPv6网络。其中，IPv4是Internet Protocol Version 4的缩写，表示IP的第四个版本，IPv6表示IP的第六个版本，是下一代互联网协议。目前，大部分用户使用的IP地址都是IPv4。Nmap工具支持对IPv4和IPv6网络进行扫描，默认扫描的是IPv4网络。如果用户要扫描IPv6网络，则需要使用-6选项启用IPv6扫描功能。

### 查看帮助

````shell
C:\>nmap -h
Nmap 7.92 ( https://nmap.org )
Usage: nmap [Scan Type(s)] [Options] {target specification}
TARGET SPECIFICATION:	# 目标规范
  Can pass hostnames, IP addresses, networks, etc.
  Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0.0-255.1-254
  -iL <inputfilename>: Input from list of hosts/networks
  -iR <num hosts>: Choose random targets
  --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
  --excludefile <exclude_file>: Exclude list from file
HOST DISCOVERY:	# 主机发现
  -sL: List Scan - simply list targets to scan
  -sn: Ping Scan - disable port scan
  -Pn: Treat all hosts as online -- skip host discovery
  -PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
  -PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
  -PO[protocol list]: IP Protocol Ping
  -n/-R: Never do DNS resolution/Always resolve [default: sometimes]
  --dns-servers <serv1[,serv2],...>: Specify custom DNS servers
  --system-dns: Use OS's DNS resolver
  --traceroute: Trace hop path to each host
SCAN TECHNIQUES:	# 扫描方式
  -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
  -sU: UDP Scan
  -sN/sF/sX: TCP Null, FIN, and Xmas scans
  --scanflags <flags>: Customize TCP scan flags
  -sI <zombie host[:probeport]>: Idle scan
  -sY/sZ: SCTP INIT/COOKIE-ECHO scans
  -sO: IP protocol scan
  -b <FTP relay host>: FTP bounce scan
PORT SPECIFICATION AND SCAN ORDER:	# 端口范围和扫描规则
  -p <port ranges>: Only scan specified ports
    Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9
  --exclude-ports <port ranges>: Exclude the specified ports from scanning
  -F: Fast mode - Scan fewer ports than the default scan
  -r: Scan ports consecutively - don't randomize
  --top-ports <number>: Scan <number> most common ports
  --port-ratio <ratio>: Scan ports more common than <ratio>
SERVICE/VERSION DETECTION:	# 服务/版本探测
  -sV: Probe open ports to determine service/version info
  --version-intensity <level>: Set from 0 (light) to 9 (try all probes)
  --version-light: Limit to most likely probes (intensity 2)
  --version-all: Try every single probe (intensity 9)
  --version-trace: Show detailed version scan activity (for debugging)
SCRIPT SCAN:	# 脚本扫描
  -sC: equivalent to --script=default
  --script=<Lua scripts>: <Lua scripts> is a comma separated list of
           directories, script-files or script-categories
  --script-args=<n1=v1,[n2=v2,...]>: provide arguments to scripts
  --script-args-file=filename: provide NSE script args in a file
  --script-trace: Show all data sent and received
  --script-updatedb: Update the script database.
  --script-help=<Lua scripts>: Show help about scripts.
           <Lua scripts> is a comma-separated list of script-files or
           script-categories.
OS DETECTION:	# 系统探测
  -O: Enable OS detection
  --osscan-limit: Limit OS detection to promising targets
  --osscan-guess: Guess OS more aggressively
TIMING AND PERFORMANCE:
  Options which take <time> are in seconds, or append 'ms' (milliseconds),
  's' (seconds), 'm' (minutes), or 'h' (hours) to the value (e.g. 30m).
  -T<0-5>: Set timing template (higher is faster)
  --min-hostgroup/max-hostgroup <size>: Parallel host scan group sizes
  --min-parallelism/max-parallelism <numprobes>: Probe parallelization
  --min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>: Specifies
      probe round trip time.
  --max-retries <tries>: Caps number of port scan probe retransmissions.
  --host-timeout <time>: Give up on target after this long
  --scan-delay/--max-scan-delay <time>: Adjust delay between probes
  --min-rate <number>: Send packets no slower than <number> per second
  --max-rate <number>: Send packets no faster than <number> per second
FIREWALL/IDS EVASION AND SPOOFING:	# 防火墙/ ids规避和欺骗
  -f; --mtu <val>: fragment packets (optionally w/given MTU)
  -D <decoy1,decoy2[,ME],...>: Cloak a scan with decoys
  -S <IP_Address>: Spoof source address
  -e <iface>: Use specified interface
  -g/--source-port <portnum>: Use given port number
  --proxies <url1,[url2],...>: Relay connections through HTTP/SOCKS4 proxies
  --data <hex string>: Append a custom payload to sent packets
  --data-string <string>: Append a custom ASCII string to sent packets
  --data-length <num>: Append random data to sent packets
  --ip-options <options>: Send packets with specified ip options
  --ttl <val>: Set IP time-to-live field
  --spoof-mac <mac address/prefix/vendor name>: Spoof your MAC address
  --badsum: Send packets with a bogus TCP/UDP/SCTP checksum
OUTPUT:	# 输出方式
  -oN/-oX/-oS/-oG <file>: Output scan in normal, XML, s|<rIpt kIddi3,
     and Grepable format, respectively, to the given filename.
  -oA <basename>: Output in the three major formats at once
  -v: Increase verbosity level (use -vv or more for greater effect)
  -d: Increase debugging level (use -dd or more for greater effect)
  --reason: Display the reason a port is in a particular state
  --open: Only show open (or possibly open) ports
  --packet-trace: Show all packets sent and received
  --iflist: Print host interfaces and routes (for debugging)
  --append-output: Append to rather than clobber specified output files
  --resume <filename>: Resume an aborted scan
  --noninteractive: Disable runtime interactions via keyboard
  --stylesheet <path/URL>: XSL stylesheet to transform XML output to HTML
  --webxml: Reference stylesheet from Nmap.Org for more portable XML
  --no-stylesheet: Prevent associating of XSL stylesheet w/XML output
MISC:
  -6: Enable IPv6 scanning
  -A: Enable OS detection, version detection, script scanning, and traceroute
  --datadir <dirname>: Specify custom Nmap data file location
  --send-eth/--send-ip: Send using raw ethernet frames or IP packets
  --privileged: Assume that the user is fully privileged
  --unprivileged: Assume the user lacks raw socket privileges
  -V: Print version number
  -h: Print this help summary page.
EXAMPLES:
  nmap -v -A scanme.nmap.org
  nmap -v -sn 192.168.0.0/16 10.0.0.0/8
  nmap -v -iR 10000 -Pn -p 80
SEE THE MAN PAGE (https://nmap.org/book/man.html) FOR MORE OPTIONS AND EXAMPLES
```

````

# 确定目标

指定扫描主机的所有IP地址在DNS服务器中，一个域名可以解析到多个IP地址。如果一个域名有多个IP地址时，Nmap默认仅探测第一个IP地址。为了能够探测到所有的IP地址，可以使用--resolve-all选项指定扫描主机中的所有IP地址。

## 指定单一目标

Nmap可以指定一个IP地址作为渗透测试的目标。指定的IP地址必须是与Nmap所在主机能够互相连通的IP地址。如果目标主机有多块网卡，则每个网卡都有一个IP地址。指定的时候需要确认哪个IP地址是可以连通的。如果用户指定的是本机，则可以指定任意一个IP地址。

```shell
nmap 192.168.1.1
```

## 指定多个主机

指定多个主机的格式可以是CIDR格式、连续主机、不连续主机或列表文件。

### CIDR格式

无类别域间路由（Classless Inter-Domain Routing，CIDR）可以将路由集中起来，在路由表中更灵活地定义地址。用户通过使用CIDR格式，可以指定一个网络内的所有主机。

Nmap工具提供了一个`-sL`选项，可以列出扫描的目标。用户可以使用该选项来验证指定的目标主机范围。该选项及其含义如下：

`-sL：列出扫描的目标。`

> sL是Scan List（扫描列表）的缩写。Nmap扫描类选项的缩写有具体的规定。其中，Scan缩写为小写字母s，特定项目List缩写为大写字母L。

```she
nmap 192.168.1.0/24 -sL
```

### 连续的主机

如果需要扫描多个IP地址连续的主机，则可以使用连字符格式。在一个IP地址中，连字符可以使用一次或多次。

实施扫描时，可以先使用-sL选项查看指定的连续主机的IP地址，以确定指定的主机范围是否正确。

```shell
nmap -sL 192.168.1.10-100
```

### 使用列表文件

当主机较多并且不连续时，可以将这些主机保存到一个列表文件中，然后直接使用这个列表文件指定多个主机。

Nmap提供了一个`-iL`选项用于指定主机列表文件。该选项及其含义如下：

`-iL <inputfilename>`：指定主机或网络列表文件。

> iL是Input from List（从列表中读取）的缩写。

```shell
nmap -iL host.txt -sL
```

### 指定扫描主机的所有IP地址

在DNS服务器中，一个域名可以解析到多个IP地址。如果一个域名有多个IP地址时，Nmap默认仅探测第一个IP地址。为了能够探测到所有的IP地址，可以使用`--resolve-all`选项指定扫描主机中的所有IP地址。

```shell
nmap --resolve-all www.baidu.com -sL
```

## 排除主机被扫描

Nmap提供了两个选项用来排除主机被扫描。这两个选项及其含义如下：

`--exclude <host1,host2...>`：指定排除的主机。其中，用户可以指定排除单个目标、多个目标或一个目标范围。

`--excludefile <exclude_file>`：指定排除的主机文件列表。

> exclude是排除的意思。

比如，扫描192.168.1.0/24网络中的所有主机，但是不扫描192.168.1.0-100范围内的主机。

```shell
nmap 192.168.1.0/24 --exclude 192.168.1.0-100 -sL
```

## 扫描方法

### 全部扫描

全部扫描就是对目标主机进行完整、全面的扫描，如主机状态、开放端口、操作系统类型和主机漏洞等。Nmap提供了一个`-A`选项，可以对目标主机实施全部扫描。该选项及其含义如下：

`-A`：实施全部扫描，以探测目标主机的操作系统和版本信息。

```shell
nmap 192.168.1.1 -A
```

# 发现主机

发现主机就是指探测网络中活动的主机。发现主机是实施渗透测试的第一步，也是最重要的一步。只有先确定目标主机是活动的，才能进一步实施渗透测试。在Nmap中，可以通过不同的方法来发现主机，如IP发现、ICMP发现、TCP发现和UDP发现等。

### IP发现

网际协议（Internet Protocol，IP）是TCP/IP体系中的网络层协议。IP是整个TCP/IP簇的核心，也是构成互联网的基础。IP位于TCP/IP模型的网络层，对上可载送传输层的各种协议信息，如TCP和UDP等；对下可将IP数据包放到链路层，通过以太网和令牌网络等各种技术来传送。

为了标识传输层协议类型，IP报文包含一个字段Protocol，用于保存传输层协议编号。Nmap允许用户基于IP构建探测数据包，用于构建不同类型的传输层数据包。通过字段Protocol，用户可以设置当前包采用哪种传输层协议。一旦目标响应这类数据包，就证明主机存在。这种探测方式称为IP发现。Nmap中的-PO选项可以用来实现IP发现。该选项及其含义如下：

`-PO <protocol list>`：使用IP数据包探测目标主机是否开启。其中，`protocol list`是协议编号列表。用户可以指定多个IP编号，如`6（TCP）`、`17（UDP）`、`1（ICMP）`和`2（IGMP）`。多个编号之间使用逗号分隔。如果没有指定协议，默认协议为`1（ICMP）`、`2（IGMP）`和`4（IP）`，等同于`-PO1,2,4`。





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


