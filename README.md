# openstack-installation

step 1: create a file in: /etc/netplan/01-netcfg.yml

The file: 02-netcfg.yml is more exhaustive, it should be used after the minimalist 01 configuration works: 

step 2: 
apply with:
sudo netplan apply

Step 3: Tell Kolla-Ansible to use the bridge

Edit /etc/kolla/globals.yml and add:

neutron_external_interface: "br-ex"

This ensures Neutron will attach floating IPs to the br-ex bridge.

Step 4: Create the external network in OpenStack

After deployment, define the provider external network:

openstack network create --share --external \
  --provider-physical-network external \
  --provider-network-type flat ext_net

  Define a subnet that matches our LAN’s/purchased IP range (but avoids conflicts):

openstack subnet create --network ext_net \
  --allocation-pool start=203.0.113.2,end=203.0.113.6 \
  --gateway 203.0.113.1 \
  --subnet-range 203.0.113.0/29 ext_subnet

  We should now you can assign floating IPs (from the pool in the subnet above) to tenant VMs, and they’ll be reachable on your LAN.
