# Create docker image

https://docs.docker.com/build/building/packaging/

## Dockerfile

Docker builds images by reading the instructions from a Dockerfile. This is a text file containing instructions that adhere to a specific format needed to assemble your application into a container image and for which you can find its specification reference in the Dockerfile reference.

## Instructions

RUN and ENTRYPOINT are two different ways to execute a script.

RUN means it creates an intermediate container, runs the script and freeze the new state of that container in a new intermediate image. The script won't be run after that: your final image is supposed to reflect the result of that script.

ENTRYPOINT means your image (which has not executed the script yet) will create a container, and runs that script.

In both cases, the script needs to be added, and a RUN chmod +x /bootstrap.sh is a good idea.