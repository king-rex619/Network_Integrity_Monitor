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
