# docker-intro

# A short introduction to using Docker for Bioinformatics

## Prerequisites

The following sessions assumes that you have Docker running on your computer (we tested with Docker version 18.09.7, build 2d0083d).

If you're running on a Linux host it further assumes that you have it set up in such a way that you don't have to prepend `sudo` to the `docker` command each time. There is a number of ways you can do that, e.g. by creating a docker group and adding your user to it or by running the Docker daemon as a non-root user (find some more info/instructions <a href="https://docs.docker.com/engine/install/linux-postinstall/" title="Post-installation steps for Linux (last accessed 24.04.2020)" target="_blank">here</a>). If for some reason you don't want to or can do any of these things, all of the below should also work if you simply prepend `sudo` to the `docker` command.

## Hands-on introduction to Docker

Let's get cracking, with:
- [Part 1](https://github.com/chrishah/docker-intro/tree/master/part-1/docker-intro-philipp.md): Running and interacting with prebuild images
- [Part 2](https://github.com/chrishah/docker-intro/tree/master/part-2/build-and-manage.md): Build and manage your own images

