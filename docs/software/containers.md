# Containers
[Containers](https://www.docker.com/resources/what-container/) provide a convenient and portable way to package and run applications in a completely isolated and self-contained environment, making it easy to manage dependencies and ensure complete reproducibility and portability. Compared to [conda environments](conda.md) or [software modules](modules.md) containers are always based on a base operating system image, usually Linux, ensuring that even the operating system is under control. Once a container is built and working as intended, it will run exactly the same forever, whereever, and is therefore the best way to bundle and distribute production-level workflows. By containerizing the application platform and its dependencies, differences in OS distributions and underlying infrastructure are abstracted away completely. Linux containers allow users to:

 - Use software with complicated dependencies and environment requirements
 - Run an application container from the Sylabs Container Library, Docker Hub, or from self-made images from the GitHub container registry
 - Use a package manager (like apt or yum) to install software without changing anything on the host system or require elevated privileges
 - Run an application that was built for a different distribution of Linux than the host OS
 - Run the latest released software built for newer Linux OS versions than that present on HPC systems
 - Archive an analysis for long-term reproducibility and/or publication

## Singularity/Apptainer
[Singularity/Apptainer](https://apptainer.org/docs/user/main/index.html) is a tool for running software containers on HPC systems, but is made specifically with scientific computing in mind. Singularity allows running Docker and any other OCI-based container natively and is a replacement for Docker on HPC systems. Singularity has a few extra advantages:

 - Security: a user in the container is the same user with the same privileges/permissions as the one running the container, so no privilege escalation is possible
 - Ease of deployment: no daemon running as root on each node, a container is simply an executable
 - Ability to run workflows that require MPI and GPU support

### Building container images
Building container images requires the user to have root/admin privileges. Currently there is no build environment on BioCloud, but alternatives for building containers are:

 - Using your own system (laptop/workstation) where you have root/elevated privileges to install Singularity or Docker and build containers, then transfer the container image file(s) to the BioCloud or publish it to a public container registry
 - Use a free cloud container build service like [https://cloud.sylabs.io](https://cloud.sylabs.io) or [https://hub.docker.com/](https://hub.docker.com/)
 - Publish a `Dockerfile` to a GitHub repository and use GitHub actions to build and publish the container to the GitHub container registry

### Pre-built container images
Usually it's not needed to build a container yourself unless you want to customize things in detail, since there are plenty of pre-built images already available that work straight of the box. For bioinformatic software the community-driven project [biocontainers.pro](https://biocontainers.pro/) should have anything you need, and if not - you can contribute! If you need a container with multiple tools installed see [multi-package containers](https://biocontainers.pro/multipackage).

### Running a container
```
# pull a container
$ apptainer pull ubuntu_22.04.sif docker://ubuntu:22.04

# run a container with default options
$ apptainer run ubuntu_22.04.sif

# start an interactive shell within a container
$ apptainer shell ubuntu_22.04.sif
```

For additional guidance use the Apptainer usage guide [here](https://apptainer.org/docs/user/main/index.html).

## Docker containers
Docker is not supported directly for non-admin users due to security and compatibility issues with our user authentication mechanism, but you can instead just run them through apptainer.
