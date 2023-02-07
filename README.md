# Container Security

1. Docker image with root user
   Do not create docker images with a root user as it gives full privilages to a user who has access to the docker container.
   
   Attack:
     Example : We can run a DOS(Denial of service) attack once we have access to the container.
     a. Run a ubuntu docker container - `docker run --rm -it ubuntu sh` 
     b. Run this command to see who is the user - `whoami`. This returns `root` as the user. This is bad.
     c. Now you can install, run whatever you want as part of the container.
   
