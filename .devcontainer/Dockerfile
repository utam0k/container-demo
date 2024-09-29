FROM ubuntu:22.04

RUN apt update 
RUN apt upgrade -y
RUN apt install -y curl gnupg
RUN echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
RUN curl -L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key" | apt-key add -
RUN apt install -y podman sudo bpftrace

RUN useradd --uid 1000 --create-home --shell /bin/bash -G sudo,root ubuntu
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER $USERNAME
WORKDIR /home/$USERNAME/
