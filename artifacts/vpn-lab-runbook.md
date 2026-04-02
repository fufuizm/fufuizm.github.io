# VPN Lab — Operational Runbook

## 1. Start / Stop Lab

```bash
# Start all VMs (VirtualBox headless)
VBoxManage startvm "pfSense"      --type headless
VBoxManage startvm "dns-server"   --type headless
VBoxManage startvm "nginx-dmz"    --type headless
VBoxManage startvm "docker-host"  --type headless

# Stop all VMs gracefully
for vm in pfSense dns-server nginx-dmz docker-host; do
  VBoxManage controlvm "$vm" acpipowerbutton
done
```

## 2. Connect via WireGuard VPN

```bash
# Bring up VPN tunnel (client side)
sudo wg-quick up wg0

# Verify connectivity
ping 192.168.20.10 -c 4
ssh -i ~/.ssh/lab_key furkan@192.168.20.10

# Check VPN status
sudo wg show
```

## 3. Verify Firewall Rules

```bash
# On pfSense via SSH
pfSsh.php playback viewconfigxml | grep rule

# On Linux VMs
sudo ufw status verbose
sudo iptables -L -n -v
```

## 4. Traffic Analysis

```bash
# Capture DNS traffic
sudo tcpdump -i eth0 port 53 -n

# Capture HTTP/HTTPS
sudo tcpdump -i eth0 'tcp port 80 or tcp port 443' -w capture.pcap

# Quick port scan (post-change validation)
nmap -sV -p 22,80,443,51820 192.168.20.10
```

## 5. Restart Services

```bash
# Restart WireGuard
sudo systemctl restart wg-quick@wg0

# Restart nginx
sudo systemctl restart nginx

# Restart DNS (Pi-hole)
sudo systemctl restart pihole-FTL
```

## 6. Rollback Procedure

1. Revert pfSense config via backup XML
2. Restore nginx config from git: `git checkout HEAD -- /etc/nginx/nginx.conf`
3. Re-apply ufw rules from script: `bash ~/scripts/apply-ufw.sh`
4. Verify with nmap and tcpdump
