# ip forwarding

### enable ip forwarding

Below is a **concise, step-by-step** rundown of how you configured both the **router** (CentOS) and the **client** (Ubuntu) based on the IP addresses and routing setup you shared.

---

## **Router (CentOS) Configuration**

### 1. Assign IPs to Network Interfaces

1. **WAN Interface** (`eno16777736`):
    - IP Address: `192.168.152.11/24`
    - Default Gateway: `192.168.152.2`*(This is your “external” side that can reach the Internet or upstream network.)*
2. **LAN Interface** (`ens34`):
    - IP Address: `10.0.0.1/24`*(This is the internal LAN subnet for your client.)*

### 2. Enable IP Forwarding

Make sure the system can forward packets between interfaces:

```bash
# Temporary enable
sudo sysctl -w net.ipv4.ip_forward=1

# Make it permanent (edit /etc/sysctl.conf or /etc/sysctl.d/*.conf)
net.ipv4.ip_forward = 1

sudo sysctl -p

```

### 3. Set Up NAT (Masquerading)

To allow the client on `10.0.0.x` to access external networks through your router’s WAN interface:

```bash
# Example using iptables
sudo iptables -t nat -A POSTROUTING -o eno16777736 -j MASQUERADE

# Allow forwarding from LAN to WAN
sudo iptables -A FORWARD -i ens34 -o eno16777736 -j ACCEPT

# Allow return traffic from WAN to LAN (RELATED,ESTABLISHED)
sudo iptables -A FORWARD -i eno16777736 -o ens34 -m state --state RELATED,ESTABLISHED -j ACCEPT

```

*(Make sure to separate states with a comma, e.g. `--state RELATED,ESTABLISHED`.)*

If you want these rules to persist after reboot, save them (for example `service iptables save`, or use `iptables-persistent` on Debian-based, etc., depending on your CentOS version and firewalld usage).

---

## **Client (Ubuntu) Configuration**

1. **LAN Interface** (`ens33`):
    - IP Address: `10.0.0.2/24`
    - Default Gateway: `10.0.0.1` *(the router’s LAN IP)*
2. **(Optional) DNS**:
    - If you need to resolve domain names, set a DNS server in your Netplan config or `/etc/resolv.conf`. Example:
        
        ```yaml
        nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
        
        ```
        
3. **Verify Configuration**:
    - Check the client’s default route:
    
    You should see:
        
        ```bash
        ip route
        
        ```
        
        ```
        default via 10.0.0.1 dev ens33
        10.0.0.0/24 dev ens33 ...
        
        ```
        

---

### ways to enable ip forwarding

### **1. Use `echo` with `sudo`**

This is the simplest and most common approach:

```bash
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'

```

- **Why it works:**
    - The `sh -c` ensures the entire command runs with root privileges, including the redirection.

---

### **2. Use the `sysctl` Command**

Another way to modify `ip_forward` is via `sysctl`:

```bash
sudo sysctl -w net.ipv4.ip_forward=1

```

- **Why it works:**
    - `sysctl` is designed for managing kernel parameters dynamically.
- **Check the Current Value:**
    
    ```bash
    sysctl net.ipv4.ip_forward
    
    ```
    

---

### **3. Make it Persistent**

If you want to make `ip_forward` enabled permanently (persists after a reboot), edit the `/etc/sysctl.conf` file:

```bash
sudo nano /etc/sysctl.conf

```

- Add this line:
    
    ```
    net.ipv4.ip_forward = 1
    
    ```
    
- Then reload the configuration:
    
    ```bash
    sudo sysctl -p
    
    ```
    

---

**Verify the Change**:

- After setting `ip_forward`, check its status:
    
    ```bash
    cat /proc/sys/net/ipv4/ip_forward
    
    ```
    
    - It should return `1` if the change was successful.