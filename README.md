# awesome-nmap-grep 💥

A collection of awesome, _grep-like_ commands for the `nmap` greppable output
(`-oG`) format. This repository aims to serve as a quick reference to modify the
 output into readable formats.

All of the below commands assume the output was saved to a file called
`output.grep`. The example command to produce this file as well as the sample
outputs was: `nmap -v --reason 127.0.0.1 -oG output.grep -p-`.

Finally, the `NMAP_FILE` variable is set to contain `output.grep`.

# commands
* [Count Number of Open Ports](#count-number-of-open-ports)
* [Top 10 Open Ports](#print-the-top-10-ports)
* [Top Service Identifiers](#top-service-identifiers)
* [Top Service Names](#top-service-names)
* [Hosts and Open Ports](#hosts-and-open-ports)
* [Banner Grab](#banner-grab)

## count number of open ports

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f2 -f4- | \
sed -n -e 's/Ignored.*//p' | \
awk -F, '{split($0,a," "); printf "Host: %-20s Ports Open: %d\n" , a[1], NF}' \
| sort -k 5 -g
```

### output
```bash
Host: 127.0.0.1            Ports Open: 16
```

### explained
```bash
$ NMAP_FILE=output.grep
$ egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f2 -f4- | \
#        | └──────┬──────┘      |                  |   └─ Select the rest of
#        |        |             |                  |       the fields which
#        |        |             |                  |       will be the open
#        |        |             |                  |       ports.
#        |        |             |                  |
#        |        |             |                  └─ Select the second field
#        |        |             |                      to print which will
#        |        |             |                      be IP Address
#        |        |             |
#        |        |             └─ The file containing the grepable output.
#        |        |
#        |        └─ Ignore lines that start with a # or contain the string
#        |            'Status: Up'
#        |
#        └─ Inverse the pattern match
    sed -n -e 's/Ignored.*//p' | \
#        |  | └──────┬───────┘
#        |  |        └─ Remove text from the string 'Ignored' onwards.
#        |  |
#        |  └─ Specify the script to execute.
#        |
#        └─ Be quiet on errors.
    awk -F, '{split($0,a," "); printf "Host: %-20s Ports Open: %d\n" , a[1], NF}' | \
#        |    └──────┬──────┘                └─┬─┘                     └─┬─┘ |
#        |           |                         |  Use the second element ┘   |            
#        |           |                         |   in array a defined by     |
#        |           |                         |   the previous split().     |
#        |           |                         |                             |
#        |           |                         |      The total columns ─────┘
#        |           |                         |        extracted.
#        |           |                         |      
#        |           |                         └─ Pad the string to 20 spaces.
#        |           |                     
#        |           └─ Split the item in the first column again by space,
#        |               storing the resultant array into a.
#        |
#        └─ Print a string from a format string
    sort -k 5 -g
```

## print the top 10 ports

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f4- | \
sed -n -e 's/Ignored.*//p' | tr ',' '\n' | sed -e 's/^[ \t]*//' | \
sort -n | uniq -c | sort -k 1 -r | head -n 10
```

### output
```bash
1 9001/open/tcp//tor-orport?///
1 9000/open/tcp//cslistener?///
1 8080/open/tcp//http-proxy///
1 80/open/tcp//http//Caddy/
1 6379/open/tcp//redis//Redis key-value store/
1 631/open/tcp//ipp//CUPS 2.1/
1 6234/open/tcp/////
1 58377/filtered/tcp/////
1 53/open/tcp//domain//dnsmasq 2.76/
1 49153/open/tcp//mountd//1-3/
```

### explained
```bash
$ NMAP_FILE=output.grep
$ egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f4- | \
#        | └──────┬──────┘      |                  └─ Select only the fields
#        |        |             |                      with the port details.
#        |        |             |
#        |        |             └─ The file containing the grepable output.
#        |        |
#        |        └─ Ignore lines that start with a # or contain the string
#        |            'Status: Up'
#        |
#        └─ Inverse the pattern match
    sed -n -e 's/Ignored.*//p' | tr ',' '\n' | sed -e 's/^[ \t]*//' |  \
#        |  | └──────┬───────┘      └┬┘ └─┬┘          └─────┬─────┘
#        |  |        |               |    |                 └─ Remove tabs and
#        |  |        |               |    |                      spaces.
#        |  |        |               |    └─ ... with newlines.
#        |  |        |               |
#        |  |        |               └─ Replace commas ...
#        |  |        |
#        |  |        └─ Remove text from the string 'Ignored' onwards.
#        |  |
#        |  └─ Specify the script to execute.
#        |
#        └─ Be quiet on errors.
    sort -n | uniq -c | sort -k 1 -r | head -n 10
#         |         |              |           └─ Print the first 10 lines.
#         |         |              |
#         |         |              └─ Output result in reverse
#         |         |
#         |         └─ Count occurrences
#         |
#         └─ Sort numerically.
```

## top service identifiers

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d ' ' -f4- | tr ',' '\n' | \
sed -e 's/^[ \t]*//' | awk -F '/' '{print $7}' | grep -v "^$" | sort | uniq -c \
| sort -k 1 -nr
```

### output
```bash
2 Caddy
2 1-3 (RPC 100005)
1 dnsmasq 2.76
1 Redis key-value store
1 OpenSSH 6.9 (protocol 2.0)
1 MySQL 5.5.5-10.1.14-MariaDB
1 CUPS 2.1
```

## top service names

### command

```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d ' ' -f4- | tr ',' '\n' | \
sed -e 's/^[ \t]*//' | awk -F '/' '{print $5}' | grep -v "^$" | sort | uniq -c \
| sort -k 1 -nr
```

### output
```bash
2 mountd
2 http
1 unknown
1 tor-orport?
1 ssl|https
1 ssh
1 redis
1 mysql
1 ipp
1 http-proxy
1 domain
1 cslistener?
```

## hosts and open ports

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f2 -f4- | \
sed -n -e 's/Ignored.*//p'  | \
awk '{print "Host: " $1 " Ports: " NF-1; for (i=2; i<=NF; i++) { split($i,a,"/"); printf "%-8s %s/%-5s %s\n" , a[2], a[3], a[1], a[5]}; }'
```

### output
```bash
Host: 127.0.0.1 Ports: 16
open     tcp/22    ssh
open     tcp/53    domain
open     tcp/80    http
open     tcp/443   https
open     tcp/631   ipp
open     tcp/3306  mysql
open     tcp/4767  unknown
open     tcp/6379
open     tcp/8080  http-proxy
open     tcp/8081  blackice-icecap
open     tcp/9000  cslistener
open     tcp/9001  tor-orport
open     tcp/49152 unknown
open     tcp/49153 unknown
filtered tcp/54695
filtered tcp/58369
```

## banner grab

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d ' ' -f2 -f4- | awk -F, '{split($1,a," "); split(a[2],b,"/"); print a[1] " " b[1]; for(i=2; i<=NF; i++) { split($i,c,"/"); print a[1] c[1] }}' | xargs -L1 nc -v -w1
```

### output
*Sample*
```bash
found 0 associations
found 1 connections:
     1:	flags=82<CONNECTED,PREFERRED>
	outif lo0
	src 127.0.0.1 port 52224
	dst 127.0.0.1 port 3306
	rank info not available
	TCP aux info available

Connection to 127.0.0.1 port 3306 [tcp/mysql] succeeded!
Y
5.5.5-10.1.14-MariaDB�uds9^MIf��!?�EgVZ>iv7KTD7mysql_native_passwordfound 0 associations

nc: connectx to 127.0.0.1 port 54695 (tcp) failed: Connection refused
nc: connectx to 127.0.0.1 port 58369 (tcp) failed: Connection refused
```
