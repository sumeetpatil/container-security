# Container Security

## Docker image with root user
   Do not create docker images with a root user as it gives full privilages to a user who has access to the docker container.
   
#### Attack
Example : We can run a DOS(Denial of service) attack once we have access to the container.
1. Run a ubuntu docker container - `docker run --rm -it ubuntu sh` 
2. Run this command to see who is the user - `whoami`. This returns `root` as the user. This is bad.
3. Now you can install, run whatever you want as part of the container.
   
