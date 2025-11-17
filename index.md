
Project 2 Documentation
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
   		- <img width="1273" height="965" alt="docker info" src="https://github.com/user-attachments/assets/5bd6141b-cf6b-490a-a823-8a80ed5e8973" />
	- `sudo docker run hello-world`
		- <img width="955" height="750" alt="image" src="https://github.com/user-attachments/assets/5402a252-058d-4dee-8efd-2ce9d0211bda" />


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
   		- <img width="950" height="1003" alt="docker-compose up -d" src="https://github.com/user-attachments/assets/49a95de2-90b3-4cf6-9cfb-3551cb8518f0" />
- Ensure it is running
	- `sudo docker ps`
   		- <img width="952" height="747" alt="docker ps" src="https://github.com/user-attachments/assets/649cb86f-a4dc-4daa-89be-d592a77d42c2" />
- Check to see if it is working online
	- `localhost:3001` 
	- <img width="1920" height="1148" alt="docker ps (uptime-kuma)" src="https://github.com/user-attachments/assets/d5a0a750-c529-4763-81e1-aea9a562483a" />
