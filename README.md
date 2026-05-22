# Network_Integrity_Monitor

# Step by step implementation 
Section 1 (Imports)
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

