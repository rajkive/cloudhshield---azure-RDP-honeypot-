# Architecture & Deployment

## Resource inventory

All resources live in the `cloudshield-lab` resource group, single Azure region.

| Resource type | Resource name | Key configuration |
|---------------|---------------|-------------------|
| Resource Group | `cloudshield-lab` | Hosts all project resources |
| Virtual Network | `cloudshield-vnet` | `10.0.0.0/16`, no peering |
| Subnet | `cloudshield-snet` | `10.0.1.0/24` |
| Network Security Group | `cloudshield-nsg` | See [safety-controls.md](safety-controls.md) |
| Public IP | `ip`| Basic SKU, static assignment |
| Virtual Machine | `web-srv-02` | Windows Server 2022 Datacenter Azure Edition, B-series |
| Log Analytics Workspace | `law-cloudshield` | Pay-as-you-go pricing |
| Data Collection Rule | `dcr-cloudshield` | Security event log forwarding |
| Microsoft Sentinel | Enabled on `law-cloudshield` | 31-day free trial |

### Why disable managed identity?

If an attacker compromises the VM, a managed identity gives them a token to query Azure Resource Manager and potentially enumerate or modify other resources in the subscription. Explicitly disabling both system- and user-assigned identities removes that attack path.

## Deployment steps

The project was deployed through the Azure portal.

### 1. Resource group

Created `cloudshield-lab` in the chosen region. All subsequent resources scoped here.

### 2. Budget alert

Configured before any compute resources existed:
- Alerts at 50%, 80%, 100%


### 3. Virtual network and NSG

- VNet `vnet-honeypot` with `10.0.0.0/16` address space
- Subnet `snet-honeypot` at `10.0.1.0/24`
- No peering, no shared services
- NSG `cloudshield-nsg` created with rules per [safety-controls.md](safety-controls.md)

### 4. Log Analytics workspace

- `law-honeypot`
- Microsoft Sentinel enabled on the workspace

### 5. VM deployment

Key configuration choices:
- Size: B-series (allocation failed on first region attempt; resolved by trying a slightly larger size in a zone with available capacity)
- Image: Windows Server 2025 Datacenter Azure Edition Gen2 x64
- Networking: attached to `vnet-honeypot` / `snet-honeypot`, existing NSG `cloudshield-nsg`
- Public IP: created `pip-honeypot` as Basic SKU, static assignment
- Management → Identity: **system-assigned managed identity explicitly disabled**
- Patch orchestration: Manual


### 6. Data Collection Rule

Created via Sentinel → Data connectors → Windows Security Events via AMA:
- Name: `dcr-cloudhshield`
- Resources: `web-srv-02`
- Data source: Windows Security Events, preset = Common (includes 4624, 4625, 4648)
- Destination: `law-cloudshield`

The Azure Monitor Agent was installed automatically by the DCR.

### 7. Egress verification

From inside the VM, ran the `Test-NetConnection` tests to confirm the deny-all-outbound rule was working before trusting the environment.

### 8. Immediately logged out

Left the VM alone to let attackers find it.