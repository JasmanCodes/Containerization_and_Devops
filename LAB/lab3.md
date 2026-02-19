# Lab Report: Deploying NGINX Using Different Base Images and Comparing Image Layers

**Lab Objectives:**
* Deploy NGINX using Official, Ubuntu-based, and Alpine-based images.
* Understand Docker image layers and size differences.
* Compare performance, security, and use-cases of each approach.

---

## Part 1: Deploy NGINX Using Official Image (Recommended Approach)

The official NGINX image is pre-optimized, uses a Debian-based OS internally, and requires minimal configuration.

### Step 1: Pull the Image
We retrieved the latest official NGINX image from the Docker registry.

![Pulling Official NGINX Image](../Screenshots/lab3_s/lab3.1.png)
*Command:* `docker pull nginx:latest`

### Step 2: Run the Container
We ran the container in detached mode (`-d`) naming it `nginx-official` and mapped port 8080 on the host to port 80 in the container.

![Running Official NGINX Container](../screenshots/Lab3i/docker_run.png)
*Command:* `docker run -d --name nginx-official -p 8080:80 nginx`

### Step 3: Verify
We verified the deployment by accessing the localhost URL. The response confirms the NGINX welcome page is active.

![Verifying NGINX Response](../screenshots/Lab3i/verify.png)
*Command:* `curl http://localhost:8080`

### Key Observations
We checked the size of the downloaded official image.

![Official NGINX Image Size](../screenshots/Lab3i/docker_images_nginx.png)
*Observation:* The official image size is approximately 161MB.

---

## Part 2: Custom NGINX Using Ubuntu Base Image

In this section, we built a custom image using `ubuntu:22.04` as the base. This approach results in a larger image but provides full OS utilities, which is useful for debugging.

### Step 1: Build the Image
We created a `Dockerfile` that installs NGINX on top of Ubuntu and built the image with the tag `nginx-ubuntu`.

![Building Ubuntu-based NGINX Image](../screenshots/Lab3i/docker_build.png)
*Command:* `docker build -t nginx-ubuntu .`

### Step 2: Run the Container
We ran the Ubuntu-based container on port 8081.

![Running Ubuntu Container](../screenshots/Lab3i/docker_run_dockerfile_ubuntu.png)
*Command:* `docker run -d --name nginx-ubuntu -p 8081:80 nginx-ubuntu`

### Observations
The resulting image is significantly larger due to the inclusion of full OS utilities.

![Ubuntu-based NGINX Image Size](../screenshots/Lab3i/docker_images_dockerfile_ubuntu.png)
*Observation:* The image size is approximately 134MB.

---

## Part 3: Custom NGINX Using Alpine Base Image

We built a custom image using `alpine:latest`. Alpine Linux is a security-oriented, lightweight Linux distribution.

### Step 1: Build Image
We built the image tagged `nginx-alpine`.

![Building Alpine-based NGINX Image](../screenshots/Lab3i/docker-build-alpine.png)
*Command:* `docker build -t nginx-alpine .`

### Step 2: Run the Container
We ran the Alpine-based container mapping port 8082 to port 80.

![Running Alpine NGINX Container](../screenshots/Lab3i/docker_run_alpine.png)
*Command:* `docker run -d --name nginx-alpine -p 8082:80 nginx-alpine`

### Observations
The Alpine-based image is extremely small compared to the others.

![Alpine-based NGINX Image Size](../screenshots/Lab3i/docker_images_alpine.png)
*Observation:* The image size is approximately 10.4MB.

---

## Part 4: Image Size and Layer Comparison

### Compare Sizes
We compared all three images side-by-side to visualize the size differences.

![Image Size Comparison](../screenshots/Lab3i/docker-images-comparison.png)

### Inspect Layers
We inspected the layer history of all three images.
* **Ubuntu** has many filesystem layers due to the full OS utilities.
* **Alpine** has minimal layers, contributing to its small size.
* **Official NGINX** is optimized but heavier than Alpine.

![Docker History Comparison](../screenshots/Lab3i/docker_history_comaprison.png)
*Command:* `docker history nginx`, `docker history nginx-ubuntu`, `docker history nginx-alpine`.

---

## Part 5: Functional Tasks (Serving Custom Content)

### Step 1: Create Custom HTML
We created a directory named `html` and added a custom `index.html` file.

![Creating Custom HTML](../screenshots/Lab3i/docker_mkdir_html.png)
*Commands:*
* `mkdir html`
* `echo "<h1>Hello from Docker NGINX</h1>" > html/index.html`

### Step 2: Run with Volume Mount
We ran a new container, mounting the local `html` directory to `/usr/share/nginx/html` inside the container. This allows NGINX to serve our custom file.

![Running NGINX with Volume Mount](../screenshots/Lab3i/docker_run_-d.png)
*Command:* `docker run -d -p 8083:80 -v $(pwd)/html:/usr/share/nginx/html nginx`

---

## Part 6: Comparison Summary

| Feature | Official NGINX | Ubuntu NGINX | Alpine NGINX |
| :--- | :--- | :--- | :--- |
| **Image Size** | Medium | Large | Very Small |
| **Ease of Use** | Very Easy | Medium | Medium |
| **Startup Time** | Fast | Slow | Very Fast |
| **Debugging Tools** | Limited | Excellent | Minimal |
| **Security Surface** | Medium | Large | Small |
| **Production Ready** | Yes | Rarely | Yes |

---

## Part 7: When to Use What

* **Official NGINX Image:** Recommended for production deployment, standard web hosting, and reverse proxy/load balancer use cases.
* **Ubuntu-Based Image:** Best for learning Linux/NGINX internals or when heavy debugging and custom system-level dependencies are required.
* **Alpine-Based Image:** Ideal for microservices, CI/CD pipelines, and cloud/Kubernetes workloads due to its small footprint.
