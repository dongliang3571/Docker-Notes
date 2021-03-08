# Docker-Notes

### Cheat Sheet

https://github.com/wsargent/docker-cheat-sheet#dockerfile

### Exit codes

https://betterprogramming.pub/understanding-docker-container-exit-codes-5ee79a1d58f6

### FQA

- Should I put database into Docker on production?

Short answer No, long answer https://medium.com/@wkrzywiec/database-in-a-docker-container-how-to-start-and-whats-it-about-5e3ceea77e50

### Dockerfile Instructions

#### Docker `CMD`

##### Creating a Dockerfile with CMD and Building an Image**

1. Start by creating a new MyDockerImage folder to store your images in:

```bash
sudo mkdir MyDockerImage
```

2. Move into that folder and create a new Dockerfile:

```bash
cd MyDockerImage
sudo touch Dockerfile
```

3. Open the Dockerfile with your favorite text editor:

```bash
vim Dockerfile
```

4. Then, add the following content to the file:

```bash
FROM ubuntu
MAINTAINER sofija
RUN apt-get update
CMD [“echo”, “Hello World”]
```

In the content above, you can see that we used the CMD instruction to echo the message Hello World when the container starts up without a specified command.

5. Save and exit the file.

6. The next step is to build a Docker image from the newly made Dockerfile. Since we are still in the MyDockerImage directory, you don’t need to specify the location of the Dockerfile, just build the image by running:

```bash
sudo docker build .
```

7. The output will tell you the name of the container. You can check to see whether it is available among the locally stored images by running:

```bash
sudo docker images
```

##### Running a Docker Container with CMD

To see CMD in action, we’ll create a container based on the image made in the previous step.
Run the container with the command:

```bash
sudo docker run [image_name]
# output
# Hello World
```

Since there is no command-line argument, the container will run the default CMD instruction and display the Hello World message. However, if you add an argument when starting a container, it overrides the CMD instruction.

For example, add the hostname argument to the docker run command:

```bash
sudo docker run [image_name] hostname
# output
# some_hostname
```

Docker will run the container and the hostname command instead of the CMD’s echo command. You can see this in the output.

#### Docker `Entrypoint`

ENTRYPOINT is the other instruction used to configure how the container will run. Just like with CMD, you need to specify a command and parameters.

##### What is the difference between CMD and ENTRYPOINT?

You cannot override the ENTRYPOINT instruction by adding command-line parameters to the docker run command. By opting for this instruction, **you imply that the container is specifically built for such use.** That is, when you do `docker run [image] [some_parameters]`, we know that [some_parameters] are the parameters for ENTRYPOINT command.

##### Creating a Dockerfile with ENTRYPOINT and Building an Image

1. Use the Dockerfile created in the CMD section and edit the file to change the instruction. Open the existing file with a text editor:

```bash
sudo nano Dockerfile
```

2. Edit the content by replacing the CMD command with ENTRYPOINT:

```bash
FROM ubuntu
MAINTAINER sofija
RUN apt-get update
ENTRYPOINT [“echo”, “Hello World”]
```

3. Save and close the file.

##### Running a Docker Container with ENTRYPOINT

1. Build a new image using the following command:

```bash
sudo docker build .
```

2. The output should show you have successfully built the new image under a given name. Now let’s run it as a container without adding any command-line parameters:

```bash
sudo docker run [container_name]
```

Running a container using Docker Entrypoint.
The output will be the same as with CMD. This is because we haven’t added any arguments to the run command.

3. To see how ENTRYPOINT works, you need to add a parameter when starting a container. Use the same command as in the previous step and add something after the container name:

```bash
sudo docker run [container_name] KnowledgeBase
# output
# Hello World KnowledgeBase
```

As you see, **Docker did not override the initial instruction of echoing Hello World. It merely added the new parameter to the existing command**.

Although you can use ENTRYPOINT and CMD in both forms, **it is generally advised to stick to exec form**. This is the more reliable solution as shell form can occasionally bring about subtle issues in the process

### Docker `Entrypoint` with `CMD`

As you have seen so far, ENTRYPOINT and CMD are similar, but not the same. What’s more, these two instructions are not mutually exclusive. That’s right, it is possible to have both in your Dockerfile.

There are many situations in which combining CMD and ENTRYPOINT would be the best solution for your Docker container. In such cases, **the executable is defined with ENTRYPOINT, while CMD specifies the default parameter**.

If you are using both instructions, make sure to keep them in exec form.

#### Run a Container with Entrypoint and CMD

1. First, we are going to modify our existing Dockerfile so it includes both instructions. Open the file with:

```bash
sudo vim Dockerfile
```

2. The file should include an ENTRYPOINT instruction specifying the executable, as well as a CMD instruction defining the default parameter which should appear if no additional ones are added to the run command:

```bash
FROM ubuntu
MAINTAINER sofija
RUN apt-get update
ENTRYPOINT [“echo”, “Hello”]
CMD [“World”]
```

3. Now, build a new image from the modified Dockerfile:

```bash
sudo docker build .
```

4. Let’s test the container by running it without any parameters. Enter the command:

```bash
sudo docker run [container_name]
# output
# Hello World
```

It will return the message `Hello World`. However, what happens when we add parameters to the docker run command?

5. Use the same command again, but this time add your name to the run command:

```bash
sudo docker run [container_name] some_input
# output
# Hello some_input
```

The output has now changed to `Hello some_input`. This is because **you cannot override ENTRYPOINT instructions, whereas with CMD you can easily do so**.

### Docker Compose

#### Instructions

`domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir`

Each of these is a single value, analogous to its `docker run` counterpart. Note that mac_address is a legacy option.

##### hostname

Hostname is not used by docker's built in DNS service. It's a counterintuitive exception, but since hostnames can change outside of docker's control, it makes some sense. Docker's DNS will resolve:

    - the container id
    - container name
    - any network aliases you define for the container on that network

The easiest of these options is the last one which is automatically configured when running containers with a compose file. The service name itself is a network alias. This lets you scale and perform rolling updates without reconfiguring other containers.

You need to be on a user created network, not something like the default bridge which has DNS disabled. This is done by default when running containers with a compose file.

Avoid using links since they are deprecated. And I'd only recommend adding host entries for external static hosts that are not in any DNS, for container to container, or access to other hosts outside of docker, DNS is preferred.

##### aliases

say you have a network in your docker-compose.yaml file

```yaml
networks:
  default:
    name: some-name
    driver: bridge

services:
  first-sevice:
    networks:
        default: # corresponding to the default network above
            aliases:
            - some-aliase-you-want # other containers can reach to it by using this alias
            - some-aliase-you-want-2 # other containers can reach to it by using this alias
  second-service:
    context: a-path
    build: some-folder
```

