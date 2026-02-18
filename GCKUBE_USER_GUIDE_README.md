# Gckube - Kubernetes Cluster Report Tool
## Gckube User Guide

---

## Table of Contents
1. [What is Gckube](#what-is-gckube)
2. [Current Features (v1.1.0)](#current-features-v101)
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

## Current Features (v1.1.0)

### Basic Cluster Information Collection
- **Kubernetes Distribution**: Detects cluster type (e.g., GKE, EKS, on-premises)
- **CNI Plugin**: Identifies installed CNI version and additional CNI specific information (for example, if Calico api server installed) 
- **Node Information**: Collects details about cluster nodes
- **Kubernetes Version**: Gathers API server version
- **Ram information on nodes**: Collects RAM information on cluster nodes

### Advanced Connectivity Checks (Optional)
- **L5 Connectivity Check**: Validates network layer connectivity to Guardicore aggregator if aggregator address is provided
   - Creates a temporary Kubernetes job that attempts to connect to the aggregator endpoint
   - Reports success or failure of connectivity
- **Orchestration to API Server Connectivity**: Verifies if Guardicore aggregator can reach the Kubernetes API server
  - Requires an aggregator endpoint to be deployed that accepts API server details and attempts to connect. The endpoint is
supported only from Centra v53.3 and above
  - The api server flag/input should be leaved empty to skip this check
  - Reports success or failure of connectivity

---

## Download

Download the latest gckube binary release:

**URL**: [gckube-linux-amd64-v1.1.0.tar.gz](https://github.com/slava-ustinov/gc-tools/blob/main/gckube-linux-amd64-v1.1.0.tar.gz)

---

## Installation

### Prerequisites
- Linux machine with amd64 architecture
- `kubectl` properly configured with access to the cluster and ability to create job in default namespace (for connectivity checks)

### Installation Steps

1. **Download the binary**
   ```bash
   wget https://github.com/slava-ustinov/gc-tools/blob/main/gckube-linux-amd64-v1.1.0.tar.gz
   ```

2. **Extract to /usr/bin**
   ```bash
   tar -xzvf gckube-linux-amd64-v1.1.0.tar.gz -C /usr/bin/
   ```

3. **Verify installation**
   ```bash
   gckube --help
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
  --cni-type calico \
  --aggregator-address 172.16.100.50 \
  --aggregator-port 443 \
  --api-server-ip 10.0.0.1 \
  --api-server-port 6443 \
  --ui-password "YourUIPassword"
```

### Command Line Flags Reference

| Flag | Description | Example | Required |
|------|-------------|---------|----------|
| `--cni-type` | Container Network Interface type | `calico`, `ovn`, `azurecni`, `amazonvpc`, `cilium` | Yes |
| `--aggregator-address` | Guardicore aggregator IP or FQDN | `172.16.100.50` or `aggr.example.com` | No* |
| `--aggregator-port` | Guardicore aggregator port | `443` (default) | No |
| `--api-server-ip` | Kubernetes API server IP | `10.0.0.1` | No* |
| `--api-server-port` | Kubernetes API server port | `6443` (default) | No |
| `--ui-password` | Guardicore UI password | `SecurePassword123` | No* |

**Note**: * Only required if performing advanced connectivity checks.

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
  --cni-type calico \
  --aggregator-address 172.16.100.50 \
  --aggregator-port 443 \
  --api-server-ip 10.0.0.1 \
  --api-server-port 6443 \
  --ui-password "MySecurePassword"
```

#### Example 3: Both commands line arguments and prompts (CNI via argument, prompts for aggregator)
```bash
gckube --cni-type ovn
```
Then when prompted:
```
Please input aggregator address ip: 172.16.100.50
Please input aggregator port (default 443): 443
```

#### Example 4: Skip Advanced Checks
```bash
gckube --cni-type cilium
```
(To skip advanced connectivity checks, leave aggregator and API server information empty when prompted)

---

## Advanced Connectivity Checks

### Overview
Advanced connectivity checks validate that cluster can connect to the aggregator (basic L5 connectivity), 
and that the Guardicore aggregator can reach the Kubernetes cluster and API server. 
To perform the check from the aggregator to api server, a special endpoint should be deployed on the aggregator. 
This feature is supported only from Centra v53.3 and above. 

### What Gets Checked

#### 1. L5 Connectivity Check
- **Purpose**: Validates network connectivity from the cluster to the aggregator
- **Method**: Attempts to reach a public aggregator endpoint
- **Results**:
  - ✅ Success: Cluster can reach aggregator
  - ❌ Failed: No network connectivity to aggregator/fairwall blocking access

#### 2. Orchestration to API Server Connectivity (supported from Centra v53.3)
- **Purpose**: Verifies if the aggregator can reach the Kubernetes API server
- **Method**: Aggregator attempts to connect to the API server using provided IP and port
- **Results**:
  - ✅ Success (200 OK): Aggregator can reach API server
  - ❌ Failed (401): Cannot authenticate to aggregator from gckube
  - ❌ Failed (500): Aggregator cannot reach API server


### Report Sections

- **cluster information report blocks**: Basic cluster configuration details
- **advanced_checks_report**: Results of connectivity validation tests
  - Status values: `"succeeded"` or `"failed"`
  - Reasoning: Explanation if a check failed (empty if successful)

---

## Version History

### v1.1.0 (Current)
- Cluster information collection
- L5 connectivity check to aggregator
- Orchestration to API server connectivity check (supported from Centra v53.3)
- Command line arguments support
- Interactive mode with user prompts

### v1.0.0 (Initial Release)
- Basic cluster information collection via kubectl commands
- Interactive mode only (no command line flags)
- No connectivity checks 

---
