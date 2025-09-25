# OpenStack Installation

## Step 1: Create a Netplan Configuration File

Create a file at the path below and copy and paste the content of from the file in this repository:

```
/etc/netplan/01-netcfg.yml
```

> Note: The file **`02-netcfg.yml`** should be ignored for now.

---

## Step 2: Apply the Configuration

Run:

```bash
sudo netplan apply
```

---

## Step 3: Tell Kolla-Ansible to Use the Bridge

Edit:

```
/etc/kolla/globals.yml
```

Add the following line:

```yaml
neutron_external_interface: "br-ex"
```

This ensures Neutron will attach floating IPs to the **`br-ex`** bridge.

Edit:

```
/etc/neutron/plugins/ml2/openvswitch_agent.ini
```
or 
```
/etc/neutron/plugins/ml2/ml2_conf.ini
```
In the [ovs] section you add following line:

```yaml
[ovs]
bridge_mappings = external:br-ex
```
---

## Step 4: Create the External Network in OpenStack

After deployment, define the provider external network:

```bash
openstack network create --share --external   --provider-physical-network external   --provider-network-type flat ext_net
```

## Example ISP-Provided Block

Our ISP/datacenter provides a public subnet. We use this block to create the subnet for internet accessibility:

- **Public Subnet:** `203.0.113.32/27`  
- **Gateway:** `203.0.113.33`  
- **Usable IPs:** `203.0.113.34 – 203.0.113.62`
  
### Define a Subnet That Matches the LAN/ISP Block

Avoid overlapping with other networks:

```bash
openstack subnet create --network ext_net   --allocation-pool start=203.0.113.34,end=203.0.113.62   --gateway 203.0.113.33   --subnet-range 203.0.113.32/27 ext_subnet
```

At this point, we can assign floating IPs (from the pool defined above) to tenant VMs, and they should be reachable from the internet as long as the appropriate ports for connection are open.

---

### Restart neutron agents

```bash
sudo systemctl restart neutron-openvswitch-agent neutron-dhcp-agent neutron-l3-agent
```

### verify bridge mapping

```bash
sudo ovs-vsctl show
```
