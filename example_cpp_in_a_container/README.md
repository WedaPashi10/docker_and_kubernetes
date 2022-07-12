
I am trying to setup two containers:
- `cpp-base-build`: Which will have depedencies required for compiling the C++ code
- `cpp-hello-world`: For production where the code would be executed

#### Base Build Container

This container we will be using for compilcation of the C++ application code. Lets make it a very basic Ubuntu Linux base container. The container only includes the basic kernel and operating system software. There are no development tools or third-party libraries in this container.

Lets look at the `Dockerfile`:

    FROM ubuntu:focal
    RUN apt-get update && apt-get install -y build-essential cmake autoconf gcc g++
    
Lets break it down:

- `Ubuntu:focal` -- Because, I want this to be running on Ubuntu 20.04.4 LTS, which is named `Focal Fossa`
- `RUN` -- Fetched updated and installs `build-essential`, `cmake`, `autoconf`, `gcc` and `g++` packages, `-y` suppresses the `(Y/n)?` prompts.

Lets build it then:

    sudo docker build . -t cpp-build-base:0.1.0

- `sudo` -- Somehow I need to run this command as Superuser (Ouch!)
- `build` -- Command to build the container based on the `Dockerfile` in present working directory
- `.` -- Points to the `Dockerfile` in the present working directory
- `-t <name>` -- Name and optionally a tag in the 'name:tag' format
- `cpp-build-base:0.1.0` -- Tag

I faced an issue where the building of container get stuck for user-input relating to geographical information. 

    Configuring tzdata
    ------------------
    Please select the geographic area in which you live. Subsequent configuration
    questions will narrow this down by presenting a list of cities, representing
    the time zones in which they are located.
     1. Africa      4. Australia  7. Atlantic  10. Pacific  13. Etc
     2. America     5. Arctic     8. Europe    11. SystemV
     3. Antarctica  6. Asia       9. Indian    12. US
    Geographic area:

Input anything, it remains stuck. I don't care why this stucks, so I found a way to suppress this prompt by making the `tzdata` prompt non-interactive.
For this I added a line in `Dockerfile` just above `RUN` command. 

    ENV DEBIAN_FRONTEND=noninteractive
    
And the tried the `build` command again. It takes a few minutes to complete, but you'd see something like below on successful building:

    Successfully built f166d11d6f0d
    Successfully tagged cpp-build-base:0.1.0

I now have a base container that I can use to build a C++ application.

#### Execution Container:

If we don't `Hello, World!` in the first code, then we aren't doing it right, are we?

This Docker container will not contain the code and will not have any of the development tools.

Lets look at the `Dockerfile`:

    # Using the base build container that I created previously to build the program.
    FROM cpp-build-base:0.1.0 AS build
    
    # Setting the /src as the working/destination directory
    WORKDIR /src
    
    # Copying Makefile and main.cpp from my workspace into the Docker container in the /src directory.
    COPY main.cpp Makefile ./

    # Execute make command in the container
    RUN make

    # Creating a second container that is using the base Ubuntu Focal Fossa operating system.
    FROM ubuntu:focal
    
    # Setting the /opt/hello-world as the working/destination directory
    WORKDIR /opt/hello-world

    # Copying the a.out program from the first container and copying it to the /opt/hello-world directory in the second container.
    COPY --from=build /src/a.out ./

    # Execute 
    CMD ["./a.out"]

I guess that needs no further explanation. Lets build the container.

    sudo docker build . -t cpp-hello-world:1.0.0

And the output is:
    
    Sending build context to Docker daemon  4.096kB
    Step 1/8 : FROM cpp-build-base:0.1.0 AS build
     ---> f166d11d6f0d
    Step 2/8 : WORKDIR /src
     ---> Using cache
     ---> dbf1146df220
    Step 3/8 : COPY main.cpp Makefile ./
     ---> Using cache
     ---> b7a893025d2a
    Step 4/8 : RUN make
     ---> Using cache
     ---> 3f9caf3b90b0
    Step 5/8 : FROM ubuntu:focal
     ---> 20fffa419e3a
    Step 6/8 : WORKDIR /opt/hello-world
     ---> Using cache
     ---> 719dbf2bb4fa
    Step 7/8 : COPY --from=build /src/a.out ./
     ---> 91908ff96a0c
    Step 8/8 : CMD ["./a.out"]
     ---> Running in de987ff5dbb0
    Removing intermediate container de987ff5dbb0
     ---> 0b61e9d765b5
    Successfully built 0b61e9d765b5
    Successfully tagged cpp-hello-world:1.0.0

Now we have the containers built. Lets run the second container as it has the executable (a.out) we are looking forward to.

    docker run --rm -it cpp-hello-world:1.0.0
    Hello World!

There you go!

If I check what docker images I have:

    docker images
    
I get:
    
    REPOSITORY        TAG       IMAGE ID       CREATED        SIZE
    cpp-hello-world   1.0.0     0b61e9d765b5   2 hours ago    72.9MB
    <none>            <none>    3f9caf3b90b0   2 hours ago    439MB
    cpp-build-base    0.1.0     f166d11d6f0d   2 hours ago    439MB
    ubuntu            latest    27941809078c   5 weeks ago    77.8MB
    ubuntu            focal     20fffa419e3a   5 weeks ago    72.8MB
    hello-world       latest    feb5d9fea6a5   9 months ago   13.3kB

Check the two IMAGE_ID we know:
- `f166d11d6f0d` of `cpp-build-base`
- `0b61e9d765b5` of `cpp-hello-world`
