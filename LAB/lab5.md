# Experiment 5: Docker - Volumes, Environment Variables, Monitoring & Networks

## Part 1: Docker Volumes - Persistent Data Storage

### Lab 1: Understanding Data Persistence

**The Problem: Container Data is Ephemeral**

```bash
# Create a container that writes data
docker run -it --name test-container ubuntu /bin/bash

# Inside container:
echo "Hello World" > /data/message.txt
cat /data/message.txt # Shows "Hello World"
exit

# Restart container
docker start test-container
docker exec test-container cat /data/message.txt

# ERROR: File doesn't exist! Data was lost.
```

**Solution: Docker Volumes**

### Lab 2: Volume Types

**1. Anonymous Volumes**

![](../Screenshots/Lab5_s/lab5.2.png)

**2. Named Volumes**

![](../Screenshots/Lab5_s/lab5.3.png)

**3. Bind Mounts (Host Directory)**

![](../Screenshots/Lab5_s/lab5.4.png)

### Lab 3: Practical Volume Examples

**Example 1: Database with Persistent Storage**

![](../Screenshots/Lab5_s/lab5.5.png)

**Example 2: Web App with Configuration Files**

![](../Screenshots/Lab5_s/lab5.6.png)

### Lab 4: Volume Management Commands
```bash
# List all volumes
docker volume ls

# Create a volume
docker volume create app-volume

# Inspect volume details
docker volume inspect app-volume

# Remove unused volumes
docker volume prune

# Remove specific volume
docker volume rm volume-name

# Copy files to/from volume
docker cp local-file.txt container-name:/path/in/volume
```

---

## Part 2: Environment Variables

### Lab 1: Setting Environment Variables

**Method 1: Using -e flag**
```bash
# Single variable
docker run -d \
  --name app1 \
  -e DATABASE_URL="postgres://user:pass@db:5432/mydb" \
  -e DEBUG="true" \
  -p 3000:3000 \
  my-node-app

# Multiple variables
docker run -d \
  -e VAR1=value1 \
  -e VAR2=value2 \
  -e VAR3=value3 \
  my-app
```

**Method 2: Using --env-file**

![](../screenshots/Lab5i/6.png)

**Method 3: In Dockerfile**

![](../screenshots/Lab5i/7.png)

### Lab 2: Environment Variables in Applications

**Python Flask Example (`app.py`)**

![](../screenshots/Lab5i/8.png)

**Dockerfile with Environment Variables**

![](../screenshots/Lab5i/9.png)

### Lab 3: Test Environment Variables
```bash
# Run with custom env vars
docker run -d \
  --name flask-app \
  -p 5000:5000 \
  -e DATABASE_HOST="prod-db.example.com" \
  -e DEBUG="true" \
  -e PORT="8080" \
  flask-app

# Check environment in running container
docker exec flask-app env
docker exec flask-app printenv DATABASE_HOST

# Test the endpoint
curl http://localhost:5000/config
```

---

## Part 3: Docker Monitoring

### Lab 1: Basic Monitoring Commands

**`docker stats` - Real-time Container Metrics**

# Live stats for all containers
![](../screenshots/Lab5i/10.png)

# Specific format output
![](../screenshots/Lab5i/11.png)

# No-stream (single snapshot)
![](../screenshots/Lab5i/12.png)

# All containers (including stopped)
![](../screenshots/Lab5i/13.png)


**Useful Format Options:**

# Custom format
docker stats --format "Container: {{.Name}} | CPU: {{.CPUPerc}} | Memory: {{.MemPerc}}"
![](../screenshots/Lab5i/14.png)

# JSON output
docker stats --format json --no-stream
![](../screenshots/Lab5i/15.png)

# Wide output
docker stats --no-stream --no-trunc
![](../screenshots/Lab5i/16.png)


### Lab 2: `docker top` Process Monitoring

# View processes in container
docker top container-name
![](../screenshots/Lab5i/17.png)

# View with full command line
docker top container-name -ef
(same as before)

# Compare with host processes
ps aux | grep docker
![](../screenshots/Lab5i/18.png)


### Lab 3: `docker logs` Application Logs

# View logs
docker logs container-name
![](../screenshots/Lab5i/19.png)

# Follow logs (like tail -f)
docker logs -f container-name

# Last N lines
docker logs --tail 100 container-name
![](../screenshots/Lab5i/20.png)

# Logs with timestamps
docker logs -t container-name
![](../screenshots/Lab5i/21.png)

### Lab 4: Container Inspection

# Detailed container info
docker inspect container-name
![](../screenshots/Lab5i/23.png)

# Specific information
docker inspect --format='{{.State.Status}}' container-name
docker inspect --format='{{.NetworkSettings.IPAddress}}' container-name
docker inspect --format='{{.Config.Env}}' container-name
![](../screenshots/Lab5i/24.png)

# Resource limits
docker inspect --format='{{.HostConfig.Memory}}' container-name
docker inspect --format='{{.HostConfig.NanoCpus}}' container-name
![](../screenshots/Lab5i/25.png)


### Lab 5: Events Monitoring
```bash
# Monitor Docker events in real-time
docker events

# Filter events
docker events --filter 'type=container'
docker events --filter 'event=start'
docker events --filter 'event=die'

# Since specific time
docker events --since '2024-01-15'

# Format output
docker events --format '{{.Type}} {{.Action}} {{.Actor.Attributes.name}}'
```

### Lab 6: Practical Monitoring Script (`monitor.sh`)

![](../screenshots/Lab5i/26.png)
![](../screenshots/Lab5i/27.png)

---

## Part 4: Docker Networks

### Lab 1: Understanding Docker Network Types

**List Networks**

# Default networks
docker network ls
![](../screenshots/Lab5i/28.png)


### Lab 2: Network Types Explained

**1. Bridge Network (Default)**

# Create custom bridge network
docker network create my-network

# Inspect network
docker network inspect my-network
![](../screenshots/Lab5i/29.png)

# Run containers on custom network
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx

# Containers can communicate using container names
docker exec web1 curl http://web2
![](../screenshots/Lab5i/31.png)


**2. Host Network**

![](../screenshots/Lab5i/32.png)

**3. None Network**

![](../screenshots/Lab5i/33.png)

**4. Overlay Network (Swarm)**
```bash
# For Docker Swarm multi-host networking
docker network create --driver overlay my-overlay
```

### Lab 3: Network Management Commands
```bash
# Create network
docker network create app-network
docker network create --driver bridge --subnet 172.20.0.0/16 --gateway 172.20.0.1 my-subnet

# Connect container to network
docker network connect app-network existing-container

# Disconnect container from network
docker network disconnect app-network container-name

# Remove network
docker network rm network-name

# Prune unused networks
docker network prune
```

### Lab 4: Multi-Container Application Example

![](../screenshots/Lab5i/34.png)

### Lab 5: Network Inspection & Debugging

# Inspect network
docker network inspect bridge
![](../screenshots/Lab5i/35.png)

# Check container IP
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container-name
![](../screenshots/Lab5i/36.png)

# View network ports
docker port container-name
![](../screenshots/Lab5i/37.png)


### Lab 6: Port Publishing vs Exposing

# PORT PUBLISHING (host:container)
docker run -d -p 80:8080 --name app1 nginx

# Dynamic port publishing
docker run -d -p 8080 --name app2 nginx
![](../screenshots/Lab5i/38.png)

# Multiple ports
docker run -d -p 80:80 -p 443:443 --name app3 nginx
![](../screenshots/Lab5i/39.png)

# Specific host IP
docker run -d -p 127.0.0.1:8080:80 --name app4 nginx
![](../screenshots/Lab5i/40.png)

EXPOSE in Dockerfile (metadata only)
Dockerfile: EXPOSE 80
Still need -p to publish


---

## Part 5: Complete Real-World Example

**Application Architecture:**
* Flask Web App (port 5000)
* PostgreSQL Database (port 5432)
* Redis Cache (port 6379)
* All connected via custom network

**Monitoring Commands:**
```bash
# Check all components
docker ps

# Monitor resources
docker stats postgres redis flask-app

# Check logs
docker logs -f flask-app

# Network connectivity test
docker exec flask-app ping -c 2 postgres
docker exec flask-app ping -c 2 redis

# View network details
docker network inspect myapp-network
```

---

## Quick Reference Cheatsheet

**Volumes:**
```bash
docker volume create <name>
docker run -v <volume>:/path
docker run -v /host/path:/container/path
docker volume ls
docker volume rm <name>
```

**Environment Variables:**
```bash
docker run -e VAR=value
docker run --env-file .env
# In Dockerfile: ENV VAR=value
```

**Monitoring:**
```bash
docker stats
docker logs -f <container>
docker top <container>
docker inspect <container>
docker events
```

**Networks:**
```bash
docker network create <name>
docker run --network <name>
docker network connect <network> <container>
docker network inspect <network>
```

---

## Practice Exercises

**Exercise 1: Database Backup**
* Create a PostgreSQL container with volume
* Backup data using `docker cp` or volume backup techniques
* Restore to new container

**Exercise 2: Multi-Service Setup**
* Create: web app + database + cache
* Use custom network for communication
* Set environment variables for configuration
* Monitor all services

**Exercise 3: Log Analysis**
* Run a container that generates logs
* Use `docker logs` with various filters
* Redirect logs to a file on host using bind mount

**Exercise 4: Network Isolation**
* Create two separate networks
* Put containers in different networks
* Test connectivity between networks
* Connect a container to both networks

---

## Cleanup

```bash
# Stop and remove all containers
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

# Remove all volumes
docker volume prune -f

# Remove all networks (except defaults)
docker network prune -f

# Remove unused images
docker image prune -f
```

---

## Key Takeaways
1. **Volumes** persist data beyond container lifecycle.
2. **Environment variables** configure containers dynamically.
3. **Monitoring commands** help debug and optimize containers.
4. **Networks** enable secure container communication.
5. Always use named volumes for production data.
6. Custom networks provide better isolation and DNS.
7. Monitor resource usage to prevent issues.
8. Use `.env` files for sensitive configuration.

*This experiment covers essential Docker features for building, configuring, and managing production-ready containerized applications.*
