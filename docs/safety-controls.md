# Safety Controls

## Network Security Group configuration

### Inbound rules

| Priority | Name | Source | Destination | Port | Protocol | Action |
|----------|------|--------|-------------|------|----------|--------|
| 1000 | `Allow-RDP-Inbound` | Any | Any | 3389 | TCP | Allow |
| 65000 | `AllowVnetInBound` | VirtualNetwork | VirtualNetwork | Any | Any | Allow |
| 65001 | `AllowAzureLoadBalancerInBound` | AzureLoadBalancer | Any | Any | Any | Allow |
| 65500 | `DenyAllInBound` | Any | Any | Any | Any | Deny |

### Outbound rules

| Priority | Name | Destination (Service Tag) | Action |
|----------|------|---------------------------|--------|
| 100 | `Allow-AzureMonitor-Out` | AzureMonitor | Allow |
| 110 | `Allow-AAD-Out` | AzureActiveDirectory | Allow |
| 120 | `Allow-Storage-Out` | Storage | Allow |
| 130 | `Allow-GuestHybrid-Out` | GuestAndHybridManagement | Allow |
| **4000** | **`Deny-Internet-Out`** | **Internet** | **Deny** |
| 65000 | `AllowVnetOutBound` (default) | VirtualNetwork | Allow |
| 65001 | `AllowInternetOutBound` (default) | Internet | Allow (unreachable) |
| 65500 | `DenyAllOutBound` (default) | Any | Deny |

**Rule priority reasoning:** Azure evaluates NSG rules lowest-priority-number first. Priority 4000 (`Deny-Internet-Out`) is evaluated before the Azure default priority 65001 (`AllowInternetOutBound`), so outbound internet traffic is denied unless it matches one of the specific Azure service allows above priority 4000. This is defense-in-depth layered on top of the defaults.

## Verification

The egress control was validated end-to-end from inside the VM:

```powershell
# Should fail
Test-NetConnection google.com -Port 443
Test-NetConnection pastebin.com -Port 443

#should pass
Test-NetConnection global.handler.control.monitor.azure.com -Port 443
```

Results confirmed the deny rule is functioning.

## VM-level controls

- **System-assigned managed identity:** Explicitly disabled during VM creation.
- **User-assigned managed identity:** None attached.
- **Admin password:** 20+ character random string.
- **Patch orchestration:** Manual.
- **Boot diagnostics:** Enabled for investigation purposes if something goes wrong.


## Budget and cost monitoring


| Budget threshold | Alert action | Investigation trigger |
|------------------|-------------|----------------------|
| 50% of $15 | Email notification | Check VM CPU, running processes, outbound traffic attempts |
| 80% of $15 | Email notification | If not already stopped, stop VM and investigate |
| 100% of $15 | Email notification | VM should already be stopped 