---
layout: post
title: "How to checksum and install source tarballs in Docker"
date: 2014-04-22 16:52
categories: [docker, linux, \*nix, devops, security, checksum]
comments: true
published: true
author: Byron Gibson
---
If you're writing a Dockefile that downloads, builds, and installs a source package, say 
node.js, nginx, haskell, whatever, you'll most likely want your Dockerfile to checksum that 
package and exit on fail.  

Docker doesn't have a built-in way to do this, and it [doesn't appear as if it is going 
to get one any time soon][1], so here's how to script it manually:

````
# Docker official ubuntu 12.04 LTS
FROM ubuntu:12.04

# SETUP
RUN cd /tmp
RUN apt-get update -y
RUN apt-get install wget build-essential automake -y
RUN wget http://nodejs.org/dist/latest/node-v0.10.26.tar.gz
RUN wget http://nodejs.org/dist/latest/SHASUMS256.txt

# CHECKSUM
# RUN checksum: continue on success, exit Docker build on fail.  Requires some 
# script-fu to reduce output down to a single 0 or non-zero POSIX exit code.  
# Details in footnote [1].  TLDR: run checksum, merge sderr into sdout (2>$1) before 
# piping stout to grep, grep for checksum success code OK, suppress all output except 
# exit codes with -qs flags on grep.  Returns 0 exit code if checksum succeeds, 1 if
# not.  Docker breaks and exits on non-zero exit code.
RUN sha256sum -c SHASUM256.txt 2>&1|grep -qs OK

# INSTALL
RUN tar -xvf node-v0.10.26.tar.gz && cd node-v0.10.26
RUN ./configure && make && make install

# CLEANUP
RUN apt-get purge wget build-essential automake -y && apt-get autoremove -y
````

Docker automatically exits Dockerfile build if any RUN command returns a non-zero 
(error) exit code.  

The problem here is that node.js's SHASUMS256.txt contains many lines
for lots of different files - source, binaries for different platforms, etc. - only 
one of which will succeed at this checksum and return 0 exit code.  All other lines 
will fail and be sent to sderr.  sterr is not included by default in pipes, only sdout, 
causing this command to return a jumble error data prior to the pipe.  Docker then 
interprets this as non-zero error codes and exits, even if the checksum succeeds.  

The solution is to manually merge sderr into stout, then pipe to grep, grep for the OK 
(case sensitive), and suppress all output except exit codes with -qs.  If OK is 
is present in the combined output, this line will return a 0 exit code.  If not, 
will return 1 and the Dockerfile build will break and exit.  

You can test this process manually by running a Docker container in interactive mode:

````
>sudo docker run -i -t ubuntu:12.04 bin/bash
````

Run the SETUP and CHECKSUM parts, then then check the CHECKSUM's exit code with `echo $?`.


[1]:https://github.com/dotcloud/docker/issues/2579
