
Project 2 Documentation [Kamdon Marshall]
---

## Docker Compose Installation (Supa EZ)

-  Refresh packages
	- `sudo pacman -Syu`
- Install Docker Compose
	- `sudo pacman -S docker docker-compose`
- Enable, start, verify
	- `sudo systemctl enable docker.service`
	- `sudo systemctl start docker.service`
	- `docker info`
	- `sudo docker run hello-world`
- Check the version to make sure I got the right one
	- `docker-compose --version` (Output: 2.40.3)

## Uptime Kuma

- Create a PD for Uptime Kuma
	- `mkdir -p ~/docker/uptime-kuma`
	- `cd ~/docker/uptime-kuma`
- Create the .yml file
	- `nano docker-compose.yml
		- `version: "3.9" services: uptime-kuma: image: louislam/uptime-kuma:latest container_name: uptime-kuma ports:- "3001:3001" volumes: - ./data:/app/data restart: unless-stopped`
- Launch Uptime Kuma
	- `sudo docker-compose up -d`
- Ensure it is running
	- `sudo docker ps`
- Check to see if it is working online
	- `localhost:3001` 
	- ![[docker ps (uptime-kuma).png]]
