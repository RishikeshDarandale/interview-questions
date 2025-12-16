# Beginner Level

<details>
  <summary>What is Docker? Explain its core purpose and how it differs from traditional virtualization.</summary>
Docker is an open-source platform that enables developers to build, ship, and run applications in lightweight, portable containers. Its core technology is based on Linux containerization features like namespaces and cgroups.

#### Core Purpose

The core purpose of Docker is to solve the problem of "it works on my machine."
Docker packages an application and all its dependencies—libraries, system tools, configuration files, and runtime—into a standardized unit called a container. This ensures that the application will run quickly and reliably regardless of the computing environment (from a developer's laptop to a testing environment to a production server).
In short, Docker provides consistency and isolation for applications.

#### How Docker Works (Containerization)

The key components of the Docker platform are:
- Docker Engine: The client-server application that runs the containerization process. It includes the server (daemon), a REST API for communication, and the command-line interface (CLI).
- Docker Image: A read-only template containing the application code, dependencies, libraries, and configuration needed to run the application. Images are built from a simple text file called a Dockerfile.
- Docker Container: A runnable instance of a Docker Image. It represents the application running in isolation from the host and other containers.

#### Docker vs. Traditional Virtualization

The fundamental difference between Docker containers and traditional Virtual Machines (VMs) lies in the presence of the Guest Operating System (OS) and the way they share the host resources.

|Feature	| Docker Containers	| Traditional Virtual Machines (VMs)|
|---------|-------------------|-----------------------------------|
|Isolation Level	| Process-level isolation.	| Hardware-level emulation/isolation.|
|Operating System	| Shares the host OS kernel.	| Requires a full Guest OS (e.g., Windows, Linux) for every VM.|
|Resource Overhead |	Very low (Only application code and shared libraries are needed). |	High (Requires resources for the Guest OS, like CPU, memory, and disk).|
|Startup Time	| Seconds (Often milliseconds).	| Minutes (Must boot an entire OS).|
|Size	| Typically tens or hundreds of MBs. |	Typically several GBs.|
|Hypervisor/Engine	 | Docker Engine (utilizes the host OS kernel). |	Hypervisor (e.g., VMware, Hyper-V) manages virtual hardware.|

#### Key Differentiator: The Operating System
- VMs: Use a Hypervisor to virtualize the hardware, allowing multiple, independent Guest OSes to run simultaneously. Each VM is a complete, isolated server.
- Containers: Use the Docker Engine to virtualize the OS layer. All containers running on a host share the same host OS kernel. This eliminates the overhead of running a separate OS for every application, making containers incredibly lightweight and fast.

</details>
