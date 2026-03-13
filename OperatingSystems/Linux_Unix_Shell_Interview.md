# Linux/Unix & Shell Scripting – Interview Notes (IBKR JD)

## Table of Contents
1. [Linux Fundamentals](#1-linux-fundamentals)
2. [File System & Permissions](#2-file-system--permissions)
3. [Process Management](#3-process-management)
4. [Networking Commands](#4-networking-commands)
5. [Shell Scripting (Bash)](#5-shell-scripting-bash)
6. [PERL Scripting](#6-perl-scripting)
7. [Log Processing & Text Processing](#7-log-processing--text-processing)
8. [Cron Jobs & Scheduling](#8-cron-jobs--scheduling)
9. [Common Interview Questions](#9-common-interview-questions)

---

## 1. Linux Fundamentals

### Essential Commands
```bash
# Navigation
pwd          # print working directory
ls -la       # list all files with permissions
cd /var/log  # change directory

# File operations
cp -r src/ dest/          # copy recursively
mv old_name new_name      # move/rename
rm -rf /tmp/old_data      # remove recursively (CAREFUL!)
mkdir -p /data/trades/2024  # create nested dirs

# File viewing
cat file.txt         # print whole file
head -20 trades.log  # first 20 lines
tail -f app.log      # follow live log (great for monitoring)
less trades.log      # paginated view
wc -l file.txt       # count lines

# Searching
grep -rn "ERROR" /var/log/app/    # recursive grep with line numbers
grep -i "aapl" trades.csv         # case-insensitive
grep -v "DEBUG" app.log           # exclude DEBUG lines
find /data -name "*.csv" -mtime -7  # files modified in last 7 days
find /tmp -size +100M              # files > 100MB
```

### Environment Variables
```bash
export JAVA_HOME=/usr/lib/jvm/java-17
export PATH=$PATH:$JAVA_HOME/bin
export ORACLE_HOME=/opt/oracle/19c
export NLS_DATE_FORMAT="YYYY-MM-DD"

echo $JAVA_HOME        # print variable
env | grep ORACLE      # list all env vars, filter Oracle
source ~/.bashrc       # reload profile
```

---

## 2. File System & Permissions

### Permission Structure
```
-rwxr-xr-- 1 ibkr staff 2048 Mar 10 09:00 trade_loader.sh
 |||||||||||
 |└──┬──┘└──┬──┘└──┬──┘
 |   owner  group  others
 └ type: - file, d dir, l symlink

r=4, w=2, x=1
rwx = 7, r-x = 5, r-- = 4
```

```bash
# chmod
chmod 755 script.sh   # rwxr-xr-x (owner can write, others can read+execute)
chmod u+x script.sh   # add execute for owner
chmod -R 644 /data    # recursively set 644

# chown
chown ibkr:staff file.txt   # change owner and group
sudo chown -R app:app /opt/app

# Special bits
chmod +s executable   # setuid/setgid
chmod +t /tmp         # sticky bit
```

### Disk Usage
```bash
df -h              # disk space by filesystem
du -sh /data/      # disk usage of directory
du -sh * | sort -rh | head -10  # top 10 largest items
lsof +D /var/log   # list open files in directory
```

---

## 3. Process Management

```bash
# View processes
ps aux             # all processes
ps aux | grep java # find java processes
top                # interactive process viewer
htop               # better interactive viewer

# Process control
kill -9 PID        # force kill (SIGKILL)
kill -15 PID       # graceful kill (SIGTERM)
killall java       # kill all java processes

# Background jobs
./long_running.sh &           # run in background
nohup ./server.sh &           # run immune to hangup (survives logout)
jobs                          # list background jobs
fg %1                         # bring job 1 to foreground

# System resources
free -h            # memory usage
vmstat 1 5         # memory/cpu stats every 1s, 5 times
iostat -x 1        # I/O stats
ulimit -n          # open file limit (important for large systems)
ulimit -n 65536    # increase fd limit

# Check ports
netstat -tlnp      # listening ports with PID
ss -tlnp           # faster alternative to netstat
lsof -i :8080      # what's using port 8080
```

---

## 4. Networking Commands

```bash
# Connectivity
ping google.com          # basic connectivity
traceroute google.com    # route hops
curl -I https://api.ibkr.com   # HTTP headers only
curl -o /tmp/data.json https://api.example.com/data  # download
wget -q -O output.csv http://data.example.com/file.csv

# Transfer
scp user@host:/remote/file.csv /local/path/    # secure copy
rsync -avz /local/ user@host:/backup/          # sync with progress

# DNS
nslookup hostname.ibkr.com
dig hostname.ibkr.com

# Network stats
netstat -s     # network statistics
ss -s          # socket statistics summary
```

---

## 5. Shell Scripting (Bash)

### Script Template
```bash
#!/bin/bash
# Script: load_trades.sh
# Author: IBKR Dev Team
# Description: Load daily trade files to Oracle

set -euo pipefail   # exit on error, undefined vars, pipe failures

# Variables
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/ibkr/trade_loader_$(date +%Y%m%d).log"
DATA_DIR="/data/trades"
ORACLE_USER="ibkr_user"

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

log "Starting trade load..."
```

### Control Flow
```bash
# If-else
if [[ -f "$DATA_DIR/trades.csv" ]]; then
    log "File found, processing..."
elif [[ -d "$DATA_DIR" ]]; then
    log "Directory exists but no file"
else
    log "ERROR: data directory not found"
    exit 1
fi

# Case statement
case "$ENVIRONMENT" in
    prod)   DB_HOST="prod-oracle.ibkr.com" ;;
    uat)    DB_HOST="uat-oracle.ibkr.com"  ;;
    dev)    DB_HOST="localhost"             ;;
    *)      echo "Unknown env: $ENVIRONMENT"; exit 1 ;;
esac

# For loop
for symbol in AAPL GOOGL MSFT AMZN; do
    log "Processing $symbol..."
    java -jar trade_loader.jar --symbol "$symbol" --date "$(date +%Y%m%d)"
done

# While loop with file reading
while IFS=',' read -r symbol quantity price; do
    echo "Symbol: $symbol, Qty: $quantity, Price: $price"
done < trades.csv
```

### Functions
```bash
check_oracle_connection() {
    local db_user=$1
    local db_pass=$2
    local db_host=$3
    sqlplus -s "${db_user}/${db_pass}@${db_host}" <<EOF
        SELECT 1 FROM dual;
        EXIT;
EOF
    return $?
}

# Call
if check_oracle_connection "$ORACLE_USER" "$ORACLE_PASS" "$DB_HOST"; then
    log "Oracle connection OK"
else
    log "ERROR: Cannot connect to Oracle"
    exit 1
fi
```

### Array Operations
```bash
# Indexed array
symbols=("AAPL" "GOOGL" "MSFT")
echo "${symbols[0]}"         # AAPL
echo "${#symbols[@]}"        # 3 (length)
symbols+=("AMZN")            # append

# Associative array
declare -A prices
prices[AAPL]=175.50
prices[GOOGL]=140.20

for sym in "${!prices[@]}"; do
    echo "$sym: ${prices[$sym]}"
done
```

### Error Handling & Cleanup
```bash
cleanup() {
    log "Cleaning up temporary files..."
    rm -f /tmp/trade_batch_*.tmp
}
trap cleanup EXIT          # always run on exit
trap 'log "Interrupted!"; exit 1' INT TERM

# Check return codes
sqlplus "$ORACLE_USER/$ORACLE_PASS@$DB_HOST" @load_data.sql
if [[ $? -ne 0 ]]; then
    log "ERROR: SQL load failed"
    exit 1
fi
```

---

## 6. PERL Scripting

### Basics
```perl
#!/usr/bin/perl
use strict;
use warnings;

# Variables
my $symbol  = "AAPL";
my $price   = 175.50;
my @symbols = ("AAPL", "GOOGL", "MSFT");
my %prices  = (AAPL => 175.50, GOOGL => 140.20);

print "Symbol: $symbol, Price: $price\n";
print "Symbols: @symbols\n";
print "AAPL price: $prices{AAPL}\n";
```

### File Processing (IBKR trade files)
```perl
#!/usr/bin/perl
use strict;
use warnings;

my $input_file  = "/data/trades/trades_20240310.csv";
my $output_file = "/data/processed/output_20240310.csv";
my $error_count = 0;

open(my $in,  '<', $input_file)  or die "Cannot read $input_file: $!";
open(my $out, '>', $output_file) or die "Cannot write $output_file: $!";

while (my $line = <$in>) {
    chomp $line;
    next if $line =~ /^#/;       # skip comments
    next if $line =~ /^\s*$/;    # skip blank lines

    my ($symbol, $qty, $price, $date) = split(/,/, $line);

    # Validate
    unless ($symbol && $qty > 0 && $price > 0) {
        warn "Invalid line: $line\n";
        $error_count++;
        next;
    }

    # Transform: uppercase symbol
    $symbol = uc($symbol);
    printf $out "%s,%d,%.2f,%s\n", $symbol, $qty, $price, $date;
}

close($in); close($out);
print "Processing complete. Errors: $error_count\n";
```

### Regex (PERL excels here)
```perl
# Extract trade ID from log
while (<STDIN>) {
    if (/TRADE_ID=(\d+).*SYMBOL=(\w+).*STATUS=(\w+)/) {
        print "ID: $1, Symbol: $2, Status: $3\n";
    }
}

# Substitute
$line =~ s/NYSE/NEW_YORK_STOCK_EXCHANGE/g;

# Split on multiple delimiters
my @fields = split(/[,|;]/, $record);
```

### DBI (Database Interface)
```perl
use DBI;

my $dbh = DBI->connect(
    "dbi:Oracle:host=oracle.ibkr.com;sid=PROD",
    "ibkr_user", "password",
    { RaiseError => 1, AutoCommit => 0 }
) or die "Cannot connect: $DBI::errstr";

my $sth = $dbh->prepare("SELECT symbol, price FROM market_data WHERE exchange = ?");
$sth->execute("NYSE");

while (my ($symbol, $price) = $sth->fetchrow_array()) {
    printf "%-10s %.2f\n", $symbol, $price;
}

$dbh->commit();
$dbh->disconnect();
```

---

## 7. Log Processing & Text Processing

```bash
# AWK – column-based processing
awk -F',' '{print $1, $3}' trades.csv          # print columns 1 and 3
awk -F',' '$2 > 1000000 {print $1}' trades.csv  # filter by value
awk -F',' '{sum += $3} END {print "Total:", sum}' trades.csv  # sum column 3

# AWK for log analysis
awk '/ERROR/ {count++} END {print count " errors found"}' app.log

# SED – stream editing
sed 's/,/|/g' trades.csv               # replace , with |
sed '/^#/d' config.txt                 # delete comment lines
sed -n '10,20p' large_file.txt         # print lines 10-20
sed -i 's/old_host/new_host/g' config  # in-place edit

# Sort and unique
sort -t',' -k3,3n trades.csv           # sort by column 3 numerically
sort -t',' -k1,1 -k2,2rn data.csv     # sort by col1 asc, col2 desc
sort data.txt | uniq -c | sort -rn     # count frequencies

# Cut
cut -d',' -f1,3 trades.csv             # extract columns 1 and 3
cut -d':' -f1 /etc/passwd              # extract usernames

# Paste (merge files)
paste -d',' file1.csv file2.csv        # merge side by side

# xargs
find /data -name "*.csv" | xargs wc -l   # count lines in all CSVs
cat trade_ids.txt | xargs -I{} curl http://api/trade/{}  # API calls
```

---

## 8. Cron Jobs & Scheduling

```bash
# Crontab format: minute hour day month weekday command
# *    *    *    *       *
# 0-59 0-23 1-31 1-12  0-7(0,7=sun)

# Edit crontab
crontab -e
crontab -l   # list
crontab -r   # remove all

# Examples
# Run trade loader daily at 6:30 AM
30 6 * * 1-5 /opt/ibkr/scripts/load_trades.sh >> /var/log/cron_trade.log 2>&1

# Run every 15 minutes during market hours (9-4 PM Mon-Fri)
*/15 9-16 * * 1-5 /opt/ibkr/scripts/sync_market_data.sh

# First day of each month
0 0 1 * * /opt/ibkr/scripts/monthly_report.sh

# Redirect both stdout and stderr
0 2 * * * /script.sh >> /log/out.log 2>> /log/err.log
# Or combine:
0 2 * * * /script.sh >> /log/combined.log 2>&1
```

---

## 9. Common Interview Questions

| Question | Key Answer |
|---|---|
| **Difference between process and thread?** | Process: independent execution with own memory; Thread: shares process memory |
| **What is a zombie process?** | Process that finished but parent hasn't called `wait()` yet; shows as `<defunct>` |
| **What is `2>&1`?** | Redirect stderr (fd 2) to stdout (fd 1) |
| **What does `set -e` do in shell?** | Script exits immediately on any command error |
| **Hard link vs soft link?** | Hard link: same inode, same filesystem; Soft/Symlink: pointer to file path, can cross filesystems |
| **What is `nohup`?** | Runs command immune to SIGHUP (terminal close); output goes to `nohup.out` |
| **How to check memory in Linux?** | `free -h`, `cat /proc/meminfo`, `vmstat` |
| **What is swap?** | Disk space used as virtual memory when RAM is full; much slower than RAM |
| **What is `$?`?** | Exit status of last command; 0 = success, non-zero = error |
| **Explain shell pipe `|`?** | Connects stdout of left command to stdin of right command |
