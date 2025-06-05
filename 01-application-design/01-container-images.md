# Container Images

This section covers how to build, manage, and modify container images, which is an essential skill for the CKAD exam (7% of the exam). You can use either of the following container tools based on your preference:

## Container Tools

### Docker
Docker is the most widely used container runtime with extensive documentation and community support. It provides a comprehensive ecosystem for building, running, and managing containers.

### containerd with nerdctl
containerd is the container runtime used by Kubernetes under the hood, and nerdctl provides a Docker-compatible CLI for containerd. Using containerd directly gives you experience with the same runtime that powers Kubernetes.

## Key Resources

- [Docker Documentation](https://docs.docker.com/)
- [containerd Documentation](https://containerd.io/docs/)
- [nerdctl GitHub](https://github.com/containerd/nerdctl)
- [Kubernetes Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/)

## Key Concepts

- **Containers**: Lightweight, standalone executable packages that include everything needed to run an application
- **Images**: Read-only templates used to create containers
- **Registries**: Repositories that store container images
- **Pods**: Groups of one or more containers that share storage and network resources
- **Dockerfile**: A text file with instructions for building a container image

## Working with Container Images

### Exercise 1: Creating a Dockerfile

**Goal**: Learn how to create a simple Dockerfile for an Apache HTTP server with a custom homepage.

**Task**: Create a Dockerfile that uses the httpd:2.4 base image and adds a custom index.html file.

**Why this matters**: Dockerfiles are the blueprint for container images. Understanding how to create them is fundamental to containerization, allowing you to package applications with their dependencies in a consistent, reproducible way. This skill is essential for creating custom images that meet your application's specific requirements.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Dockerfile**

Create a file named `Dockerfile` with the following content:

```Dockerfile
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

**What this does**:

- `FROM docker.io/httpd:2.4` - Specifies the base image to use (Apache HTTP Server version 2.4)
- `RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html` - Creates a simple index.html file with the text "Hello, World!"

</p>
</details>

### Exercise 2: Building an Image and Inspecting Layers

**Goal**: Learn how to build a container image from a Dockerfile and inspect its layer structure.

**Task**: Build the image from the Dockerfile created in Exercise 1 and examine the layers that make up the image.

**Why this matters**: Understanding image layers is crucial for creating efficient, smaller images. Each layer represents a step in your Dockerfile, and optimizing these layers can significantly reduce image size, improve build times, and enhance security by minimizing the attack surface.

<details><summary>show solution</summary>
<p>

**Step 1: Build the image**

<tabs>
<tab name="Docker">

```bash
$ docker build -t apache-hello-world .
STEP 1/2: FROM httpd:2.4
STEP 2/2: RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
Successfully built ef4b14a72d02
Successfully tagged apache-hello-world:latest
```
</tab>
<tab name="containerd/nerdctl">

```bash
$ nerdctl build -t apache-hello-world .
[+] Building 2.3s (5/5) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 115B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [1/2] FROM docker.io/library/httpd:2.4
 => [2/2] RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
 => exporting to image
 => => exporting layers
 => => writing image sha256:ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
 => => naming to docker.io/library/apache-hello-world:latest
```
</tab>
</tabs>

**Step 2: List the available images**

<tabs>
<tab name="Docker">

```bash
$ docker images
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
apache-hello-world   latest    ef4b14a72d02   8 seconds ago   148MB
httpd               2.4       98f93cd0ec3b   7 days ago      148MB
```
</tab>
<tab name="containerd/nerdctl">

```bash
$ nerdctl images
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
apache-hello-world   latest    ef4b14a72d02   8 seconds ago   148MB
httpd               2.4       98f93cd0ec3b   7 days ago      148MB
```
</tab>
</tabs>

**Step 3: Inspect the image layers**

<tabs>
<tab name="Docker">

```bash
$ docker image inspect apache-hello-world --format '{{.RootFS.Layers}}'
[sha256:ad6562704f37 sha256:c234616e1912 sha256:c23a797b2d04 sha256:ede2e092faf0 sha256:971c2cdf3872 sha256:61644e82ef1f]
```
</tab>
<tab name="containerd/nerdctl">

```bash
$ nerdctl image inspect apache-hello-world -f '{{.RootFS.Layers}}'
[sha256:ad6562704f37 sha256:c234616e1912 sha256:c23a797b2d04 sha256:ede2e092faf0 sha256:971c2cdf3872 sha256:61644e82ef1f]
```
</tab>
</tabs>

**What this does**:

<tabs>
<tab name="Docker">

- `docker build -t apache-hello-world .` - Builds an image from the Dockerfile in the current directory and tags it with a descriptive name
- `docker images` - Lists all images available locally
- `docker image inspect` - Shows detailed information about the image, including layers
- The output shows that our custom image adds just one small layer on top of the httpd base image
</tab>
<tab name="containerd/nerdctl">

- `nerdctl build -t apache-hello-world .` - Builds an image from the Dockerfile in the current directory and tags it with a descriptive name
- `nerdctl images` - Lists all images available locally
- `nerdctl image inspect` - Shows detailed information about the image, including layers
- The output shows that our custom image adds just one small layer on top of the httpd base image
</tab>
</tabs>

</p>
</details>

### Exercise 3: Running a Container and Testing It

**Goal**: Learn how to run a container from your custom image and verify it's working correctly.

**Task**: Start a container from your custom image, exposing port 8080, and test that the web server is serving your custom page.

**Why this matters**: Running containers is a fundamental skill for container orchestration. This exercise demonstrates how to expose container ports to the host system, a critical concept for networking in containerized environments. Proper testing ensures your containerized application functions as expected before deployment.

<details><summary>show solution</summary>
<p>

**Step 1: Run the container in the background**

<tabs>
<tab name="Docker">

```bash
$ docker run -d --name apache-web-server -p 8080:80 apache-hello-world
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8
```
</tab>
<tab name="containerd/nerdctl">

```bash
$ nerdctl run -d --name apache-web-server -p 8080:80 apache-hello-world
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8
```
</tab>
</tabs>

**Step 2: Check the running container**

<tabs>
<tab name="Docker">

```bash
$ docker ps
CONTAINER ID   IMAGE                COMMAND            CREATED        STATUS        PORTS                  NAMES
2f3d7d613ea6   apache-hello-world   "httpd-foreground" 5 seconds ago  Up 5 seconds  0.0.0.0:8080->80/tcp   apache-web-server
```
</tab>
<tab name="containerd/nerdctl">

```bash
$ nerdctl ps
CONTAINER ID   IMAGE                COMMAND            CREATED        STATUS        PORTS                  NAMES
2f3d7d613ea6   apache-hello-world   "httpd-foreground" 5 seconds ago  Up 5 seconds  0.0.0.0:8080->80/tcp   apache-web-server
```
</tab>
</tabs>

**Step 3: Test the web server**

```bash
$ curl http://localhost:8080
Hello, World!
```

**What this does**:

<tabs>
<tab name="Docker">

- `docker run -d` - Runs the container in detached mode (in the background)
- `--name apache-web-server` - Gives the container a descriptive name for easy reference
- `-p 8080:80` - Maps port 8080 on the host to port 80 in the container
- `apache-hello-world` - Specifies the image to use
- `docker ps` - Shows running containers
</tab>
<tab name="containerd/nerdctl">

- `nerdctl run -d` - Runs the container in detached mode (in the background)
- `--name apache-web-server` - Gives the container a descriptive name for easy reference
- `-p 8080:80` - Maps port 8080 on the host to port 80 in the container
- `apache-hello-world` - Specifies the image to use
- `nerdctl ps` - Shows running containers
</tab>
</tabs>

- `curl http://localhost:8080` - Tests that the web server is responding correctly

**Additional useful commands**:

<tabs>
<tab name="Docker">

- `docker stop apache-web-server` - Stops the running container
- `docker start apache-web-server` - Starts a stopped container
- `docker rm apache-web-server` - Removes the container (must be stopped first)
- `docker logs apache-web-server` - Shows the logs from the container
</tab>
<tab name="containerd/nerdctl">

- `nerdctl stop apache-web-server` - Stops the running container
- `nerdctl start apache-web-server` - Starts a stopped container
- `nerdctl rm apache-web-server` - Removes the container (must be stopped first)
- `nerdctl logs apache-web-server` - Shows the logs from the container
</tab>
</tabs>

- `curl localhost:8080` - Tests that the web server is responding correctly with our custom page

</p>
</details>
