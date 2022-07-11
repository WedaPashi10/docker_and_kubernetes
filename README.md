# Docker and Kubernetes
Before taking a deeper dive, lets familiarize ourselves with crucial terminologies.

### Docker
- Is a container-based technology
- Is a tool for creating and managing `containers`
 
### Container
- Is a standardized unit of software:
  - A package of code and dependencies required to rn that code. Example: A specific GNU Make version required for compiling a certain piece of code
  - Always yields the exact same application and execution behavior, nomatter where or by whom it might be executed
- Different Development and Production environment
  - No longer _"Hey, it works on my machine!"_ answer to a bug! Build, Develop and test in exactly the same environment as where it would be deployed eventually.
- Different Development Environment within a same team
  - Yeah, we always have those star developers working on multiple projects simultaneously. Switching the library versions on project-to-project basis is a recipe for disaster.
  - Every team member must have the exactly same development environment

### Why Containers?
Makes you think, why would we want independent, standardized packages?
Imagine this -- You have a `C++` project, the `g++` version you are using is say `9.4.0`. Lets assume that the default `C++` standard it uses is `C++14`. For whatever reason, you want it to be `C++17`. This can be done is 2 ways - Either by updaing the Makefile (or build setting in IDE) and share it with all developers in the team, or by creating a container with all such required configurations and/or depedencies and shared with the team. Everyone gets the same pie! I believe the latter approach is safer.

### Are Virtual Machines and Containers intended for the same purpose?
Kind of, yes. You have a base (Host) OS and on top of it, you have a virtual OS instance running. It can have its own libraries, depedencies, dev tools, etc. There can be multiple instances of virtual OS as well. So, it works! However, it is resouce-exhaustive. This approach wastes a lot of space on the hard drive, eats up RAM and tends to be laggy. Another factor to consider is duplication of common elemts in multiple virtual OS instances. For example, all of them would have `vim`, `git`, `minicom`, `wireshark` installations -- a lot of duplication.

As opposed to that, in `Dockers`, you have Host OS, and a OS built-in or Emulated Container Support, a `Docker Engine` and `Containers` of each instace that we require. This `Docker Engine` is responsible to provide all the required resouces (such as installations) to `Containers`. This ensures functionality without having the requirement of having the bloated dependencies and OS instaces installed for each Container requirement. It is imaginable that this `Docker Engine` happens to be a light-weight utility for simulating the virtualization amongst all `Containers`.

So, when Docker Containers and Virtual Machines are compared:
+ Docker Containers have low impact on Host OS, Very fast, minimal resouce-usage, almost no duplication of shareable resources
+ Docker Containers are better in terms of Sharing, distribution and rebuilding -- via images and config files

***
# Docker Installation

