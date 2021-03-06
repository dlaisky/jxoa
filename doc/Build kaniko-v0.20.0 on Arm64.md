# Create kaniko container image

This document describes how to build kaniko-v0.20.0 on Arm64  from  the following link: https://github.com/GoogleContainerTools/kaniko

# Environment

The build is run natively on Arm64 machines. The server used is :   

- Memory : 32 G  
- CPU: 32 cores  

* kubectl version: 

  Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-15T16:58:53Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/arm64"}

  Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.13", GitCommit:"4c00c3c459261e8ff3381c1070ddf798f0131956", GitTreeState:"clean", BuildDate:"2020-12-09T11:09:27Z", GoVersion:"go1.13.15", Compiler:"gc", Platform:"linux/arm64"}

* OS: Linux version 4.15.0-128-generic (buildd@bos02-arm64-005) (gcc version 7.5.0 (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04))

* The kubernetes cluster has one master and two workers.

# Prerequisites

- install go  version>=1.13.8

  For arm64, follow this link to download Go 1.13.8

  https://golang.google.cn/dl/go1.13.8.linux-arm64.tar.gz

  Download the archive and extract it.

- install Pre-commit

  I used pip. Other installation methods exist as well.

  `$ pip install pre-commit`

- install Dep

  follow this link to install:

  https://github.com/golang/dep

- update go proxy and environment

  Optionally, set up GOPROXY suitable to your development environment.

  `$ go env -w GO111MODULE=on`
  `$ go env -w GOPROXY=https://goproxy.io,direct`

- set GOPATH

  `$ export GOPATH=your file`

# Build kaniko

```shell
$ git clone https://github.com/GoogleContainerTools/kaniko.git
$ cd kaniko/cmd/executor/
$ mkae -f ../../Makefile
GOARCH=amd64 GOOS=linux CGO_ENABLED=0 go build -ldflags '-extldflags "-static" -X github.com/GoogleContainerTools/kaniko/pkg/version.version=v1.0.0 -w -s  ' -o out/executor github.com/GoogleContainerTools/kaniko/cmd/executor
...
```

if successfully, the folder `out` will be generated, the binary file has been built in folder `out`. You can see it .

```shell
$ ls out -l
total 49532
-rwxr-xr-x 1 root root 50720768 Sep 22 11:25 executor
```

Next, building container image with the file executor.

# Build images 

The kaniko Dockerfile

```dockerfile
FROM ubuntu:18.04
RUN mkdir /kaniko
COPY executor /kaniko/executor
ENV HOME /root
ENV USER /root
ENV PATH /usr/local/bin:/kaniko
ENV DOCKER_CONFIG /kaniko/.docker/
WORKDIR /workspace
ENTRYPOINT ["/kaniko/executor"]
```

`docker build -t kaniko:arm64 .`

The kaniko is only used to build container image, you can from the url to test it.

https://github.com/GoogleContainerTools/kaniko