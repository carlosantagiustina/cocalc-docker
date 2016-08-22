# SageMathCloud Docker image

This is a self-contained single-image multi-user SageMathCloud server.

As is, this is ABSOLUTELY NOT SECURE in any way, shape, or form yet!  Do not trust it if you run it for some sort of production use.  For example, there is no rethikdb password set yet, so any user could do anything.

## Instructions

To store your local SMC data in the directory ~/smc, and run SageMathCloud (via docker), make sure you have about 7GB disk space free, then type:

     docker run -v ~/smc:/projects -P williamstein/sagemathcloud

Type `docker ps` to see what port were exposed, and connect to either the encrypted or non-encrypted ports.

    docker ps

which might output

    CONTAINER ID        IMAGE                        COMMAND                 CREATED             STATUS              PORTS                                           NAMES
    9eff7133bbd6        williamstein/sagemathcloud   "/bin/sh -c ./run.py"   3 minutes ago       Up 3 minutes        0.0.0.0:32779->80/tcp, 0.0.0.0:32778->443/tcp   evil_almeida

If the port is 32779 (as it is above), then visit http://localhost:32779/


## Issues

  - gp doesn't work at all, due to the Ubuntu ppa being broken


## Build

Build the image

    make build

Run the image (to test)

    make run

How I pushed this

    docker tag smc:latest williamstein/sagemathcloud
    docker login --username=williamstein
    docker push  williamstein/sagemathcloud
