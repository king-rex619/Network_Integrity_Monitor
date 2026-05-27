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
