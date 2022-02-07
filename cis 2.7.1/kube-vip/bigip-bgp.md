#access the IMI Shell
imish

#Switch to enable mode
enable

#Enter configuration mode
config terminal

#Setup route bgp with AS Number 64512
router bgp 64512

#Create BGP Peer group
neighbor kube-vip-k8s peer-group

#assign peer group as BGP neighbors
neighbor kube-vip-k8s remote-as 64512

#we need to add all the peers: the other BIG-IP, our k8s components
# For Ipv6, provide relevant ip address
neighbor <BIG-IP1 IP ADDRESS> peer-group kube-vip-k8s

#save configuration
write

#exit
end