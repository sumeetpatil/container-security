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

## Privileged Mode
   Do not run the containers in privilage mode. It gives full access to the host machine.

#### Attack
1. Run `docker run --privileged -it --rm ubuntu sh`
2. As the continer is running in privileged mode. It disables all the security measures.
3. Run command `mount` which gives all the mount points
4. Now you can mount any of these mounts from the host system
5. Run `mkdir /hostroot && mount /dev/vda1 hostroot`
6. This mount the host machines data to your container

#### Prevention
1. Avoid running the containers in privilage mode


## Socket Misconfigurations
Sometimes people just mount the docker socket inside the container. If an attacker gains access to this container, he literally has root access.

#### Attack
1. Mount socket `docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu:latest sh`
2. Run `apt-get update ; apt-get install docker.io -y` to install docker client
3. `docker --version` to check if docker is running
4. `docker run -it --rm centos sh`
5. With this you can run containers in host machine
6. As we can its very dangerous to mount the docker socket inside any container. The centos container can do anything that the host system can do as root.


#### Prevention
1. Try to avoid docker socket. 

## Misconfig in Docker HTTP REST API
By default unix uses docker socket. You can expose it via HTTP as well. People do not care security of this HTTP REST API.

#### Attack
1. Expose docker via REST API
2. `sudo cp /lib/systemd/system/docker.service /etc/systemd/system/`
3. `sudo vi /etc/systemd/system/docker.service`
4. Add `-H tcp://0.0.0.0:2376` at the end of ExecStart
5. `sudo systemctl daemon-reload` reload new config
6. `sudo systemctl restart docker.service` restart docker
7. `curl http://localhost:2376/version` -> gives the docker version
8. `sudo apt-get install jq -y` this is a json formatter
9. `docker run --rm -it -d ubunut` run an ubuntu instance
10. `http://localhost:2376/containers/json | jq` gives list of containers(similar to `docker ps`)
11. Stop container `http://localhost:2376/containers/<container-name>/stop`
12. Here the API is unauthenticaed
13. API is Non-SSL

This gives attacer a clear access.

#### Prevention
1. Run an apache2 or nginx as a proxy
2. Configure SSL, Authentication in nginx/apache2
3. Config for apache 2 - 
4. Run these commands
   ```
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2enmod proxy_balancer
   sudo a2enmod lbmethod_byrequests
   sudo systemctl restart apache2
   sudo vi /etc/apache2/sites-available/000-default.conf
   
   Add this content -
   <VirtualHost *:80>
    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:2376/
    ProxyPassReverse / http://127.0.0.1:2376/
   </VirtualHost>
   
   sudo systemctl restart apache2
   ```
5. With these config you are able to proxy your load to port 80. 
   `curl http://localhost/version` should give you the docker version
6. Adding authentication - 
   ```
   sudo apt install apache2-utils
   sudo htpasswd -c /etc/apache2/.htpasswd sammy
   Enter your password here
   
   Check if password exists 
   cat /etc/apache2/.htpasswd
   
   vi /etc/apache2/sites-available/000-default.conf
   
   Add this content -
   
   <Location />
      AuthType Basic
      AuthName "Restricted Content"
      AuthUserFile /etc/apache2/.htpasswd
      Require valid-user
   </Location>
   
   Run - sudo systemctl restart apache2
   
   ```
   
7. Now access `curl http://localhost/version` this should prompt you username/password



