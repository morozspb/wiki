## Полный конфиг «Офис с IoT и камерами»


 VLAN 10 — офис\
 VLAN 20 — гости\
 VLAN 30 — IoT\
 VLAN 40 — камеры\
 VLAN 99 — management\
ether1 — trunk, ether2-5 — access, ether6 — management


/interface bridge

add name=bridge1 vlan-filtering=no

/interface bridge port\
add bridge=bridge1 interface=ether1 pvid=1      # trunk\
add bridge=bridge1 interface=ether2 pvid=10     # office\
add bridge=bridge1 interface=ether3 pvid=20     # guest\
add bridge=bridge1 interface=ether4 pvid=30     # IoT\
add bridge=bridge1 interface=ether5 pvid=40     # cameras\
add bridge=bridge1 interface=ether6 pvid=99     # management\

/interface bridge vlan\
add bridge=bridge1 vlan-ids=10 tagged=ether1,bridge1 untagged=ether2\
add bridge=bridge1 vlan-ids=20 tagged=ether1,bridge1 untagged=ether3\
add bridge=bridge1 vlan-ids=30 tagged=ether1,bridge1 untagged=ether4\
add bridge=bridge1 vlan-ids=40 tagged=ether1,bridge1 untagged=ether5\
add bridge=bridge1 vlan-ids=99 tagged=ether1,bridge1 untagged=ether6\

/interface vlan\
add interface=bridge1 vlan-id=10 name=vlan10-office\
add interface=bridge1 vlan-id=20 name=vlan20-guest\
add interface=bridge1 vlan-id=30 name=vlan30-iot\
add interface=bridge1 vlan-id=40 name=vlan40-cam\
add interface=bridge1 vlan-id=99 name=vlan99-mgmt\

/ip address\
add address=192.168.10.1/24 interface=vlan10-office\
add address=192.168.20.1/24 interface=vlan20-guest\
add address=192.168.30.1/24 interface=vlan30-iot\
add address=192.168.40.1/24 interface=vlan40-cam\
add address=192.168.99.1/24 interface=vlan99-mgmt\

### Изоляция: IoT и камеры не ходят в офисную сеть
/ip firewall filter\
add chain=forward in-interface=vlan30-iot out-interface=vlan10-office action=drop comment="Block IoT to office"\
add chain=forward in-interface=vlan40-cam out-interface=vlan10-office action=drop comment="Block cameras to office"\
add chain=forward in-interface=vlan20-guest dst-address=192.168.0.0/16 action=drop comment="Block guest to LAN"

### Включаем фильтрацию — последний шаг!
/interface bridge set bridge1 vlan-filtering=yes