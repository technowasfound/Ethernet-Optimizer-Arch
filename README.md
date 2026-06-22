This error occurs because some Markdown parsers and browser rendering engines
struggle with punctuation marks like colons (:) and ampersands (&) inside
internal anchor links.

To resolve this, the section headers have been simplified to use standard
alphanumeric English names (changing How it Works: Under the Hood to Technical
Overview, and replacing & with and). This ensures the generated fragment slugs
strictly adhere to standard GitHub Flavored Markdown (GFM) slugification rules
and resolve correctly.

Here is the revised, fully corrected README.md file with stable internal
navigation anchors:

# Ethernet Optimizer for Arch Linux (`ethoptimizer`)

A modular command-line network diagnostic and kernel tuning utility designed for Arch Linux and its derivatives (such as CachyOS, EndeavourOS, and Artix). 

This tool automates the benchmarking of your network connection, diagnoses potential hardware and configuration bottlenecks, and safely applies targeted kernel and driver-level optimizations. It is designed to mitigate packet jitter, lower latency spikes under load (bufferbloat), and maximize sustained throughput.

---

## Table of Contents
1. [Core Features](#core-features)
2. [Technical Overview](#technical-overview)
3. [Tuning Profiles](#tuning-profiles)
4. [Installation and Setup](#installation-and-setup)
5. [Command Reference](#command-reference)
6. [Safety and Rollbacks](#safety-and-rollbacks)
7. [Directory Structure](#directory-structure)

---

## Core Features

*   **Network Path Diagnostics:** Checks Maximum Transmission Unit (MTU), physical line negotiation speed, duplex configuration, current congestion control algorithm, and queries DNS resolving latency across multiple public servers.
*   **Throughput & Quality Benchmarking:** Safely measures round-trip time (RTT), packet jitter, and raw downstream/upstream bandwidth using standard HTTP socket transfers.
*   **Active Kernel Tuning (`sysctl`):** Modifies Linux kernel network parameters (such as TCP window scaling, selective acknowledgments, and socket memory allocations) dynamically.
*   **Hardware-Level Tuning (`ethtool`):** Adjusts driver ring buffers and enables hardware-assisted packet offloading options where supported by your physical interface card.
*   **Zero-Dependency Installer:** Packaged as a clean Bash and Python configuration builder that compiles and packages your setup without third-party dependencies.

---

## Technical Overview

To understand how `ethoptimizer` enhances your system's network performance, here is an explanation of the specific kernel network variables (`sysctl`) and hardware properties it adjusts:

### 1. Queuing Disciplines (Qdisc) & Bufferbloat Control
Standard Linux networks often buffer too much data in software queues before sending it to the network card. This creates **bufferbloat**—causing your latency to spike whenever a large download or upload occurs.
*   **`net.core.default_qdisc = fq` (Fair Queueing):** `ethoptimizer` replaces standard FIFO queues with Fair Queueing. `fq` paces packets out to the network adapter evenly instead of sending them in bursts, reducing packet drops and maintaining stable latency.

### 2. Congestion Control Algorithm
*   **`net.ipv4.tcp_congestion_control = bbr`:** By default, Linux uses **CUBIC**, which measures packet loss to determine network congestion. When a packet is dropped, CUBIC aggressively cuts your bandwidth in half. Google’s **BBR (Bottleneck Bandwidth and RTT)** algorithm measures actual bottleneck bandwidth and round-trip times instead of relying on packet loss. This allows your connection to maintain maximum throughput even on lossy lines.

### 3. Handshake and Connection Timing
*   **`net.ipv4.tcp_fastopen = 3`:** Allows client systems to send request payloads inside the initial SYN handshake packet. This reduces the latency of establishing new TCP sessions by skipping a complete round-trip.
*   **`net.ipv4.tcp_timestamps = 0` (Low Latency / Gaming):** Disabling TCP timestamps strips 10 bytes of timestamp headers from every single packet. This reduces packet processing overhead on your CPU and saves frame bandwidth, which can help stabilize sub-millisecond ping times.

### 4. TCP Window Sizing & Memory Buffers
To prevent fast connections (like gigabit fiber) from choking, the system must allow a large **TCP Window Scale**:
*   **`net.core.rmem_max` & `net.core.wmem_max`:** Increases the maximum allowed read and write socket buffer memory to allow window scaling to open wider on high-bandwidth, long-distance routes.

---

## Tuning Profiles

You can switch between predefined profiles configured in `/etc/ethoptimizer/config.conf` to match your target workload:

| Profile Name | Objective | Primary Configurations Applied |
| :--- | :--- | :--- |
| **`gaming`** | Minimizes latency jitter and packet delays. | `tcp_timestamps` disabled, lower coalescence interval, Fair Queueing (`fq`) scheduler, BBR enabled. |
| **`streaming`**| Prevents buffer starvation during long bulk downloads. | Moderate buffer scaling (32MB limit), selective acknowledgments (`tcp_sack`) enabled, `fq_codel` scheduler. |
| **`server`**   | Optimizes system for high concurrency. | Elevated memory window values, maximum socket backlogs, BBR enabled. |
| **`max-perf`** | Maximizes raw throughput on high-speed cards. | Maximum kernel TCP buffer sizes (128MB limit), maximum physical RX/TX buffer rings (4096), TCP hardware offloads (TSO, GSO, GRO) forced on. |
| **`general`**  | Standard, safe system tuning. | Safe default buffer allocations, cubic congestion controls, standard TCP timestamps. |

---

## Installation and Setup

Follow these steps to deploy and run `ethoptimizer` globally on your Arch system.

### 1. Install System Dependencies
Verify that your system has the standard network diagnostic utilities installed:
```bash
sudo pacman -S --needed iproute2 ethtool procps-ng coreutils systemd

2. Clone the Repository & Compile the Package

Run the build script to generate the folder structures and files:

git clone https://github.com/technowasfound/Ethernet-Optimizer-Arch.git
cd Ethernet-Optimizer-Arch
bash build.sh

3. Install Globally to System Folders

Move the compiled application and default profiles to their proper system paths:

# Copy and enable executable binary
sudo cp ethoptimizer-project/usr/bin/ethoptimizer /usr/bin/ethoptimizer
sudo chmod +x /usr/bin/ethoptimizer

# Copy configuration files
sudo mkdir -p /etc/ethoptimizer
sudo cp ethoptimizer-project/etc/ethoptimizer/config.conf /etc/ethoptimizer/config.conf

4. Enable Background Telemetry Daemon (Optional)

To log network performance metrics over time:

sudo cp ethoptimizer-project/usr/lib/systemd/system/ethoptimizer-monitor.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now ethoptimizer-monitor.service

Command Reference

To utilize the application, run commands prefixed with sudo (since system-level
TCP flags require root access to modify):

1. Run Diagnostic Audit

Audit your active network interfaces, current MTU size, active queuing
disciplines, and DNS benchmark latencies:

sudo ethoptimizer diagnose

2. Execute Performance Benchmarking

Measure real-time latencies, network packet stability, and current
downstream/upstream bandwidth capabilities:

sudo ethoptimizer speed-test

3. Apply a Tuning Profile

Optimize your ethernet configurations by calling the optimizer with a target
preset:

# Apply Low-Latency Gaming settings
sudo ethoptimizer optimize --profile gaming

# Apply Ultra-High Speed throughput settings
sudo ethoptimizer optimize --profile max-perf

4. Run a Dry Run (Simulation)

Inspect what configurations would be changed by a profile without applying any
actual writes to your running kernel:

sudo ethoptimizer optimize --profile gaming --dry-run

Safety and Rollbacks

System stability is a key priority. Before making any modifications to
/proc/sys/net or call-outs to driver configurations, ethoptimizer creates an
automated snapshot of your previous states.

  - The backup is saved as a timestamped JSON file to
    /var/lib/ethoptimizer/backups/.
  - If you experience network instability or driver conflicts under an optimized
    profile, restore your system's original values by running:
    sudo ethoptimizer revert
    This reads your most recent backup snapshot, restores those settings, and
    deletes active persistent overrides from /etc/sysctl.d/99-ethoptimizer.conf.

Directory Structure

When deployed, the program configures the following file tree:

/
├── etc/
│   └── ethoptimizer/
│       └── config.conf                   # Global options and customizable profile definitions
├── usr/
│   ├── bin/
│   │   └── ethoptimizer                  # Core Python CLI program
│   └── lib/
│       └── systemd/
│           └── system/
│               └── ethoptimizer-monitor.service  # Optional background tracking service
├── var/
│   ├── lib/
│   │   └── ethoptimizer/
│   │       └── backups/                  # Safety backup storage location
│   └── log/
│       └── ethoptimizer/                 # Background performance telemetry log files


