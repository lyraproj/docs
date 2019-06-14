---
title: Installing Lyra
layout: default
category: Setting up Lyra
order: 1
---

# Installing Lyra

On Linux and MacOS you can download Lyra and build it with Go. Lyra is also available as a Homebrew package, or as a Docker container.   

## Run Lyra in a Docker container

To run Lyra in a Docker container, make sure you've [installed the latest stable version of Docker](https://docs.docker.com/install/).

To set up the Lyra Docker container:

1. Create a `lyra-local` directory to save your work locally: 

   ```
   cd
   mkdir lyra-local
   ```

2. Pull the latest Lyra container:

   ```
   docker pull lyraproj/lyra:latest
   ```

3. Run the container in interactive mode and mount the directory at `/src/lyra/local` to your `local-lyra` directory:

   ```
   docker run -it \
   --mount type=bind,src=$HOME/lyra-local,dst=/src/lyra/local \
   lyraproj/lyra:latest /bin/ash
   ```

## Install Lyra with Homebrew

Use the following command to install Lyra with Homebrew:

```
brew install lyraproj/lyra/lyra
```

## Build Lyra with Go

Lyra requires [Go](https://golang.org/doc/install) 1.11 or higher, and [go modules](https://github.com/golang/go/wiki/Modules) enabled.

To install and Build Lyra, follow these instructions:
1. Clone the Lyra repository: 

   ```
   git clone https://github.com/lyraproj/lyra
   ```

2. Build Lyra: 

   ```
   cd lyra; make
   ```

3. (Optional) If you intend to work with typescript, run `make smoke-test-ts`. This checks for an appropriate version of Node.js

> **Note:** You must run the `make` command each time you merge changes from the upstream Lyra repository.