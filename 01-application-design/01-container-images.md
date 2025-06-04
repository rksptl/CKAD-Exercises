# Container Images with Podman

This section covers how to use Podman to build, manage, and modify container images, which is an essential skill for the CKAD exam (7% of the exam).

## Key Resources

- [Podman Documentation](https://podman.io/docs)
- [Kubernetes Dockershim Removal FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/)

## Introduction to Podman

Podman (Pod Manager) is a daemonless container engine for developing, managing, and running OCI containers on Linux systems. It provides a Docker-compatible command line interface that can simply replace Docker in most use cases.

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

```bash
:~$ podman build -t simpleapp .
STEP 1/2: FROM httpd:2.4
STEP 2/2: RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
COMMIT simpleapp
--> ef4b14a72d0
Successfully tagged localhost/simpleapp:latest
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
```

**Step 2: List the available images**

```bash
:~$ podman images
REPOSITORY               TAG         IMAGE ID      CREATED        SIZE
localhost/simpleapp      latest      ef4b14a72d02  8 seconds ago  148 MB
docker.io/library/httpd  2.4         98f93cd0ec3b  7 days ago     148 MB
```

**Step 3: Inspect the image layers**

```bash
:~$ podman image tree localhost/simpleapp:latest
Image ID: ef4b14a72d02
Tags:     [localhost/simpleapp:latest]
Size:     147.8MB
Image Layers
├── ID: ad6562704f37 Size:  83.9MB
├── ID: c234616e1912 Size: 3.072kB
├── ID: c23a797b2d04 Size: 2.721MB
├── ID: ede2e092faf0 Size: 61.11MB
├── ID: 971c2cdf3872 Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└── ID: 61644e82ef1f Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

**What this does**:

- `podman build -t simpleapp .` - Builds an image from the Dockerfile in the current directory and tags it as "simpleapp"
- `podman images` - Lists all images available locally
- `podman image tree` - Shows the layered structure of the image, with each layer representing a step in the Dockerfile
- The output shows that our custom image adds just one small layer (6.144kB) on top of the httpd base image

</p>
</details>

### Exercise 3: Running a Container and Testing It

**Goal**: Learn how to run a container from your custom image and verify it's working correctly.

**Task**: Start a container from your custom image, exposing port 8080, and test that the web server is serving your custom page.

**Why this matters**: Running containers is a fundamental skill for container orchestration. This exercise demonstrates how to expose container ports to the host system, a critical concept for networking in containerized environments. Proper testing ensures your containerized application functions as expected before deployment.

<details><summary>show solution</summary>
<p>

**Step 1: Run the container in the background**

```bash
:~$ podman run -d --name test -p 8080:80 localhost/simpleapp
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8
```

**Step 2: Check the running container**

```bash
:~$ podman ps
CONTAINER ID  IMAGE                       COMMAND           CREATED        STATUS            PORTS                 NAMES
2f3d7d613ea6  localhost/simpleapp:latest  httpd-foreground  5 seconds ago  Up 6 seconds ago  0.0.0.0:8080->80/tcp  test
```

**Step 3: Test the web server**

```bash
:~$ curl http://localhost:8080
Hello, World!
```

**What this does**:

- `podman run -d` - Runs the container in detached mode (in the background)
- `--name test` - Gives the container a name for easy reference
- `-p 8080:80` - Maps port 8080 on the host to port 80 in the container
- `localhost/simpleapp` - Specifies the image to use
- `podman ps` - Shows running containers
- `curl http://localhost:8080` - Tests that the web server is responding correctly

**Additional useful commands**:

- `podman stop test` - Stops the running container
- `podman start test` - Starts a stopped container
- `podman rm test` - Removes the container (must be stopped first)
- `podman logs test` - Shows the logs from the container
- `curl 0.0.0.0:8080` - Tests that the web server is responding correctly with our custom page

</p>
</details>
