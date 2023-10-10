# Roll out OPNsense on Proxmox Hosts 

This is an Ansible playbook to roll out OPNsense on Proxmox. It is a simplified version of the _KPMA1985_'s script to be found here: https://github.com/kpma1985/ansible_proxmox_hetzner/tree/master.

The playbook rolls out OPNsense with a minimum of two interfaces (WAN + x number of LAN-interfaces). You can simply adjust the interface configurations as described below. It will establish a rule to expose the WebGUI over the WAN interface, if that variable is set to true. It will enable Outbound NAT, if you want this to be enabled. You can define the root password to your liking.

### NOTE
The script will destroy a running VM based on the defined VM ID, if the variable _force_clean_ exists and is set to _true_.

## Usage

### 1. Install Ansible
This highly depends on your environment, so please google that for yourself.

### 2. Adjust the inventory
Fill in the required information. Note: This script works with SSH-keys, not with passwords. 

### 3. Adjust the variables in group_vars/all.yml

### 4. Run the playbook from the console:

```
ansible-playbook -i inventory.yml install_opnsense.yml
```

## List of Variables

<table>
    <thead>
        <tr>
            <th></th>
            <th>Variable</th>
            <th>Sample Value</th>
            <th>Remarks</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td style="vertical-align:top"><strong>Globals</strong></td>
            <td>_install_directory</td>
            <td><code>'/tmp/opnsense'</code></td>
            <td>Directory in which the OPNsense image is downloaded to, and where the XML configuration is adjusted within.</td>
        </tr>
        <tr>
            <td rowspan="4" style="vertical-align:top"><strong>VM</strong></td>
            <td>vm_id</td>
            <td><code>2000</code></td>
            <td>ID of the VM to be created.</td>
        </tr>
        <tr>
            <td>vm_name</td>
            <td><code>'opnsense-vm'</code></td>
            <td>ID of the VM to be created.</td>
        </tr>
        <tr>
            <td>storage_type</td>
            <td><code>'local_lvm'</code></td>
            <td>torage type to be used. Depending on your Proxmoxes config. Could also be like <code>'local_zfs'</code> or any other useful value.</td>
        </tr>
        <tr>
            <td>disk_size</td>
            <td><code>'10G'</code></td>
            <td>System Disk size. <code>'10G'</code> should be plenty enough for a small homelab.</td>
        </tr>
        <tr>
            <td rowspan="4" style="vertical-align:top"><strong>WAN</strong></td>
            <td>external_ip</td>
            <td><code>'192.168.1.85'</code></td>
            <td>External IP of the VM. Alternatively, <code>'dhcp'</code> should work as well.</td>
        </tr>
        <tr>
            <td>mtu</td>
            <td><code>1</code></td>
            <td>MTU of the network. Set it <code>1</code> to have it copied from the underlying Network Interface.</td>
        </tr>
        <tr>
            <td>wan_interface</td>
            <td><code>'vtnet0'</code></td>
            <td>Name of the interface to be used for WAN. Typically, it will be <code>'vtnet0'</code>, referring to the first virtual network adapter attached to the VM, but you can align it to your needs.</td>
        </tr>
        <tr>
            <td>gateway</td>
            <td><code>'192.168.1.1'</code></td>
            <td>IP of the gateway to be used. Typically, it will be the router within your environment. Leave it empty (<code>''</code>) to define no gateway.</td>
        </tr>
        <tr>
            <td>subnet</td>
            <td><code>'24'</code></td>
            <td>The subnet to be used. Typically, it will be something like <code>'24'</code>.</td>
        </tr>
        <tr>
            <td style="vertical-align:top"><strong>LANs</strong></td>
            <td>lan_networks</td>
            <td><code>- { id: 'lan', description: 'LAN', ip: '10.0.0.1', mtu: '1500', bridge: 'vmbr1', interface: 'vtnet1', subnet: '24' }</code></td>
            <td>List of LANs to be created. Each LAN is defined by these parameters:<ul><li>id: Id of the LAN in OPNsense</li>
                <li>description: Description of the LAN in OPNsense</li><li>ip: IP within the LAN</li>
                <li>subnet: Subnet to be used</li><li>bridge: Bridge-Device to be used on Proxmox</li><li>interface: Interface to be used for the LAN on OPNsense</li></ul></td>
        </tr>
        <tr>
            <td rowspan="3" style="vertical-align:top"><strong>OPNsense</strong></td>
            <td>opnsense_image_url</td>
            <td><code>"https://mirror.ams1.nl.leaseweb.net/opnsense/releases/23.7/OPNsense-23.7-nano-amd64.img.bz2"</code></td>
            <td>Download-Link for the OPNsense-image to roll out. Use the nano-variant.</td>
        </tr>
        <tr>
            <td>opnsense_image_path</td>
            <td><code>"{{ _install_directory }}/OPNsense.img.bz2"</code></td>
            <td>Where to store the downloaded image.</td>
        </tr>
        <tr>
            <td>opnsense_config_url</td>
            <td><code>"https://raw.githubusercontent.com/opnsense/core/master/src/etc/config.xml.sample"</code></td>
            <td>URL of the sample XML-configuration file to be used as ... sample.</td>
        </tr>
        <tr>
            <td rowspan="4" style="vertical-align:top"><strong>Additional Settings</strong></td>
            <td>enable_webgui_via_wan</td>
            <td><code>false</code></td>
            <td>Indicates whether the WebGUI should be exposed via the WAN interface. Useful values are <code>true</code> or <code>false</code>. Note: Exposing the WebGUI via WAN implies a huge risk when used outside a secure environment!</td>
        </tr>
        <tr>
            <td>enable_outbound_nat</td>
            <td><code>false</code></td>
            <td>Indicates whether Outbound NAT should be enabled. Often required in Homelab-scenarios. Useful values are <code>true</code> or <code>false</code>.</td>
        </tr>
        <tr>
            <td>webgui_port</td>
            <td><code>8443</code></td>
            <td>Port of the WebGUI, regardless of being exposed externally or internally only. Recommendation: Don't leave it on 443, since then every SSL-traffic would need to be mapped to a different port.</td>
        </tr>
        <tr>
            <td>admin_password</td>
            <td><code>'ChangeMe123!'</code></td>
            <td>Password for the root user. Adjust it to your own needs, but adjust it.</td>
        </tr>
    </tbody>
</table>