# **1. Create DigitalOcean Droplet**
After creating an account...
### **Droplet Specs**
- Region: NYC
- Image: Ubuntu 24.04 LTS
- Size: Basic / 1 CPU / 1 GB RAM
- My droplet’s public IP:
	- `138.197.23.123`
- Pass: **Redacted** lol
<img width="2528" height="1323" alt="Digital Ocean" src="https://github.com/user-attachments/assets/ed653029-75f6-4b83-8de0-a0fe41705218" />

# **2. Install Docker**
I SSH'd into the droplet using my terminal (bounced back and forth between this and DigitalOcean's web console too)

```
ssh root@138.197.23.123
```

Installed Docker using:
```
apt update
apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

Checked to see that installed:
- sudo docker --version

# **3. Installing Wireguard
A directory for Wireguard to be created:
- `mkdir ~/wireguard`
- `cd ~/wireguard`
Docker Compose File *(same as the prev. docker proj)*:
- `nano docker-compose.yml`
```
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - SERVERURL=138.197.23.123
      - SERVERPORT=51820
      - PEERS=1
      - PEERDNS=1.1.1.1
    volumes:
      - ./config:/config
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

Started Docker & Checked Logs:
```
docker compose up -d
docker logs wireguard // for the qr code
```

Getting the Config:
```
docker exec -it wireguard /app/show-peer 1 // qr
ls ~/wireguard/config/peer1/
cat ~/wireguard/config/peer1/peer1.conf

```

# **4. Testing VPN on mobile device, EZ
https://ipleak.net/

1. Installed WireGuard on my phone
2. Inside the app, "Add Tunnel, Create from QR Code", and then scanned the QR code that was generated before
3. VPN OFF
	- ![phone tunnel vpn off](https://github.com/user-attachments/assets/75008e78-e4b5-4b58-8fff-4af385a3f873)
    - ![phone vpn (off)](https://github.com/user-attachments/assets/12e00bf5-c06e-4e63-9903-b7dd7783e999)

4. VPN ON![Uploading phone vpn (off).jpg…]()
	- ![phone vpn (on)](https://github.com/user-attachments/assets/223dc9a7-3168-4470-8159-93c9df054118)
	- ![phone tunnel vpn on](https://github.com/user-attachments/assets/86b2249e-723e-4362-b91f-62630348edbd)

# **5. Downloaded `peer1.conf` locally:**

I was having difficulty with Arch on this part. I could not get the peer1.conf to be found by WireGuard on Arch. Server was workign fine, but I was having problems client-side, so I just installed it locally on my computer instead using WinSCP (I just wanted to use WinSCP lol)

Connected to the droplet: 
- Protocol: SCP
- Host: `138.197.23.123`
- Username: `root`
- Pass: same as the Digital Ocean one

Downloaded the peer1.conf to my desktop
# **6. WireGuard VPN**
## **6.1 Import the Tunnel on WireGuard Desktop (Windows)**
1. Chose **“Import tunnel(s) from file”**
2. Selected `peer1.conf`
## **6.2 Testing the Connection on Computer
https://ipleak.net/

1) VPN OFF
	- <img width="2558" height="1377" alt="VPN (OFF)" src="https://github.com/user-attachments/assets/0b8b80cc-a1dd-440f-89ac-5b2594a66e3d" />

2) VPN ON
	- <img width="2558" height="1377" alt="VPN (ON)" src="https://github.com/user-attachments/assets/5cb6a4f1-ed81-4c69-9ffc-75bf6918b6d5" />
