# Gckube - Kubernetes Cluster Report Tool
## Gckube User Guide

---

## Table of Contents
1. [What is Gckube](#what-is-gckube)
2. [Current Features (v1.4.0)](#current-features-v140)
3. [Download](#download)
4. [Installation](#installation)
5. [How to Use](#how-to-use)
6. [Advanced Connectivity Checks](#advanced-connectivity-checks)

---

## What is Gckube

**Gckube** is a Guardicore Kubernetes cluster diagnostic tool designed to help support teams quickly gather 
comprehensive information about Kubernetes cluster configurations and verify connectivity to/from Guardicore aggregator.

### Purpose
- Automatic collection of cluster information in a structured JSON format
- Addition to the manual Site Survey questionnaires
- Verification of cluster prerequisites before deploying Guardicore solution

---

## Current Features (v1.4.0)

### Basic Cluster Information Collection
- **Kubernetes Distribution**: Detects cluster type (e.g., GKE, EKS, AKS, OCP, RKE, RKE2, k3s, on-premises), including EKS auto-mode detection
- **CNI Plugin**: Identifies the installed CNI type (auto-detected when possible) and version, plus CNI-specific information (e.g., Calico API server availability, Cilium `monitor-aggregation` log-aggregation config on Cilium 1.19+)
- **Node Information**: Collects details about cluster nodes, including any node taints
- **Kubernetes Version**: Gathers API server version
- **Ram information on nodes**: Collects RAM information on cluster nodes
- **Istio**: Detects whether Istio is installed, its version, mTLS mode, and sidecar-injected namespaces

### Advanced Connectivity & Pre-Install Checks (Optional)
- **L5 Connectivity Check**: Validates network layer connectivity to Guardicore aggregator if aggregator address is provided
   - Creates a temporary Kubernetes job that attempts to connect to the aggregator endpoint
   - Reports success or failure of connectivity
- **HostPath Admission Check**: Verifies the cluster's admission policy allows the agent's required hostPath mounts (`/proc`, `/lib/modules`, `/sys`), reporting `allowed`/`blocked`/`unknown`
- **Custom Namespace**: The pre-install check Job can run in a user-specified namespace via `--namespace`, instead of the `default` namespace
- **Orchestration to API Server Connectivity**: Verifies if Guardicore aggregator can reach the Kubernetes API server
  - **Note**: This feature requires a special aggregator endpoint that is currently not available - planned for future release
  - When the endpoint becomes available (expected in Centra v53.4+), it will accept API server details and attempt to connect
  - Currently, this check will be skipped with status "not_performed"

### Output
- **Colorized output**: Use `--color` to colorize the JSON printed to stdout (keys bold, strings yellow, booleans green/red, numbers cyan)

---

## Download

Download the latest gckube binary release for your architecture:

### Choose Your Architecture

**AMD64 (Intel/AMD processors):**
- Use this for most Linux systems with x86_64 processors
- **URL**: [gckube-linux-amd64-v1.4.0.tar.gz](https://raw.githubusercontent.com/slava-ustinov/gc-tools/main/gckube-linux-amd64-v1.4.0.tar.gz)

**ARM64 (ARM-based processors):**
- Use this for Linux systems running on ARM64 processors (for example, AWS Graviton, Raspberry Pi, or a Linux VM/container on Apple Silicon)
- **URL**: [gckube-linux-arm64-v1.4.0.tar.gz](https://raw.githubusercontent.com/slava-ustinov/gc-tools/main/gckube-linux-arm64-v1.4.0.tar.gz)

💡 **How to check your architecture:**
```bash
uname -m
# Output: x86_64 → Use AMD64 binary
# Output: aarch64 or arm64 → Use ARM64 binary
```

---

## Installation

### Prerequisites
- Linux machine (AMD64 or ARM64 architecture)
- `kubectl` properly configured with access to the cluster and ability to create job in default namespace (for connectivity checks)

### Installation Steps

1. **Download the binary for your architecture**

   **For AMD64 (x86_64):**
   ```bash
   wget https://raw.githubusercontent.com/slava-ustinov/gc-tools/main/gckube-linux-amd64-v1.4.0.tar.gz
   ```

   **For ARM64 (aarch64):**
   ```bash
   wget https://raw.githubusercontent.com/slava-ustinov/gc-tools/main/gckube-linux-arm64-v1.4.0.tar.gz
   ```

2. **Extract to /usr/bin**
   ```bash
   # For AMD64:
   tar -xzvf gckube-linux-amd64-v1.4.0.tar.gz -C /usr/bin/
   
   # For ARM64:
   tar -xzvf gckube-linux-arm64-v1.4.0.tar.gz -C /usr/bin/
   ```

3. **Verify installation**
   ```bash
   gckube --help
   gckube -v
   # Expected output: gckube version is v1.4.0
   ```

---

## How to Use

### Basic Usage (Interactive Mode)
Simply run gckube without any arguments. You will be prompted to enter required information:

```bash
gckube
```

### Using Command Line Arguments
You can provide all required information via command line arguments:

```bash
gckube \
  --cni calico \
  --agg-add 172.16.100.50 \
  --agg-port 443
```

### Command Line Flags Reference

| Flag | Short | Description | Example | Required |
|------|-------|-------------|---------|----------|
| `--cni` | `-c` | Container Network Interface type | `calico`, `ovn`, `azurecni`, `amazonvpc`, `cilium` | No* |
| `--agg-add` | `-a` | Guardicore aggregator IP or FQDN | `172.16.100.50` or `aggr.example.com` | No** |
| `--agg-port` | `-p` | Guardicore aggregator port | `443` (default) | No |
| `--namespace` | `-n` | Namespace for the pre-install check Job | `guardicore` (defaults to `default`) | No |
| `--color` | | Colorize the JSON output printed to stdout | | No |
| `--version` | `-v` | Print gckube version and exit | | No |

**Note**: * If omitted, gckube attempts to auto-detect the CNI, then falls back to an interactive prompt.
**Note**: ** Only required if performing advanced connectivity checks.

### Usage Examples

#### Example 1: Basic Cluster Report with Interactive Prompts
```bash
$ gckube
Generating cluster report...
Please input currently running CNI (calico, ovn, azurecni, amazonvpc, cilium): calico
Cluster Report:
{
  "cluster_information": {
    ...
  }
}
```

#### Example 2: Full Report with Connectivity Checks, using Command Line Arguments
```bash
gckube \
  --cni calico \
  --agg-add 172.16.100.50 \
  --agg-port 443
```

#### Example 3: Both commands line arguments and prompts (CNI via argument, prompts for aggregator)
```bash
gckube --cni ovn
```
Then when prompted:
```
Please input aggregator address ip: 172.16.100.50
Please input aggregator port (default 443): 443
```

#### Example 4: Skip Advanced Checks
```bash
gckube --cni cilium
```
(To skip advanced connectivity checks, leave aggregator information empty when prompted)

---

## Advanced Connectivity Checks

### Overview
Advanced connectivity checks validate that cluster can connect to the aggregator (basic L5 connectivity),
and that the Guardicore aggregator can reach the Kubernetes cluster and API server.

**Note**: The orchestration-to-API-server connectivity check requires a special aggregator endpoint that is
**currently not available - planned for future release** (expected in Centra v53.4+). Until then, this check
will be skipped with status "not_performed".

### What Gets Checked

#### 1. L5 Connectivity Check
- **Purpose**: Validates network connectivity from the cluster to the aggregator
- **Method**: Attempts to reach a public aggregator endpoint
- **Results**:
  - ✅ Success: Cluster can reach aggregator
  - ❌ Failed: No network connectivity to aggregator/firewall blocking access

#### 2. Orchestration to API Server Connectivity (Planned for Future Release)
- **Purpose**: Will verify if the aggregator can reach the Kubernetes API server
- **Status**: Currently not available - requires aggregator endpoint support (planned for Centra v53.4+)
- **Current Behavior**: Check is skipped with status "not_performed" and reasoning "Aggregator endpoint not available - planned for future release"
- **Future Method**: Aggregator will attempt to connect to the API server using provided IP and port
- **Expected Results** (when available):
  - ✅ Success (200 OK): Aggregator can reach API server
  - ❌ Failed (401): Cannot authenticate to aggregator from gckube
  - ❌ Failed (500): Aggregator cannot reach API server


### Report Sections

- **cluster information report blocks**: Basic cluster configuration details
- **advanced_checks_report**: Results of connectivity validation tests
  - Status values: `"connected successfully"` or `"failed"`
  - Reasoning: Explanation if a check failed (empty if successful)

---

## Version History

## v1.4.0 (Current)
- Added Istio check: detects installation, version, mTLS mode, and sidecar-injected namespaces
- Added CNI auto-detection (no longer requires `--cni` or an interactive prompt when detectable)
- Added node taints report and EKS auto-mode detection
- Added `--namespace` flag to run the pre-install check Job in a user-specified namespace
- Added hostPath admission check (`/proc`, `/lib/modules`, `/sys`) to the pre-install check Job
- Added Cilium `monitor-aggregation` log-aggregation config check (required `none` on Cilium 1.19+)
- Added `--color` flag to colorize JSON output
- Fixed OpenShift (OCP) distribution detection

### v1.3.1
- Removed unused flags

### v1.3.0
- Multi-architecture support: Separate binaries for AMD64 and ARM64
- Support for running cluster reports on ARM64 Kubernetes nodes

### v1.2.0
- Added -v flag for quick version check
- Bug fixes and improvements

### v1.1.1
- Bug fixes and improvements to advanced connectivity checks
- Improved error handling and reporting for failed checks

### v1.1.0
- Cluster information collection
- L5 connectivity check to aggregator
- Command line arguments support
- Interactive mode with user prompts

### v1.0.0 (Initial Release)
- Basic cluster information collection via kubectl commands
- Interactive mode only (no command line flags)
- No connectivity checks 

---
