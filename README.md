
**<h1>Securing Your SSH Service: Best Practices</h1>**  

The most common method to connect to a Linux system is via SSH. Here are several ways to enhance the security of your SSH service. Some tips may be familiar, others new. After applying these configurations, restart the SSH service:  
- For **RHEL-based systems**: `systemctl restart sshd`  
- For **Debian-based systems**: `systemctl restart ssh`  

---

### 🔰 **Disable Root Login**  
Modify `/etc/ssh/sshd_config` to disable root access:  
```bash  
PermitRootLogin no  
```  

---

### 🔰 **Use SSH Keys Instead of Passwords**  
1. In the SSH config file (`/etc/ssh/sshd_config`), disable password authentication:  
   ```bash  
   PasswordAuthentication no  
   ```  
2. Generate an SSH key pair (use a passphrase for extra security):  
   ```bash  
   ssh-keygen -t rsa -b 4096  
   ```  
3. Connect using the private key:  
   ```bash  
   ssh -i /path/to/private_key user@server  
   ```  

---

### 🔰 **Change the Default SSH Port (22)**  
1. Edit `/etc/ssh/sshd_config` and specify a custom port (e.g., 2929):  
   ```bash  
   Port 2929  
   ```  
2. **Open the new port in the firewall**:  
   - **RHEL**:  
     ```bash  
     firewall-cmd --permanent --add-port=2929/tcp  
     firewall-cmd --reload  
     ```  
   - **Debian**:  
     ```bash  
     ufw allow 2929/tcp  
     ufw reload  
     ```  

---

### 🔰 **Enable Fail2Ban**  
Install and configure Fail2Ban to block brute-force attacks:  
1. Install Fail2Ban:  
   - RHEL: `yum install fail2ban -y`  
   - Debian: `apt install fail2ban -y`  
2. Configure `/etc/fail2ban/jail.local` to block IPs after 3 failed attempts:  
   ```bash  
   [sshd]  
   enabled = true  
   maxretry = 3  
   bantime = 3600  
   findtime = 3600  
   ```  
3. Restart and enable Fail2Ban:  
   ```bash  
   systemctl restart fail2ban  
   systemctl enable fail2ban  
   ```  

---

### 🔰 **Restrict SSH Access to Specific Users**  
Add allowed users (e.g., `bob` and `alice`) to `/etc/ssh/sshd_config`:  
```bash  
AllowUsers bob alice  
```  

---

### 🔰 **Restrict Access via Firewall Rules**  
Allow SSH connections only from trusted IP addresses:  
- **RHEL (firewalld)**:  
  ```bash  
  firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="xxx.xxx.xxx.xxx" port protocol="tcp" port="2929" accept'  
  firewall-cmd --reload  
  ```  
- **Debian (UFW)**:  
  ```bash  
  ufw allow from xxx.xxx.xxx.xxx to any port 2929  
  ufw reload  
  ```  

---

### 🔰 **Port Knocking**  
Hide the SSH port until a specific sequence of "knocks" is received:  
1. Install `knockd`:  
   - RHEL: `yum install knock-server -y`  
   - Debian: `apt install knockd -y`  
2. Configure `/etc/knockd.conf`:  
   ```bash  
   [openSSH]  
   sequence = 7000,8000,9000  
   seq_timeout = 5  
   command = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 2929 -j ACCEPT  
   ```  
3. Enable and restart `knockd`:  
   ```bash  
   systemctl enable knockd  
   systemctl restart knockd  
   ```  
4. To connect, "knock" the ports in sequence:  
   ```bash  
   knock -v server_ip 7000 8000 9000  
   ssh user@server_ip  
   ```  
   The SSH port remains closed until the correct knock sequence is sent.  

---

By implementing these steps, you can significantly strengthen the security of your SSH service. 

Stay secure! 🤟

<br />




<p align="center">
 <br/>
<img src="https://i.imgur.com/lPeqEh3.png" height="80%" width="80%" alt="bank Network"/>


<br />

