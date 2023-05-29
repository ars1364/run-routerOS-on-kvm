# run-routerOS-on-kvm
run RouterOS on kvm with OpenVswitch virtual port
### Ubuntu 22.04.2 LTS (Jammy Jellyfish) installed on HP Gen9
instll KVM and other neccessaries tools:
```
apt install cpu-checker -y
kvm-ok
apt install qemu-kvm virt-manager virtinst libvirt-clients bridge-utils libvirt-daemon-system -y
wget https://download.mikrotik.com/routeros/6.48.7/chr-6.48.7.img.zip
unzip chr-6.48.7.img.zip
mv chr-6.48.7.img /var/lib/libvirt/images/chr-6.48.7.qcow2
```
# import qcow2 disk using virt-manager GUI
```
ssh -YC4 router-director
virt-manager
```
! set disk bus type to `virtio`
### sample network config for 2 port(1 NIC) using LACP bond 
```
network:
  bonds:
    bond0:
      interfaces:
      - eno49
      - eno50
      parameters:
        lacp-rate: slow
        mode: 802.3ad
        transmit-hash-policy: layer2+3
  ethernets:
    eno1:
      dhcp4: true
    eno2:
      dhcp4: true
    eno3:
      dhcp4: true
    eno4:
      dhcp4: true
    eno49: {}
    eno50: {}
  version: 2
  vlans:
    Vlan1710:
      id: 1710
      link: bond0
      dhcp4: false

    Vlan1709:
      addresses:
      - 172.20.128.60/24
      id: 1709
      link: bond0
      nameservers:
        addresses:
        - 1.1.1.1
      routes:
      - to: default
        via: 172.20.128.1
```
#### install openv vswitch
```
apt install openvswitch-switch openvswitch-common
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex Vlan1710
```
#### edit virsh file of vm and change following lines in it
```
    <interface type='bridge'>
      <mac address='xx:54:00:xx:13:xx'/>
      <source bridge='br-provider'/>
      <virtualport type='openvswitch'>
      </virtualport>
      <model type='virtio'/>
    </interface>
```
