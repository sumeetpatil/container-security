# Container Security

## Docker image with root user
   Do not create docker images with a root user as it gives full privilages to a user who has access to the docker container.
   
#### Attack
Example : We can run a DOS(Denial of service) attack once we have full access(root privilages) to the container.
1. Run a ubuntu docker container - `docker run --rm -it ubuntu sh` 
2. Run this command to see who is the user - `whoami`. This returns `root` as the user. This is bad.
3. Now you can install/run whatever you want as part of the container.
4. Lets install `apt-get update && apt-get install stress -y`
5. Stress is a tool by which you can generate huge load on a system
6. Run `stress --vm 2 --vm-bytes 2G --timeout 30` 
   ```
   –vm this flag helps us in creating workers for the job. More workers, more stress.
   –vm-bytes this flag is used to set the amount of memory.
   –timeout flag directs the stress tool to stop the job(s) after X seconds. Here it is 30 seconds.
   ```
   When this command is executed the docker container (VM as well) will become slow/unresponsive for 30 seconds. This means that any application running on the container will be unresponsive for long.
   
#### Prevention
1. Build images with a non root user
2. Create a file with name `Dockerfile` with the following content -
   ```
    ##########################################
   # Dockerfile to change from root to 
   # non-root privilege
   ###########################################
   # Base image is Ubuntu
   FROM ubuntu:latest
   # Add a new user "john" with user id 8877
   RUN useradd -u 8877 john
   # Change to non-root privilege
   USER john
   ```
 3. Run `docker build . -t ubuntu:test`. With this command, the we create a docker image - ubuntu:test
 4. Run the docker container - `docker run --rm -it ubuntu:test`. Here `whoami` will give `john`
 5. With this continer you cannot install any new libraries. Test with `apt-get intall stress` which gives `Permission denied` error
 6. You cannot be a root with such images which makes your containers secure
