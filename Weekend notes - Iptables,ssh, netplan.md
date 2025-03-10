
``` 
[AP] [tgt1] [tgt2] [tgt3]
```

## Reverse tunnel

from AP 
```bash
ssh -L 1111:TGT2:22 user@TGT1
```

```bash 
ssh -R 0.0.0.0:8080:localhost:80 user@localhost -p 1111
```

```bash
GatewayPorts yes
```

---
## 📌 Practice 1: Tunnel from AP → TGT1 → TGT2 on Port 8080

```bash
[AP] [TGT1] [TGT2] [TGT3]
```

```bash 
ssh -L 8080:TGT2:8080 user@TGT1 -N -f
```

```bash
curl http://localhost:8080
```

## 📌 Practice 2: Tunnel from AP → TGT2 → TGT1

```bash 
ssh -R 9090:TGT1:22 user@TGT2 -N -f
```

```bash
ssh -p 9090 user@localhost
```
### **🔹 Explanation**

- `-R 9090:TGT1:22`: Any connection to `TGT2:9090` will be forwarded to `TGT1:22`.
- `user@TGT2`: SSH into `TGT2` as `user`.
- `-N`: No shell, just forwarding.
- `-f`: Background process.
## **🔥 Summary**

| **Test**   | **Command**                             | **Purpose**                         |
| ---------- | --------------------------------------- | ----------------------------------- |
| **Test 1** | `ssh -L 8080:TGT2:8080 user@TGT1 -N -f` | AP → TGT1 → TGT2 (Port 8080)        |
| Test 2     | ssh -R 9090:TGT1:22 user@TGT2 -N -f     | AP → TGT2 → TGT1 (Shared Interface) |

---

## Static IPv6 addresses NETPLAN 

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
      addresses:
        - "192.168.1.101/24"
      routes:
        - to: "default"
          via: "192.168.1.1"
    ens34:
      addresses:
        - "10.10.1.1/24"
        - "fd00:10:10:1::1/64"
    ens35:
      addresses:
        - "10.10.2.1/24"
        - "fd00:10:10:2::1/64"
```

---
# SSH IPv6 tunnel Practice
## 📌 Test 1: Tunnel from AP → TGT1 → TGT2 on Port 8080

```bash 
ssh -L [::1]:8080:[fd00:10:10:2::2]:8080 user@[fd00:10:10:1::1]
```
### **🔹 Explanation**

- **`-L [::1]:8080:[fd00:10:10:2::2]:8080`**
    - Traffic from `AP:8080` forwards to `TGT2:8080` through `TGT1`.
- **`user@[fd00:10:10:1::1]`**
    - SSH into `TGT1` via its IPv6 ULA (`fd00:10:10:1::1`).

```bash 
curl -g -6 http://[::1]:8080
```

## 📌 Test 2: Tunnel from AP → TGT2 → TGT1

```bash 
ssh -R [fd00:10:10:2::2]:9090:[fd00:10:10:1::1]:22 user@[fd00:10:10:2::2] -N -f
```
### **🔹 Explanation**

- **`-R [fd00:10:10:2::2]:9090:[fd00:10:10:1::1]:22`**
    - Any connection to `TGT2:9090` forwards to `TGT1:22`.
- **`user@[fd00:10:10:2::2]`**
    - SSH into `TGT2` via its IPv6 ULA (`fd00:10:10:2::2`).

```bash
ssh -p 9090 user@[::1]
```


## **🔥 Summary**

|**Test**|**Command**|**Purpose**|
|---|---|---|
|**Test 1 (Forward Tunnel)**|`ssh -L [::1]:8080:[fd00:10:10:2::2]:8080 user@[fd00:10:10:1::1] -N -f`|AP → TGT1 → TGT2 (Port 8080)|
|**Test 2 (Reverse Tunnel)**|`ssh -R [fd00:10:10:2::2]:9090:[fd00:10:10:1::1]:22 user@[fd00:10:10:2::2] -N -f`|AP → TGT2 → TGT1 (Shared Interface)|

## **Ensure IPv6 firewall allows SSH (port 22) & forwarding**:

```bash 
sudo ip6tables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo ip6tables -A INPUT -p tcp --dport 8080 -j ACCEPT
```

---
## **🔹 Step 1: Create SSH Tunnels (Run on AP)**

### **✅ IPv4 SSH Tunnel**

```bash
ssh -L 9999:10.10.3.2:9999 user@10.10.1.2 -N -f
```
### **✅ IPv6 SSH Tunnel**

```bash
ssh -L [::1]:9999:[fd00:10:10:3::2]:9999 user@[fd00:10:10:1::2] -N -f
```
### ** Explanation**

- `-L <local>:<remote>:<remote-port>` → Redirects traffic from **AP** to **TGT3**.
- **IPv4 Tunnel:**
    - Listens on `AP:9999`.
    - Forwards traffic to `TGT3:9999` via `TGT1`.
- **IPv6 Tunnel:**
    - Listens on `[::1]:9999` on AP.
    - Forwards traffic to `[fd00:10:10:3::2]:9999` via `TGT1`.

---

## **🔹 Step 2: Start Netcat Listener on TGT3**

On **TGT3**, start a **Netcat listener**:
```bash
nc -lvnp 9999
```
- **IPv4 Listener**: `nc -lvnp 9999` listens on `10.10.3.2:9999`.
- **IPv6 Listener**: `nc -lvnp 9999 -6` listens on `[fd00:10:10:3::2]:9999`.

---

## **🔹 Step 3: Send Data Through the Tunnel**

On **AP**, send a message via **IPv4**:
```bash
echo "Hello TGT3 from AP (IPv4)!" | nc -4 127.0.0.1 9999
```

On **AP**, send a message via **IPv6**:
```bash
echo "Hello TGT3 from AP (IPv6)!" | nc -6 ::1 9999
```

## **📌 Final Summary**

|**Step**|**Command**|**Machine**|
|---|---|---|
|**1. Create SSH Tunnel (IPv4)**|`ssh -L 9999:10.10.3.2:9999 user@10.10.1.2 -N -f`|`AP`|
|**2. Create SSH Tunnel (IPv6)**|`ssh -L [::1]:9999:[fd00:10:10:3::2]:9999 user@[fd00:10:10:1::2] -N -f`|`AP`|
|**3. Start Netcat Listener**|`nc -lvnp 9999`|`TGT3`|
|**4. Send Data (IPv4)**|`echo "Hello TGT3 from AP (IPv4)!"|nc -4 127.0.0.1 9999`|
|**5. Send Data (IPv6)**|`echo "Hello TGT3 from AP (IPv6)!"|nc -6 ::1 9999`|

---

### **📌 Home Network Topology Table**

|**Host**|**Interface**|**IPv4 Address**|**IPv6 Address (ULA)**|**Purpose**|
|---|---|---|---|---|
|**AP**|`ens33`|`192.168.1.X/24`|`fd00:1::AP/64`|External (Updates)|
|**AP**|`ens34`|`10.10.1.1/24`|`fd00:10:10:1::1/64`|Internal (To TGT1)|
|**TGT1**|`ens33`|`192.168.1.101/24`|`fd00:1::1/64`|External (Updates)|
|**TGT1**|`ens34`|`10.10.1.2/24`|`fd00:10:10:1::2/64`|Connected to AP|
|**TGT1**|`ens35`|`10.10.2.1/24`|`fd00:10:10:2::1/64`|Connected to TGT2|
|**TGT2**|`ens33`|`192.168.1.102/24`|`fd00:1::2/64`|External (Updates)|
|**TGT2**|`ens34`|`10.10.2.2/24`|`fd00:10:10:2::2/64`|Connected to TGT1|
|**TGT2**|`ens35`|`10.10.3.1/24`|`fd00:10:10:3::1/64`|Connected to TGT3|
|**TGT3**|`ens33`|`192.168.1.103/24`|`fd00:1::3/64`|External (Updates)|
|**TGT3**|`ens34`|`10.10.3.2/24`|`fd00:10:10:3::2/64`|Connected to TGT2|
|**TGT3**|`ens35`|`10.10.4.1/24`|`fd00:10:10:4::1/64`|Internal (Future Expansion)|
## No Shit works 
```bash 
ssh -L 9999:10.10.3.2:9999 s2
```
- Need to enable `9999` outbound on the devices we are touching
## NC iptables rules 

```bash 
iptables -A INPUT -p tcp --dport 9999 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p udp --dport 9999 -m conntrack --ctstate NEW -j ACCEPT
iptables -A OUTPUT -p tcp --dport 9999 -j ACCEPT
iptables -A OUTPUT -p udp --dport 9999 -j ACCEPT
```

## If Using Reverse Shell (Netcat -e)
```bash
iptables -A INPUT -p tcp --dport 4444 -m conntrack --ctstate NEW -j ACCEPT
iptables -A OUTPUT -p tcp --sport 4444 -j ACCEPT
```