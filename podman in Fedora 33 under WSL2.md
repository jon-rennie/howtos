## Using podman with Fedora 33 under WSL 

This howto describes how to install and use podman, buildah and skopeo under Fedora 33 as a distribution in Windows Subsystem for Linux (WSL) version 2

Based on [article by Jonathan Bowman](https://dev.to/bowmanjd/using-podman-on-windows-subsystem-for-linux-wsl-58ji)

Starting point is to install Fedora 33 as a WSL distro as described in [this howto](https://github.com/jon-rennie/howtos/blob/main/Install%20Fedora%2033%20for%20WSL%202.md)


### 01 - install podman, skopeo and buildah podman in Fedora 33 under WSL2
    [jon@jon-lenovo-m700 system32]$ sudo dnf install podman skopeo buildah which
    [jon@jon-lenovo-m700 ~]$ which buildah
    /usr/bin/buildah
    [jon@jon-lenovo-m700 ~]$ which skopeo
    /usr/bin/skopeo
    [jon@jon-lenovo-m700 ~]$ which podman
    /usr/bin/podman

### 02 - workarounds for lack of systemd
As WSL distros do not have or use systemd, there is no $XDG_RUNTIME_DIR available for podman to use for temporary files. We create this as needed by appending this to .bashrc:

    if [[ -z "$XDG_RUNTIME_DIR" ]]; then
        export XDG_RUNTIME_DIR=/run/user/$UID
        if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
            export XDG_RUNTIME_DIR=/tmp/$USER-runtime
            if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
            mkdir -m 0700 "$XDG_RUNTIME_DIR"
            fi
        fi
    fi

### 03 - add containers.conf file to /etc/containers 
Fedora version of podman does not create /etc/containers/containers.conf file. So we copy from /usr

    [jon@jon-lenovo-m700 ~]$ sudo cp /usr/share/containers/containers.conf /etc/containers/

### 04 - remove reliance on systemd and journald
As WSL distros do not have or use systemd or journald we need to edit /etc/containers/containers.conf file to remove reliance on them

    [jon@jon-lenovo-m700 ~]$ sudo vi /etc/containers/containers.conf
#### change these vars to these values:
1. cgroup_manager = "cgroupfs"
2. events_logger = "file"

### 05 - re-install shadow-utils
[jon@jon-lenovo-m700 ~]$ sudo dnf reinstall shadow-utils

### 06 - test it - use podman to pull down UBI8 image from Red Hat Registry and run it
    [jon@jon-lenovo-m700 ~]$ podman run -it ubi8
    Resolved short name "ubi8" to a recorded short-name alias (origin: /etc/containers/registries.conf.d/shortnames.conf)
    Trying to pull registry.access.redhat.com/ubi8:latest...
    Getting image source signatures
    Copying blob cca21acb641a done
    Copying blob d9e72d058dc5 done
    Copying config 3269c37eae done
    Writing manifest to image destination
    Storing signatures
    [root@26b5c93e1bed /]# uname -a
    Linux 26b5c93e1bed 4.19.128-microsoft-standard #1 SMP Tue Jun 23 12:58:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
    [root@26b5c93e1bed /]# cat /etc/redhat-release
    Red Hat Enterprise Linux release 8.3 (Ootpa)
    [root@26b5c93e1bed /]# exit

### 07 - test it - use skopeo to inspect a remote image 
    [jon@jon-lenovo-m700 ~]$ skopeo inspect docker://registry.access.redhat.com/ubi8/ubi-minimal
    {
        "Name": "registry.access.redhat.com/ubi8/ubi-minimal",
        "Digest": "sha256:4b9899b5c2906aae8e8fcd1012a5949e98bda68192c5e7bf6c1e171686c97d7a",
    ...

### 08 - test it - create a ContainerFile
Copy and paste this into a ContainerFile (example filename = ubi8-minimal-openjdk-containerfile)

    FROM registry.access.redhat.com/ubi8/ubi-minimal:latest
    LABEL maintainer Jon Rennie <jon@jonrennie.com>

    # set Java version
    ENV JAVA_MAJOR_VERSION=8

    # install OpenJDK8
    RUN microdnf install java-1.8.0-openjdk-headless --nodocs \
            openssl \
            curl \
            ca-certificates ;\
            microdnf clean all

    ENV JAVA_HOME /etc/alternatives/jre

    # Gives uid
    USER 1001

### 009 - test it - use buildah to build a container image using that ContainerFile
    [jon@jon-lenovo-m700 ~]$ buildah bud -t ubi8-minimal-openjdk8 -f ubi8-minimal-openjdk-containerfile
    ...

### 010 - test it - make sure the new image is showing up in localhost container image store
    [jon@jon-lenovo-m700 ~]$ buildah images
    REPOSITORY                                    TAG      IMAGE ID       CREATED              SIZE
    localhost/ubi8-minimal-openjdk8               latest   6d779d7da177   About a minute ago   263 MB

### 011 - test it - run this new image from localhost

    [jon@jon-lenovo-m700 ~]$ podman run -it localhost/ubi8-minimal-openjdk8 /bin/bash
    bash-4.4$ uname -a
    Linux 505726a9dd79 4.19.128-microsoft-standard #1 SMP Tue Jun 23 12:58:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
    bash-4.4$ java -version
    openjdk version "1.8.0_275"
    OpenJDK Runtime Environment (build 1.8.0_275-b01)
    OpenJDK 64-Bit Server VM (build 25.275-b01, mixed mode)
    bash-4.4$ echo $JAVA_MAJOR_VERSION
    8
