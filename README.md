Here is a complete, professional, and humble README.md guide tailored
specifically for your Ethernet-Optimizer-Arch repository. You can copy and paste
this directly into your GitHub editor shown in your screenshot.

# Ethernet Optimizer for Arch Linux (`ethoptimizer`)

A modular command-line network diagnostic and optimization tool designed specifically for Arch-based Linux distributions (Arch Linux, CachyOS, EndeavourOS, Artix, etc.). It automates the benchmarking, diagnostics, and tuning of system network stacks and Ethernet drivers to help reduce packet loss, improve jitter, and optimize throughput.

---

## Key Features

*   **Network Diagnostics:** Audits MTU settings, ethtool driver properties (buffer rings, coalescing), TCP congestion control status, and queries localized DNS benchmarks.
*   **Performance Metrics:** Performs safe ping, jitter, and multi-threaded throughput tests using standard HTTP/HTTPS endpoints.
*   **Targeted Optimization Profiles:** Pre-configured optimization presets tailored for various workloads (Gaming, High-Throughput Streaming, General Use, Server, and Max Performance).
*   **System Integration:** Optional background performance-monitoring systemd daemon.
*   **Safety Safeguards:** Implements complete profile configuration rollbacks, validation tests, and a `dry-run` mechanism to inspect system alterations before applying them.

---

## Performance Tuning Profiles

Each profile modifies kernel system configurations (`sysctl`) and hardware adapter structures (`ethtool`):

| Profile Name | Target Optimization | Primary Adjustments |
| :--- | :--- | :--- |
| `gaming` | Reduces latency and queue delays | Disables unnecessary TCP timestamps, lowers coalescence timings, sets `fq` qdisc, enables BBR |
| `streaming` | Maximizes bulk-buffering and flow consistency | Expands buffer capacities to mitigate throughput decay, implements BBR with `fq_codel` |
| `server` | High-concurrency network environments | Optimizes network backlogs and increases system-wide socket memory limits |
| `max-perf` | Maximum possible throughput and ring buffers | Fully scales adapter ring sizes (rx/tx 4096), locks offload technologies (TSO/GSO/GRO) |
| `general` | Balanced use-case with baseline optimizations | Safe default configurations using standard Cubic congestion engines |

---

## Dependencies

The tool relies on standard Linux utilities. On Arch-based systems, verify these are installed:

```bash
sudo pacman -S --needed iproute2 ethtool procps-ng coreutils systemd

Note: Python 3 is installed by default on almost all Arch distributions and is
required to run the core executable.

Installation and Setup

1. Download the Executable and Configuration

Place the script in /usr/bin (or /usr/local/bin) and copy the default profile
configuration file to /etc/ethoptimizer/:

# Clone this repository (or copy the files manually)
git clone https://github.com/technowasfound/Ethernet-Optimizer-Arch.git
cd Ethernet-Optimizer-Arch

# Create the required directories
sudo mkdir -p /etc/ethoptimizer /var/lib/ethoptimizer/backups /var/log/ethoptimizer

# Deploy the files
sudo cp ethoptimizer.py /usr/bin/ethoptimizer
sudo chmod +x /usr/bin/ethoptimizer
sudo cp config.conf /etc/ethoptimizer/config.conf

2. Configure the Background Daemon (Optional)

If you wish to continuously monitor network latency and jitter metrics in the
background:

sudo cp ethoptimizer-monitor.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now ethoptimizer-monitor.service

Command Reference

The utility uses a simple sub-command structure:

sudo ethoptimizer [COMMAND] [OPTIONS]

Basic Commands

  - Diagnostic Audit: Inspect the current hardware and kernel parameters of your
    active interface:
    sudo ethoptimizer diagnose
  - Safe Dry-Run: View what settings would be modified under a specific profile
    without applying them:
    sudo ethoptimizer optimize --profile gaming --dry-run
  - Apply Optimizations: Commit optimizations to the active interface:
    sudo ethoptimizer optimize --profile gaming
  - Performance Benchmarking: Test current ping, download speed, and upload
    speeds:
    sudo ethoptimizer speed-test
  - Generate an HTML Report: Compile diagnostics and performance benchmarks into
    an interactive HTML dashboard:
    sudo ethoptimizer report --output ~/my_network_report.html
  - Revert System Settings: Instantly restore kernel configurations and adapter
    settings to their original state prior to optimization:
    sudo ethoptimizer revert

Configuration Customization

You can define your own testing nodes and tweak specific kernel profile
properties by editing the configuration file:

sudo nano /etc/ethoptimizer/config.conf

The system will automatically parse and apply the rules declared inside this
configuration during profile switches.

Safety & Rollbacks

Before modifying any kernel sysctl keys or calling ethtool to alter hardware
features, ethoptimizer serializes and saves a timestamped JSON backup snapshot
of your exact system settings to /var/lib/ethoptimizer/backups/.

Running the revert command reads the most recent snapshot file, applies the
values back to the running system, and purges the persistent systemd/sysctl
overrides—returning your network stack to its exact prior state.

