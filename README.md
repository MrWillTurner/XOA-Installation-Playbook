# XCP-ng XOA Installation Playbook

This Ansible playbook automates the installation and configuration of Xen Orchestra Appliance (XOA) on XCP-ng servers.

## Requirements

### Control Node
- Ansible 2.9 or higher
- Python 3.x

### XCP-ng Host
- XCP-ng server with SSH enabled
- Minimum resources:
  - 2 vCPUs available
  - 4 GB RAM available
  - 10 GB disk space free
- Network connectivity

## Pre-Installation Steps

Before running the playbook, you need to manually accept the SSH fingerprints:

1. For XCP-ng host:
```bash
ssh root@<xcpng-host-ip>
# Accept the fingerprint when prompted
exit
```

2. After XOA is installed, for XOA host:
```bash
ssh xo@<xoa-ip-address>
# Accept the fingerprint when prompted
exit
```

These steps are security measures that help verify the authenticity of the remote hosts.

## Playbook Structure

The playbook consists of several roles that handle different aspects of the installation:

### 1. add-sshkey
Handles SSH key setup between the control node and XCP-ng host.
- Adds SSH public key to authorized_keys
- Tests SSH key-based authentication

### 2. install-xoa
Manages the XOA VM installation on XCP-ng.
- Network configuration (IP, netmask, gateway, DNS)
- Network interface selection (automatic or manual)
- Storage repository selection
- XOA image upload and VM creation
- VM configuration (2 vCPUs, 4 GB RAM)
- Initial VM startup

### 3. prepare-xoa
Handles post-installation configuration of XOA.
- Waits for XOA to be fully started
- Displays post-installation information

## Usage

1. Clone this repository:
```bash
git clone <repository-url>
cd <repository-directory>
```

2. Create your inventory file:
```ini
# Example inventory file structure
[xcp-ng]
xcpng-host ansible_host=192.168.1.100 ansible_user=root

[all:vars]
# Optional: Network configuration (if not provided, will be prompted)
xoa_ipaddress=192.168.1.3
xoa_netmask=255.255.255.0
xoa_gateway=192.168.1.1
xoa_dns=8.8.8.8
```

3. Configure network settings using one of these methods:
   - Add variables to inventory file (as shown above)
   - Use command line parameters
   - Let the playbook prompt for values interactively (default)

4. Run the playbook:
```bash
ansible-playbook playbook.yml
```

## Configuration

### SSH Fingerprint Handling

The playbook requires SSH fingerprint acceptance at two stages:

1. **Pre-Installation (Required)**
   Before running the playbook, manually accept the XCP-ng host's SSH fingerprint:
   ```bash
   ssh root@<xcpng-host-ip>
   # Accept the fingerprint when prompted
   exit
   ```

2. **Post-VM Creation (Required)**
   After XOA VM is created but before configuration:
   ```bash
   ssh root@<xoa-ip-address>
   # Accept the fingerprint when prompted
   exit
   ```

These steps are security measures that help verify the authenticity of the remote hosts and cannot be automated for security reasons.

### Network Configuration

The playbook requires network configuration for XOA. You can provide this in several ways:

1. **Interactive Setup (Default)**
If no network configuration is provided, the playbook will interactively prompt for:
- IP address for XOA
- Netmask
- Gateway
- DNS server (optional, will use gateway if not specified)

The playbook will display the configuration for review before proceeding.

2. **Inventory File (Recommended)**
```ini
[all:vars]
xoa_ipaddress=192.168.1.3
xoa_netmask=255.255.255.0
xoa_gateway=192.168.1.1
xoa_dns=8.8.8.8
```

3. **Playbook Variables**
```yaml
- hosts: xcp-ng
  vars:
    xoa_ipaddress: "192.168.1.3"
    xoa_netmask: "255.255.255.0"
    xoa_gateway: "192.168.1.1"
    xoa_dns: "8.8.8.8"
  roles:
    - role: install-xoa
```

4. **Command Line**
```bash
ansible-playbook playbook.yml \
  -e "xoa_ipaddress=192.168.1.3" \
  -e "xoa_netmask=255.255.255.0" \
  -e "xoa_gateway=192.168.1.1" \
  -e "xoa_dns=8.8.8.8"
```

### Network Interface Selection

The playbook provides flexible network interface configuration:

1. **Default Behavior**
   - Automatically looks for 'eth0'
   - No configuration needed if using eth0

2. **Custom Interface**
   You can specify a different interface in several ways:

   a. In inventory file (recommended):
   ```ini
   [all:vars]
   xoa_network_interface=eth1
   ```

   b. In playbook:
   ```yaml
   - hosts: xcp-ng
     vars:
       xoa_network_interface: "eth1"
     roles:
       - role: install-xoa
   ```

   c. Command line:
   ```bash
   ansible-playbook playbook.yml -e "xoa_network_interface=eth1"
   ```

3. **Automatic Selection**
   - Searches for specified interface name in XCP-ng network list
   - Case-sensitive match
   - Falls back to interactive selection if not found

4. **Interactive Fallback**
   If the specified interface is not found, the playbook will:
   - List all available network interfaces
   - Show interface details (UUID and name)
   - Allow manual selection
   - Validate the selection

5. **Important Notes**
   - Interface name must match exactly
   - Case-sensitive matching
   - Interactive fallback ensures you can always select the correct interface
   - Selection is validated before proceeding

## Installation Process

1. Network Configuration:
   - Collect or prompt for network settings
   - Validate configuration

2. Network Interface Selection:
   - Automatic detection of specified interface
   - Interactive selection if automatic fails

3. Storage Selection:
   - List available storage repositories
   - Interactive selection of SR
   - Minimum 10 GB space required

4. XOA Installation:
   - Upload XOA image
   - Create VM with fixed resources:
     - 2 vCPUs
     - 4 GB RAM
     - 10 GB minimum disk space
   - Configure network settings
   - Initial startup

## Network Requirements

- The selected network interface must:
  - Exist on the XCP-ng host
  - Be properly configured and up
  - Have access to:
    - XCP-ng management interface
    - Internet (for XOA updates)
    - Your management network (for accessing XOA web interface)
- Static IP configuration required
- DNS configuration recommended (defaults to gateway if not specified)

## Post-Installation

After successful installation:
1. XOA will be accessible via its configured IP address
2. Default credentials will be displayed
3. You can access the web interface to complete setup

## Notes

- VM resources are fixed according to XOA requirements
- Network configuration is required but can be provided interactively
- Network interface selection supports both automatic and manual modes
- All configuration can be automated through inventory/playbook variables

## Troubleshooting

If you encounter issues:
1. Ensure all requirements are met
2. Check network connectivity
3. Verify network configuration values
4. Verify SSH fingerprint acceptance:
   - Check if you can SSH to XCP-ng host
   - Check if you can SSH to XOA after creation
5. Network interface issues:
   - Verify interface name exists
   - Check interface is up and configured
   - Try interactive selection if automatic fails
6. Verify storage repository has enough free space
7. Check XOA is accessible after installation

## Contributing

Feel free to submit issues and pull requests to improve the playbook. 