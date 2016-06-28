# awesome-nmap-grep 💥

A collection of awesome, _grep-like_ commands for the `nmap` greppable output
(`-oG`) format. This repository aims to serve as a quick reference to modify the
 output into readable formats.

All of the below commands assume the output was saved to a file called
`output.grep`.

# commands
* [Count Number of Open Ports](#count-number-of-open-ports)
* [Top 10 Open Ports](#print-the-top-10-ports)

## count number of open ports

### command

```bash
egrep -v "^#|Status: Up" output.grep | cut -d' ' -f2 -f4- | awk '{printf "Host: %-20s Ports Open: %d\n" , $1, NF-1}' | sort -k 5 -g
```

### explained
```bash
$ egrep -v "^#|Status: Up" output.grep | cut -d' ' -f2 -f4- | \
#   |   |       |               |           |       |   |     └─ Continue the
#   |   |       |               |           |       |   |         command.
#   |   |       |               |           |       |   |
#   |   |       |               |           |       |   └─ Select the rest of
#   |   |       |               |           |       |       the fields which
#   |   |       |               |           |       |       will be the open
#   |   |       |               |           |       |       ports.
#   |   |       |               |           |       |
#   |   |       |               |           |       └─ Select the second field
#   |   |       |               |           |           to print which will
#   |   |       |               |           |           be IP Address
#   |   |       |               |           |
#   |   |       |               |           └─ Cut the lines by the space
#   |   |       |               |               delimiter.
#   |   |       |               |
#   |   |       |               └─ The file containing the grepable output.
#   |   |       |
#   |   |       └─ Ignore lines that start with a # or contain the string
#   |   |           'Status: Up'
#   |   |
#   |   └─ Inverse the pattern match
#   |
#   └─ Print lines matching a pattern
#
    awk '{printf "Host: %-20s Ports Open: %d\n" , $1, NF-1}' | sort -k 5 -g
#   |       |               |                      |    |       |
#   |       |               |                      |    |       └─ Sort the
#   |       |               |                      |    |           resultant
#   |       |               |                      |    |           string
#   |       |               |                      |    |           numerically.
#   |       |               |                      |    |
#   |       |               |                      |    └─ Number fields
#   |       |               |                      |        available, minus the
#   |       |               |                      |        one already matched
#   |       |               |                      |        for the IP.
#   |       |               |                      |
#   |       |               |                      └─ The IP Address
#   |       |               |
#   |       |               └─ Pad the string to 20 spaces
#   |       |
#   |       └─ Print a string using a format string
#   |
#   └─ Scan for patterns and process them.
```

## print the top 10 ports

### command

```bash
egrep -v "^#|Status: Up" output.grep | cut -d' ' -f4- | sed -n -e 's/Ignored.*//p' | tr ',' '\n' | sed -e 's/^[ \t]*//' | sort -n | uniq -c | sort -k 1 -r | head -n 10
```

### explained
```bash
$ egrep -v "^#|Status: Up" output.grep | cut -d' ' -f4- | \
#   |   |       |               |           |       |   └─ Continue the
#   |   |       |               |           |       |       command.
#   |   |       |               |           |       |
#   |   |       |               |           |       └─ Select only the fields
#   |   |       |               |           |           with the port details.
#   |   |       |               |           |
#   |   |       |               |           └─ Cut the lines by the space
#   |   |       |               |               delimiter.
#   |   |       |               |
#   |   |       |               └─ The file containing the grepable output.
#   |   |       |
#   |   |       └─ Ignore lines that start with a # or contain the string
#   |   |           'Status: Up'
#   |   |
#   |   └─ Inverse the pattern match
#   |
#   └─ Print lines matching a pattern
    sed -n -e 's/Ignored.*//p' | tr ',' '\n' | sed -e 's/^[ \t]*//' |  \
#   |    |  |     |               |  |         |            └─ Whack spaces and
#   |    |  |     |               |  |         |                tabs.
#   |    |  |     |               |  |         |
#   |    |  |     |               |  |         └─ Stream Editor for filtering
#   |    |  |     |               |  |             and transforming text
#   |    |  |     |               |  |
#   |    |  |     |               |  └─ Replace commas with newlines.
#   |    |  |     |               |
#   |    |  |     |               └─ Translate or delete characters
#   |    |  |     |
#   |    |  |     └─ Remove text from the string 'Ignored' onwards
#   |    |  |
#   |    |  └─ Specify the script to execute.
#   |    |
#   |    └─ Be quiet on errors.
#   |
#   └─ Stream Editor for filtering and transforming text
    sort -n | uniq -c | sort -k 1 -r | head -n 10
#   |    |       |           |           └─ Print the first 10 lines.
#   |    |       |           |
#   |    |       |           └─ Sort the output again, this time by the
#   |    |       |               amount of occurrences in reverse.
#   |    |       |
#   |    |       └─ Count unique occurrences of lines, adding a counter.
#   |    |  
#   |    |
#   |    |
#   |    └─ Sort numerically.
#   |
#   └─ Sort lines numerically or alphabetically.
```
