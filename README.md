# Network_Integrity_Monitor

Network-Integrity-Monitor is a cybersecurity-focused project designed to monitor and analyze network activities for suspicious behaviour, unauthorized devices, and potential security threats within a local network environment.

# Step by step implementation 

The Network-Integrity-Monitor works by discovering connected devices, monitoring network traffic, analyzing packets and ports, detecting unusual behaviour, generating alerts, and logging suspicious activities for security analysis.

# SECTION 1 (IMPORTS)

```bash
 import argparse
    import hashlib
    import json
    import os
    import socket
    import subprocess
    import sys
    import time
    from concurrent.futures import ThreadPoolExecutor, as_completed
    from datetime import datetime
    from pathlib import Path
```
Modules Used & Their Functions:
argparse → Handles command-line arguments and allows users to run commands like init, check, and update directly from the terminal.
hashlib → Generates secure SHA-256 hashes used to create unique fingerprints for files and data integrity verification.
json → Stores and retrieves monitoring data in a structured format for saving baselines and configuration information.
socket → Enables network communication and port checking to determine whether specific ports are open or active on devices.
subprocess → Allows the program to execute system commands such as ping for network connectivity testing.
sys → Provides control over script execution, including handling errors and safely exiting the program when needed.
ThreadPoolExecutor → Improves performance by running multiple scanning tasks concurrently instead of sequentially.
datetime → Records timestamps and monitoring times for logs, reports, and baseline creation.
Path → Simplifies file and directory management across different operating systems.

# SECTION 2 (CONFIGURATION)

``` bash
BASELINE_DIR = Path.home() / ".netwatch" / "baselines"
    COMMON_PORTS = [21, 22, 23, 25, 53, 80, 110, 143, 443,
                    445, 3306, 3389, 5432, 6379, 8080, 8443]

    RESET  = "\033[0m"
    RED    = "\033[91m"
    GREEN  = "\033[92m"
    YELLOW = "\033[93m"
    CYAN   = "\033[96m"
    BOLD   = "\033[1m"
    DIM    = "\033[2m"
```
Configuration Settings:
BASELINE_DIR defines the storage location where NetWatch saves baseline snapshots and monitoring data.
On Kali Linux,
COMMON_PORTS contains a predefined list of frequently used network ports scanned by the tool during monitoring and integrity checks.
Examples include:
22 → SSH (Secure Remote Access)
80 → HTTP (Web Traffic)
443 → HTTPS (Secure Web Traffic)
3306 → MySQL Database Service
3389 → RDP (Remote Desktop Protocol)

These ports help identify active services, detect unusual activity, and improve network visibility.
Terminal Color Codes

The tool uses terminal color codes such as RED, GREEN, and YELLOW to improve readability and highlight important information.
Green → Successful operations
Red → Warnings or detected issues
Yellow → Notifications or ongoing processes

These are ANSI escape sequences interpreted by the terminal to display colored output for a clearer monitoring experience.

# SECTION 3 (HELPER FUNCTIONS)
``` bash
    def load_baseline(target):
    def save_baseline(target, data):
    def fingerprint(data):
    def ts():
```

What is a "function"? A function is a mini-program inside the main program. You give it something, it does a job, and gives something back. Think of it like a vending machine — you put in money (input), it gives you a snack (output).

baseline_path(target):Takes a target like "192.168.1.0/24" and converts it into a safe filename like "192-168-1-0_24.json". Slashes and dots are replaced because filenames can't contain those characters.

load_baseline(target): Opens the saved baseline file for a target and reads it into Python so we can compare it later. If no file exists yet, it returns nothing (None).

save_baseline(target, data):Saves the current scan results into a JSON file. ThisWhat is a "function"? A function is a mini-program inside the main program. You give it something, it does a job, and gives something back. Think of it like a vending machine — you put in money (input), it gives you a snack (output).

baseline_path(target): Takes a target like "192.168.1.0/24" and converts it into a safe filename like "192-168-1-0_24.json". Slashes and dots are replaced because filenames can't contain those characters.

load_baseline(target): Opens the saved baseline file for a target and reads it into Python so we can compare it later. If no file exists yet, it returns nothing (None).

save_baseline(target, data): Saves the current scan results into a JSON file. This is the "photo" the security guard takes.

fingerprint(data): Takes all the scan data, converts it to text, then runs SHA-256 hashing on it to create a unique code. Example: Data: {"192.168.1.1": {"ports": [22, 80]}} Hash: a3f5c8d2e1b9... (long unique string)

If even ONE character in the data changes, the hash will be completely different. This is how we know if someone tampered with the baseline file itself.

ts(): Short for "timestamp". Simply returns the current date and time as a readable string. Example output: "2026-05-22 14:30:00"

# SECTION 4 (NETWORK SCANNING FUNCTIONS)
``` bash
    def ping(ip):
    def resolve(ip):
    def scan_port(ip, port):
    def scan_host(ip, ports):
    def expand_cidr(cidr):
    def do_scan(targets, ports, workers):
```
ping(ip): Sends a tiny test message to an IP address. If the device replies, it is alive (True). If nothing replies, it is offline (False). This works just like typing "ping 192.168.1.1" in your terminal — netwatch does it automatically for every IP in the range.

resolve(ip): Tries to find the name of a device from its IP. For example: 192.168.1.1 → "router.local" 192.168.1.42 → "nas.local" If no name is found, it returns "unknown". scan_port(ip, port): Tries to connect to one specific port on a device. If the connection works → port is OPEN (True). If the connection is refused or times out → CLOSED (False). This is like knocking on door number 22 of a house. If someone answers, the door is open. scan_host(ip, ports): Combines ping + resolve + scan_port for ONE device. First checks if the device is alive. If alive → resolves its name → checks all ports. Returns a dictionary (a record) like: { "ip": "192.168.1.1", "hostname": "router.local", "open_ports": [22, 80, 443], "last_seen": "2026-05-22 14:30:00" } If the device is offline → returns nothing (None). expand_cidr(cidr): Takes a subnet range like "192.168.1.0/24" and expands it into a full list of all IP addresses:["192.168.1.1", "192.168.1.2", ..., "192.168.1.254"] The /24 means 254 possible addresses. Without this, you would have to type every IP by hand. do_scan(targets, ports, workers=64): The main scanning engine. It takes a list of IPs and scans ALL of them at the same time using 64 parallel "workers" (threads). Without parallel scanning: Scan 254 IPs one by one = very slow (minutes) With 64 parallel workers: All 254 IPs scanned almost simultaneously = fast As hosts are found, it prints them live to the screen.

# SECTION 5 (COMMANDS)

These are the main actions the user can run. cmd_init(target, ports, force): This runs when you type: netwatch init 192.168.1.0/24 Step by step:

1. Checks if a baseline already exists for this target.
2. If it exists and --force was NOT used → stops and warns you (to avoid overwriting accidentally).
3. Expands the CIDR range into individual IPs.
4. Runs do_scan() to scan all of them.
5. Saves all results + a SHA-256 fingerprint into a JSON file inside ~/.netwatch/baselines/.
6. Prints how many hosts were found. ───────────────────────────────── cmd_check(target, ports): ───────────────────────────────── This runs when you type: netwatch check 192.168.1.0/24
  This is the most important command. Here is what it does:

1. Loads the saved baseline (old photo).
2. Runs a fresh scan (takes a new photo).
3. Compares old vs new — looks for 3 types of changes: NEW HOST: An IP that appears in the new scan but was NOT in the baseline. Could be an unauthorized device! MISSING HOST: An IP that was in the baseline but did NOT respond in the new scan. Device might be off, or unplugged from the network. PORT CHANGE: A known host now has different ports open. Maybe someone started a new service (opened a port) or shut one down (closed a port). Example: Port 23 (Telnet) suddenly opened on your router = suspicious!

4. If nothing changed → prints green "Unchanged" message.
5. If changes found → prints red/yellow alerts for each one.
───────────────────────────────── cmd_update(target, ports): ───────────────────────────────── This runs when you type: netwatch update 192.168.1.0/24 Use this when YOU made a legitimate change to the network and want netwatch to accept the new state.

For example:
* You added a new computer → update baseline.


*  You opened port 443 on your server → update baseline.It rescans and overwrites the old baseline.
───────────────────────────────── cmd_list(): ───────────────────────────────── This runs when you type: netwatch list Lists all the baseline files currently stored. Shows the target, how many hosts, and when it was last scanned.

───────────────────────────────── cmd_remove(target): ───────────────────────────────── This runs when you type: netwatch remove 192.168.1.0/24 Deletes the baseline file for that target. Use this if you no longer want to monitor a specific network range.

# SECTION 6 (ARGUMENT PARSING)-Lines 263–320
``` bash
def parse_ports(s):
    def main():
```
parse_ports(s): Converts the --ports argument from a string into a list of numbers. Examples: "22,80,443" → [22, 80, 443] "1-1024" → [1, 2, 3, ..., 1024] "22,80,1-100"→ [22, 80, 1, 2, ..., 100] main(): This is the entry point — the very first thing that runs when you execute netwatch.

It uses argparse to:

1. Read what command you typed (init/check/update/etc.)
2. Read any extra options (--ports, --force)
3. Call the correct function based on the command Think of it as the receptionist of the program. You walk in, tell it what you want, and it directs you to the right department.

# SECTION 7 — HOW IT ALL CONNECTS (The Big Picture)

Here is the full flow in plain English:

YOU TYPE: netwatch init 192.168.1.0/24 ↓ main() reads "init" and calls cmd_init() ↓ cmd_init() calls expand_cidr() to get all 254 IPs ↓ cmd_init() calls do_scan() with those 254 IPs ↓ do_scan() uses 64 threads to call scan_host() for each IP ↓ scan_host() calls ping() → resolve() → scan_port() x16 ↓ Results collected → fingerprint() creates a SHA-256 hash ↓ save_baseline() writes everything to a JSON file ↓ Done! "✔ Baseline stored"

────────────────────────────────────

YOU TYPE: netwatch check 192.168.1.0/24 ↓ main() reads "check" and calls cmd_check() ↓ load_baseline() reads the old JSON file ↓ do_scan() runs a fresh scan of the same range ↓ Old results vs New results are compared ↓ Differences become alerts (NEW HOST, PORT CHANGE, etc.) ↓ Alerts printed to the terminal in red/yellow ↓ Done!

# SECTION 8 (KEY CONCEPT SUMMARISED)

CONCEPT WHAT IT MEANS IN SIMPLE TERMS ───────────────────────────────────────────────────────── Baseline A saved snapshot of your network state Hash (SHA-256) A unique fingerprint made from data CIDR (/24) A shorthand for a range of IP addresses Port A numbered "door" on a computer for a specific type of network traffic Thread A mini-task running in parallel with others JSON A simple text format for storing structured data Ping A test message to check if a device is online Resolve Looking up the name of a device by its IP

# SECTION 9 (THE BASELINE FILE) What Gets Saved

After running "netwatch init", a file like this is created at: /root/.netwatch/baselines/192-168-1-0_24.json

{ "target": "192.168.1.0/24", "ports": [22, 80, 443, ...], "created_at": "2026-05-22 14:30:00", "hosts": { "192.168.1.1": { "ip": "192.168.1.1", "hostname": "router.local", "open_ports": [22, 80, 443], "last_seen": "2026-05-22 14:30:00" }, "192.168.1.42": { "ip": "192.168.1.42", "hostname": "nas.local", "open_ports": [22, 445], "last_seen": "2026-05-22 14:30:00" } }, "checksum": "a3f5c8d2e1b94f67..." }

The "checksum" at the bottom is the SHA-256 hash of all the hosts data. If someone edits this file manually, the checksum will no longer match — and netwatch will know the baseline itself was tampered with.





# === END OF EXPLANATION
move the file to /usr/local/bin/
``` bash
sudo mv netwatch /usr/local/bin/
```
exicute the file
``` bash
chmod +x netwatch
```
run the file using a designated ip address
``` bash
netwatch (ip adder)
```
