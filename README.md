Deploy OPNsense on Proxmox
==========================

This role downloads and deploys a OPNsense Nano Image on Proxmox. It is loosely based on the ansible_proxmox_hetzner-role by KPMA1985 (https://github.com/kpma1985/ansible_proxmox_hetzner/tree/master). It allows to configure some base specifics so one ends up with a useful deployment.

Requirements
------------

None specifically. Ansible. SSH. Proxmox Host. 

Role Variables
--------------

### Globals
---
Global variables for the Role.

- `install_directory`: Installation directory on Proxmox', defaults to `'/tmp/opnsense'`
- `hostname` (optional): Hostname for the OPNsense installation, defaults to `'OPNsense'`. 
- `domain` (optional): Domain portion of the OPNsense installation, defaults to `'localdomain.tld'`.
- `timezone` (optional): Timezone for the installation, ie. `'Europe/Berlin'`.
- `admin_password`: Password for the root account, defaults to `'ChangeMe123!'`.
- `force_clean` (optional): Deletes an existing VM with the `vm_id` defined below. Acceptable values: `true` or `false`. Understood to be `false` when not explicitely set.

### VM
---
VM-specific settings.

- `vm_id`: Id of the VM to be created, ie. `2000`. **NOTE**: Role exection will fail when the VM already exists, see `force_clean` above.
- `vm_storage_type` (optional): Type of the storage to be used. Check your Proxmox for details. Typical values are `'local-lvm'` or `'local-zfs'`. Defaults to `'local-lvm'`.
- `vm_name` (optional): Name of the VM in Proxmox. Give it a name without spaces or underscores, defaults to `'opensense-vm'`.
- `vm_disk_size` (optional): Size of the main disk drive of the VM. Defaults to `'10G'`.
- `vm_memory` (optional): Memory allocation of the VM in MB, defaults to `2048`.
- `vm_sockets` (optional): Number of CPU-sockets to assign to the VM. Defaults to `1`.
- `vm_cores` (optional): Number of CPU-cores to assign to the VM. Defaults to `1`.
- `vm_cpu_type` (optional): Type of CPU to assign. Defaults to `'x86-64-v2-AES'`, but can for example also be `'host'`.
- `vm_start_on_boot` (optional): Indicates, whether the VM should be started automatically when the system boots. Defaults to `true`.

### Gateways
---
Definition of Gateways. Optional.

- `gateways`: List of gateways to be used.

Each entry consists of the following parameters:
- `name`: Name of the gateway, ie. `'WAN_GW'`.
- `gateway`: IP-address of the gateway, ie. `'192.168.1.1'`.
- `interface`: Interface to be used, ie. `'wan'`.
- `priority` (optional): Priority of the gateway, defaults to `255`.
- `default` (optional): Indicates whether the gateway is the default gateway, possible values are `true` and `false`, defaults to `true`.
- `description` (optional): Descriptive text for the gateway.
- `ipv6` (optional): Indicates whether the gateway is an IPv6-gateway, possible values are `true` and `false`, defaults to `false`.
- `monitor` (otional): Indicates whether the gateway-IP should be monitored for availability, possible values are `true` and `false`, defaults to `false`.
- `interval` (optional): Monitoring interval. Defaults to `1`.
- `weight` (optional): Weight of the gateway. Defaults to `1`.

Sample usage:
```
gateways:
  - { name: 'WAN_GW', gateway: '192.168.1.1', interface: 'wan', default: true, description: 'WAN Gateway' }
```

### DNS Servers
---
Defines a list of DNS-servers to be used. Optional.

- `dns_servers` (optional): List of DNS-servers, one per line.

Sample usage:
```
dns_servers:
  - '8.8.4.4'
  - '9.9.9.9'
```

### WebGUI access
---
Defines settings for accessing the WebGUI. Triggers the creation of firewall rules for additional interfaces on which the WebGUI should be made available. Per OPNsense default, the WebGUI is exposed at the `LAN`-interface, once a `WAN`-interface is defined. **Adjust and handle with care!**

- `webgui_port`: Sets the port on which the WebGUI is exposed. Default value is `8443`.
- `additional_webgui_interfaces` (optional): Defines a list of additional interfaces on which the WebGUI is exposed. No need to include the `lan`-interface here.

Sample usage:
```
additional_webgui_interfaces:
  - 'wan'
  - 'net' 
```

### WAN
---
Defines the WAN settings.

- `wan_ip`: IP of the WAN interface, ie. `'192.168.1.85'`.
- `wan_interface` (optional): Name of the WAN interface within OPNsense, defaults to `'vtnet0'`.
- `wan_gateway` (optional): Name of the Gateway towards the external network, defaults to `'WAN_GW'`, **should match the gateway name as defined above**.
- `wan_subnet` (optional): Subnet for the WAN, defaults to `'24'`.
- `wan_mtu` (optional): MTU for the WAN net, defaults to `1` for the Proxmox host (which copies the settings from the virtual network bridge assigned), defaults to `1500` for OPNsense (which is the default for most networks).
- `wan_bridge` (optional): Name of the virtual network bridge to be used from the Proxmox host, defaults to `'vmbr0'` if not set explicitely.

### LANs
---
Defines the LAN settings.

- `lan_networks`: List of LAN networks to be exposed internally.

Each list entry consists of the following parameters:

- `id`: Id of the network, ie. `'lan'`.
- `ip`: IP-address of the network interface. ie. `'10.0.0.254'`.
- `bridge`: Proxmox Network Bridge to be used, ie. `'vmbr1'`. 
- `interface`: Interface-name within OPNsense, ie. `'vtnet1'`.
- `description` (optional): Descriptive text, ie `'LAN'`.
- `mtu` (optional): MTU for the network, defaults to `'1500'`.
- `subnet` (optional): Subnet to be used, defaults to `'24'`.

Sample usage:

```
lan_networks:
  - { id: 'lan', ip: '10.0.0.252', mtu: '1450', bridge: 'kube01', interface: 'vtnet1', subnet: '24' }
  - { id: 'net', ip: '10.10.10.254', bridge: 'vmbr1', interface: 'vtnet2' }
```

### OPNsense-Settings
---
Some OPNsense-specific settings.

- `enable_outbound_nat` (optional): Indicates, whether outbound NAT should be enabled, defaults to `false`.
- `block_private_networks_on_wan` (optional): Indicates, whether Private Networks (ie. 192.168.0.0/16) should be allowed on the WAN interface. Defaults to `false`.
- `block_bogons_on_wan` (optional): Indicates, whether Bogon Networks (impossible networks) should be allowed on the WAN interface. Defaults to `false`.

### Misc
---
Defines miscellanous settings, ie. the image URL to be used.

- `opnsense_image_url` (optional): The URL of the OPNsense Nano Image to be downloaded, defaults to `"https://mirror.ams1.nl.leaseweb.net/opnsense/releases/23.7/OPNsense-23.7-nano-amd64.img.bz2"`.
- `opnsense_image_path` (optional): The local file name for the downloaded image, defaults to `"{{ install_directory }}/OPNsense.img.bz2"`.
- `opnsense_config_url` (optional): The URL for the sample XML file to be used by the Role for configuring OPNsense. Defaults to `"https://raw.githubusercontent.com/opnsense/core/master/src/etc/config.xml.sample"`.


Example Playbook
----------------

First: Clone the repo into a folder within the `roles`-subfolder:

```
mkdir -P opentaq_opnsense/roles
cd opentaq_opnsense/roles
git clone https://github.com/opentaq/opnsense_proxmox.git
cd ..
```

Create an inventory file called `inventory.yml` pointing to your Proxmox host in the main directory:

```
all:
  hosts:
    proxmox:
      ansible_host: 192.168.1.10
      ansible_user: root
```

Then create a file called `playbook.yml` in the main folder, with something like this in it:

```
---
- hosts: proxmox
  roles:
    - opnsense_proxmox
  vars:
    # Globals
    hostname: 'bastion'
    domain: 'opentaq.local'
    enable_outbound_nat: true
    admin_password: 'YouWillNeverGuessThis!'

    # VM
    vm_id: 2000
    vm_storage_type: 'local-zfs'
    vm_name: 'opentaq-vm'
    vm_cpu_type: 'host'

    # Gateways
    gateways:
      - { name: 'WAN_GW', gateway: '192.168.1.1', interface: 'wan', default: true, description: 'WAN Gateway' }
    
    # DNS-Servers
    dns_servers:
      - '8.8.4.4'
      - '9.9.9.9'

    # WAN
    wan_ip: '192.168.1.85'
    wan_interface: 'vtnet0'
    wan_gateway: 'WAN_GW'
    wan_bridge: 'vmbr0'

    # LANs
    lan_networks:
      - { id: 'lan', description: 'LAN', ip: '10.0.0.252', mtu: '1450', bridge: 'kube01', interface: 'vtnet1', subnet: '24' }
    
    # WebGUI
    webgui_port: 8443
    additional_webgui_networks:
      - "wan"

```

Adjust it to your needs and save it.

You should now have this structure:

```
opentaq_opnsense/
└── playbook.yml
└── inventory.yml
└── roles/
      └── opnsense_proxmox/
```

Now you can execute the playbook from the main folder:

```
ansible-playbook playbook.yml -i inventory.yml
```

**Enjoy!**

License
-------

Apache 2.0


Author Information
------------------

Karsten Samaschke<br />
opentaq.tv@gmail.com<br />
https://opentaq.tv