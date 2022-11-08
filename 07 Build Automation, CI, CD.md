**Installing Jenkins**

- You need a dedicated server for jenkins, jenkins needs atleast 1GB of RAM but recommended is 4GB
- Expose port 22 for your IP and 8080 for all sources
- `ssh root@[sever-ip]`
- `apt update`
- `apt install docker.io`
- `docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts` = 50000 port is where is communication between master and worker nodes is happening
- `docker exec -it [container-id] bash`
- `cat /var/jenkins_home/secrets/initialAdminPassword` = to get the initial admin password
- open jenkins on browser
- create a admin user

**Installing Build Tools**

2 ways to install tools.

1. Jenkins plugin. eg, for Maven; Manage Jenkins>Global Tool Configuration>Maven>Add Maven. If the build tools not in global tool configuration plugin>manager>available>[plugin-name]>install.
1. Install tools directly on server. eg,
   - `docker exec -u 0 -it [jekins-container-id] bash`
   - `apt update`
   - `curl` = to check if curl is install
   - `apt install curl` = install curl
   - `curl -sL http://deb.nodesource.com/setup_10.x -o nodesource_setup.sh`
   - `bash nodesource_setup.sh`
   - `apt install nodejs`

**Jenkins Basics**

Running a Simple Job

- New Item>Freestyle projects = to create a simple job
- Add a build step
  ```
  npm --version
  ```
  for maven commands select invoke top-level maven target, select maven plugin
  ```
  --version
  ```
- Jenkins main view to see list of jobs
- Click on the job and build now to run the job

Configure Git Repository

- Select Job>Configure>Source Code Management

Running Commands

- To execute file in repo, inside script

  ```sh
  # set permission
  chmod +x freestyle-build.sh

  # execute the script
  ./freestyle-build.sh
  ```

- Similarily you can run test, package commands, the workspace directory will be inside docker container in `/var/jenkins_home/workspace/[job-name]`

**Docker in Jenkins**

- `docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker jenkins:lts` = mount docker directories to the container as a volume, this will make docker available
- `docker exec -u 0 -it [jekins-container-id] bash`
- `chmod 666 /var/run/docker.sock` = 666 read/write permission for everybody including the jenkins user
- You can now execute docker commands in the execute shell
  ```sh
  docker build -t my-app:1.0 .
  ```
- Create crendials = Manage Jenkins>Manage Credientials>Store>Add Credentials. Job>configure>environment>check use secret text(s) or files(s)>username and password(seperated)>enter username and password vairable
  ```sh
  docker build -t [repository-host]/[image-name]:[version] .
  echo $PASSWORD | docker login -u $USERNAME -password-stdin [repository-ip]:[repository-port](optional for dockerhub)
  docker push [repository-host]/[image-name]:[version]
  ```
- To push on nexus
  - Edit the docker to allow HTTP communication
    - `vim /etc/docker/daemon.json`
      ```json
      {
        "insecure-registries": ["[nexus-ip]:[repository-connector-port]"]
      }
      ```
    - `systemctl restart docker`
    - `docker start [jenkins-container-id]`
    - `docker exec -u 0 -it [jekins-container-id] bash`
    - `chmod 666 /var/run/docker.sock
  - Create crendials = Manage Jenkins>Manage Credientials>Store>Add Credentials. Job>configure>environment>check use secret text(s) or files(s)>username and password(seperated)>enter username and password vairable
  - Push to nexus
    ```sh
    docker build -t [nexus-ip]:[nexus-port]/[image-name]:[version] .
    echo $PASSWORD | docker login -u $USERNAME -password-stdin [nexus-ip]:[nexus-port]
    `docker push [nexus-ip]:[nexus-port]/[image-name]:[image-tag]`
    ```
