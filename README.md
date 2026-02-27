# eXpOS Docker Setup

This guide explains how to set up eXpOS safely using Docker, without installing legacy libraries on your host system, while enabling a bidirectional sync between the host machine and the Docker container (useful for version control and backups).

## Folder Structure

```bash
OS_Lab/
├── Dockerfile
└── expos_container/      # MUST be empty initially
```

## Dockerfile

```Dockerfile
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

# Install required packages
RUN apt-get update && apt-get install -y \
    gcc make flex bison \
    libreadline-dev libc6-dev libfl-dev \
    wget curl unzip \
    git vim build-essential \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m expos
USER expos

# Set working directory to user's home
WORKDIR /home/expos
```

## Build Image

From inside `OS_Lab/` directory, run the following command

```bash
docker build -t expos:ubuntu20.04 .
```

## Run the Container

```bash
docker run -it \
  -v $PWD/expos_container:/home/expos \
  --name expos \
  expos:ubuntu20.04
```

- Mounts `expos_container` -> `/home/expos`
- Changes inside the container are reflected on the host system

## Download & Build eXpOS (INSIDE CONTAINER)

Once inside the container shell:

```bash
curl -sSf https://raw.githubusercontent.com/eXpOSNitc/expos-bootstrap/main/download.sh | sh
```

This will create `/home/expos/myexpos` inside your container. You will be able to view the changes in your local machine as well under `OS_Lab/expos_container/myexpos`.

Inside container shell, enter the following commands:

```bash
cd myexpos
make
```

If successful, you should see the following folders inside `myexpos` directory:

```bash
- spl/
- expl/
- xfs-interface/
- xsm/
```

**eXpOS installation is now complete. You may continue follownig everything from [eXpOS Roadmap](https://exposnitc.github.io/Roadmap.html$0) starting from Stage 2**.

## Additional Helpful Notes

- The `expos_container/` folder on your host machine will contain all the files related to eXpOS. You can use this folder for version control (e.g., git) or backups.
- The Docker container provides an isolated environment, so you won't have to worry about installing legacy libraries on your host system. All necessary dependencies are contained within the Docker image.
- If you want to stop working on eXpOS, simply exit the container. Your work will be preserved in the `expos_container/` folder, and you can restart the container later to continue where you left off.
- Modify `.gitignore` in `expos_container/` to your needs. For example, you might want to ignore build artifacts while keeping source files under version control. 

## Helper Scripts

- For the convenience of users, I've created helper scripts in the `myexpos/scripts` directory to ease development process. Currently this repo has `compile_spl` and `runxfs` scripts.
- Run `chmod +x $HOME/myexpos/scripts/*` to make the scripts executable.
- Copy `.bashrc` from `root` directory to `/home/expos/.bashrc` to add the helper scripts to your PATH.
- You can now start using `compile_spl` and `runxfs` commands from anywhere inside the container to compile SPL files and load batch files with xfs-interface respectively.


## Useful Commands

### Exit the Container

```bash
exit
```

This stops the container but does not delete anything.

### Restarting the Container Later

If you exit the container, restart it using:

```bash
docker start expos
docker exec -it expos /bin/bash
```

All your work remains preserved inside `expos_container/`.

### Clean Installation (Deletes eXpOS Files)

If you want a completely fresh eXpOS setup, you must manually delete the container as well as the bind-mounted folder:

```bash
# Stops and deletes the container
docker rm -f expos
# Deletes the bind-mounted folder
rm -rf expos_container/
```

This permanently deletes all eXpOS files. You will need to repeat the setup from scratch.
