# Load-Balancing-LB-PCC-3-Line-ISP

################################################
# LOAD BALANCING
# https://fb.me/buananet.pbun
# Load Balancing Metode -> PCC
################################################

/ip firewall address-list
add address=192.168.0.0/24 list=LOCAL-IP
add address=172.16.0.0/24 list=LOCAL-IP
add address=10.0.0.0/24 list=LOCAL-IP

/ip firewall nat
add chain=srcnat out-interface="ether1" action=masquerade
add chain=srcnat out-interface="ether2" action=masquerade
add chain=srcnat out-interface="ether3" action=masquerade

/ip route
add check-gateway=ping distance=1 gateway="192.168.1.1" routing-mark="to-ether1"
add check-gateway=ping distance=1 gateway="192.168.2.1" routing-mark="to-ether2"
add check-gateway=ping distance=1 gateway="192.168.3.1" routing-mark="to-ether3"
add check-gateway=ping distance=1 gateway="192.168.1.1"
add check-gateway=ping distance=2 gateway="192.168.2.1"
add check-gateway=ping distance=3 gateway="192.168.3.1"

/ip firewall mangle
add action=mark-connection chain=input in-interface="ether1" new-connection-mark="cm-ether1" passthrough=yes
add action=mark-connection chain=input in-interface="ether2" new-connection-mark="cm-ether2" passthrough=yes
add action=mark-connection chain=input in-interface="ether3" new-connection-mark="cm-ether3" passthrough=yes
add action=mark-routing chain=output connection-mark="cm-ether1" new-routing-mark="to-ether1" passthrough=yes
add action=mark-routing chain=output connection-mark="cm-ether2" new-routing-mark="to-ether2" passthrough=yes
add action=mark-routing chain=output connection-mark="cm-ether3" new-routing-mark="to-ether3" passthrough=yes
add action=mark-connection chain=prerouting dst-address-list=!LOCAL-IP dst-address-type=!local new-connection-mark="cm-ether1" passthrough=yes per-connection-classifier=both-addresses-and-ports:3/0 src-address-list=LOCAL-IP
add action=mark-connection chain=prerouting dst-address-list=!LOCAL-IP dst-address-type=!local new-connection-mark="cm-ether2" passthrough=yes per-connection-classifier=both-addresses-and-ports:3/1 src-address-list=LOCAL-IP
add action=mark-connection chain=prerouting dst-address-list=!LOCAL-IP dst-address-type=!local new-connection-mark="cm-ether3" passthrough=yes per-connection-classifier=both-addresses-and-ports:3/2 src-address-list=LOCAL-IP
add action=mark-routing chain=prerouting connection-mark="cm-ether1" dst-address-list=!LOCAL-IP new-routing-mark="to-ether1" passthrough=yes src-address-list=LOCAL-IP
add action=mark-routing chain=prerouting connection-mark="cm-ether2" dst-address-list=!LOCAL-IP new-routing-mark="to-ether2" passthrough=yes src-address-list=LOCAL-IP
add action=mark-routing chain=prerouting connection-mark="cm-ether3" dst-address-list=!LOCAL-IP new-routing-mark="to-ether3" passthrough=yes src-address-list=LOCAL-IP
