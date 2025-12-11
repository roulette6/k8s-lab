# LLM prompts

## SNAT pools
I am using kubeadm with Calico open source as my CNI. I want to create outgoing NAT pools for my pods.

I only want outgoing NAT to take place when pods initiate a connection outside the cluster.

I want to use 192.168.2.0/24 as my NAT pool for all outbound NAT.

I want all the pods with the same label to get the same IP address assigned from the NAT pool. For example, all pods with the label of "app: nginx" should be assigned IP 192.168.2.10 and all pods with the label of "app: pihole" should be assigned IP address 192.168.2.11.


Is everything I want possible? If yes, give me the configs and instructions. If not, let me know what I can do instead. Ask me any questions if anything is unclear.

