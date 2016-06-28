# awesome-nmap-grep 💥

A collection of awesome, _grep-like_ commands for the `nmap` greppable output
(`-oG`) format. This repository aims to serve as a quick reference to modify the
 output into readable formats.

All of the below commands assume the output was saved to a file called
`output.grep`. The example command to produce this file as well as the sample
outputs was: `nmap -v --reason 127.0.0.1 -sV -oG output.grep -p-`.

Finally, the `NMAP_FILE` variable is set to contain `output.grep`.

# commands
* [Count Number of Open Ports](#count-number-of-open-ports)
* [Top 10 Open Ports](#print-the-top-10-ports)
* [Top Service Identifiers](#top-service-identifiers)
* [Top Service Names](#top-service-names)

## count number of open ports

### command
```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d' ' -f2 -f4- | \
sed -n -e 's/Ignored.*//p' | \
awk -F, '{split($0,a," "); printf "Host: %-20s Ports Open: %d\n" , a[1], NF}' \
| sort -k 5 -g
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

## top service names

### command

```bash
NMAP_FILE=output.grep
egrep -v "^#|Status: Up" $NMAP_FILE | cut -d ' ' -f4- | tr ',' '\n' | \
sed -e 's/^[ \t]*//' | awk -F '/' '{print $5}' | grep -v "^$" | sort | uniq -c \
| sort -k 1 -nr
```
