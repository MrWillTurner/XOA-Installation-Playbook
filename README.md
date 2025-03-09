# XCP-ng XOA Installation Playbook

Ansible playbook for automated installation of Xen Orchestra Appliance (XOA) on XCP-ng servers.

## Requirements

- **Control Node:**
  - Ansible 2.9+
  - Python 3.x

- **XCP-ng Host:**
  - SSH enabled
  - 2 vCPUs available
  - 4 GB RAM available
  - 10 GB disk space free
  - Network connectivity

## Quick Start

1. Clone and enter repository:
```bash
git clone <repository-url>
cd <repository-directory>
```

2. Create inventory file:
```ini
[xcp-ng]
xcpng-host ansible_host=192.168.1.100 ansible_user=root
```

3. Run playbook:
```bash
ansible-playbook playbook.yml
```

## Installation Process

The playbook performs these steps to configure XOA:

1. **Network Setup**
   - Collects network configuration (interactive or from inventory)
   - Validates network settings
   - Selects and configures network interface

2. **Storage Preparation**
   - Lists available storage repositories
   - Selects repository with sufficient space (10GB+)
   - Prepares for VM installation

3. **VM Creation**
   - Downloads XOA appliance image
   - Creates VM with required resources
   - Configures network settings
   - Sets up autostart preference

4. **VM Configuration**
   - Configures boot parameters
   - Creates and configures network interface
   - Sets static IP configuration
   - Starts the VM

5. **Post Setup**
   - Verifies VM is running
   - Retrieves VM IP address
   - Updates inventory with VM details
   - Displays access information

6. **XCP-ng Integration**
   - Registers XOA instance with default credentials
   - Automatically adds XCP-ng host to XOA
   - Establishes connection between XOA and XCP-ng

## Configuration Options

### Network Settings

Configure network in one of these ways:

1. **Interactive (Default)**: Playbook will prompt for:
   - IP address
   - Netmask
   - Gateway
   - DNS (optional)

2. **Inventory File**:
```ini
[all:vars]
xoa_ipaddress=192.168.1.3
xoa_netmask=255.255.255.0
xoa_gateway=192.168.1.1
xoa_dns=8.8.8.8  # Optional, defaults to gateway
```

3. **Command Line**:
```bash
ansible-playbook playbook.yml \
  -e "xoa_ipaddress=192.168.1.3" \
  -e "xoa_netmask=255.255.255.0" \
  -e "xoa_gateway=192.168.1.1"
```

### Network Interface

Default: `eth0`

Change interface in inventory:
```ini
[all:vars]
xoa_network_interface=eth1
```

Or via command line:
```bash
ansible-playbook playbook.yml -e "xoa_network_interface=eth1"
```

If specified interface isn't found, playbook will:
- List available interfaces
- Allow manual selection

### VM Autostart

Control whether XOA starts automatically with XCP-ng host.

Default: Enabled

Disable in inventory:
```ini
[all:vars]
xoa_autostart=false
```

Or via command line:
```bash
ansible-playbook playbook.yml -e "xoa_autostart=false"
```

## Post-Installation

After successful installation:

1. Access XOA:
   - Web UI: `https://<xoa-ip>`
   - Default credentials: `admin@admin.net`/`admin`

2. Important:
   - Change web interface password on first login
   - Configure any additional XOA settings as needed

## Troubleshooting

1. **Network Issues**
   - Verify network settings
   - Check interface exists and is up
   - Ensure connectivity to XCP-ng host

2. **Storage Issues**
   - Verify 10 GB minimum space
   - Check storage repository access

3. **Access Issues**
   - Verify XOA IP is reachable
   - Check web UI is accessible
   - Confirm credentials are correct

4. **XCP-ng Connection Issues**
   - Verify XCP-ng host is reachable
   - Check root credentials or SSH key
   - Ensure no firewall blocks XOA to XCP-ng communication (ports 80, 443) 