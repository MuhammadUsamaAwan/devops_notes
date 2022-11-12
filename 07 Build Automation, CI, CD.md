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

**Creating a Pipeline Job**

- Connect to git repository = New Item>Pipeline>Pipeline>Pipeline script from SCM
- Job will read the Jenkinfile and execute the steps
- You can see jenkins enivornment variables from `https://[jekins-ip]:[jenkins-port]/env-vars.html`

**Jenkinfile Syntax**

```groovy
// make external groovy script global
def gv

// indicates that we are writing pipeline
pipeline {
  // where to execute
  agent any
  // enivornment
  enivornment {
    NEW_VERSION = '1.3.0'
  }
  tools {
    maven 'Maven' // Maven is the name of the Maven installation in the Global Tool configuration
  }
  parameters {
    string(name: 'VERSION', defaultValue: '', description: 'version to deploy on prod')
    choice(name: 'VERSION', choices: ['1.1.0', '1.2.0'], description: '')
    booleonParam(name: 'executeTests', defaultValue: true, description: '')
  }
  // where the work happens
  stages {
    stage("init") {
      steps {
        // load external groovy script
        script {
          gv = load 'script.groovy'
        }
      }
    }
    stage("build") {
      steps {
        // double quotes must to use variable
        echo "building version ${NEW_VERSION}"
        sh 'mvn install'
      }
    }
    stage("test") {
      when {
        // conditionals
        experession {
          (BRANCH_NAME == 'dev' || BRANCH_NAME == 'master') && params.executeTests
        }
      }
      steps {
        script {
          // you can write groovy here
          gv.buildApp()
        }
      }
    }
    stage("deploy") {
      // user input
      input {
        message 'Select environment'
        ok 'Done'
        parameters {
          choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: '')
        }
      }
      // another way directly assign input to a environment variable
      script {
        env.ENV = input message: 'Select environment', ok: 'Done', parameters: [choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: '')]
      }
      steps {
        echo "deploying to ${ENV}"
        // crendials
        withCrendials([usernamePassword(crendials: 'docker-hub-repo', usernameVariable: 'USER' ,passwordVariable: 'PWD')]) {
          // use the variables her
          // you need have crendials and crendials binding plugins in jenkins
        }
      }
    }
  }
  // executed after steps
  post {
    always {
    }
    success {
    }
    failure {
    }
  }
}
```

**Pipeline Job Example**

```groovy
// Jenkinfile

def gv

pipeline {
  agent any
  tools {
    maven 'Maven'
  }
  stage("init") {
      steps {
        script {
          gv = load 'script.groovy'
        }
      }
  }
  stages {
    stage("build jar") {
      steps {
        script {
          gv.buildJar()
        }
      }
    }
    stage("build image") {
      steps {
        script {
          gv.buildImage()
        }
      }
    }
    stage("test") {
  }
}
```

```groovy
// script.groovy

def buildJar() {
  echo "building the application..."
  sh "mvn package"
}

def buildImage() {
  echo "building the docker image..."
  withCrendials([usernamePassword(crendials: 'docker-hub-repo', usernameVariable: 'USER' ,passwordVariable: 'PWD')]) {
    sh 'docker build -t [registry-host]/[image-name]:[image-tag] .'
    sh "echo $PWD | docker login -u $USER --password-stdin"
    sh 'docker push [registry-host]/[image-name]:[image-tag]'
  }
}
```

**Mutibranch Pipeline**

- New Item>Multibranch pipeline = create a multibranch pipeline
- Branch Sources>Add sources = to add a repository
- Behaviours>Add>Filter by name(regular expression)>.\* = add behaviour for all branches, it is better to define the rest of the configuration in the Jenkinsfile
- Add branch-based login in Jenkinsfile

  ```groovy
  pipeline {
    agent none
    stages {
        stage("test") {
            steps {
                script {
                }
            }
        }
        stage("build") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }****
            steps {
                script {
                }
            }
        }
        stage("deploy") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                }
            }
        }
    }
  }
  ```

**Credentials in Jenkins**

- Credientials Scope
  1. System = only available on jenkins server, not for jenkins jobs
  1. Global = accessible everywhere
  1. Project = available in the job section, limited to project, only for multibranch pipeline

**Jenkins Shared Library**

- Extension of pipeline, has its own repository, written in groovy
- Shared library folder structure
  ```sh
  var # folder containing all the functions that we can call from Jenkinsfile, eachh function/execution step is its own groovy file
  src # helper code
  resources # use external libraries for non-groovy file
  ```
- write groovy script

  ```groovy
  // var/buildJar.groovy

  def call() {
    echo "building the application..."
    sh "mvn clean package"
  }
  ```

  ```groovy
  // var/buildImage.groovy

  def call(String imageName) {
    echo "building the docker image..."
    withCredentials([usernamePassword(credentialsId: 'docker-hub-rep', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        sh "docker build -t $imageName ."
        sh "echo $PASS | docker login -u $USER --password-stdin"
        sh "docker push $imageName"
    }
  }
  ```

- Manage jenkins>Configure System>Global Pipeline Libraries =>
  make jenkins shared library globally available
- Use shared library in a Jenkinsfile

  ```groovy
  @Library('name-defined-in-global-library') // use underscore at the end if not using external scripts
  @Library('name-defined-in-global-library@2.0') // this will overwrite the version set in global

  state("build jar") {
    steps {
        script {
            buildJar()
        }
    }
  }
  state("build image") {
    steps {
        script {
            buildImage '[repository-host]/[image-name]:[image-tag]'
        }
    }
  }
  ```

- Extract logic into groovy classes

  ```groovy
  // src/com.example/Docker.groovy

  package com.example

  class Docker implements Serializable {
    Docker(script) {
        this.script = script
    }

    def dockerBuild(String imageName) {
        script.echo "building the docker image..."
        script.sh "docker build -t $imageName ."
    }

    def dockerLogin() {
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-rep', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        script.sh "docker build -t $imageName ."
        script.sh "echo $script.PASS | docker login -u $script.USER --password-stdin"
      }
    }

    def dockerPush(String imageName) {
        script.sh "docker push $imageName"
    }
  }
  ```

  ```groovy
  // var/dockerLogin.groovy

  import com.example.Docker

  def call() {
    return new Docker(this).dockerLogin()
  }
  ```

  ```groovy
  // var/dockerPush.groovy

  import com.example.Docker

  def call(String imageName) {
    return new Docker(this).dockerPush()
  }
  ```

  ```groovy
  // var/dockerBuild.groovy

  import com.example.Docker

  def call(String imageName) {
    return new Docker(this).dockerBuild(imageName)
  }
  ```

  ```groovy
  // Jenkinsfile

  state("build image") {
    steps {
        script {
            dockerBuild '[repository-host]/[image-name]:[image-tag]'
            dockerLogin()
            dockerPush '[repository-host]/[image-name]:[image-tag]'
        }
    }
  }
  ```

- Project scoped shared library = reference the library directly in Jenkinsfile
  ```groovy
  library identifier: 'jenkins-shared-library@master', modernSCM([$class: 'GitSCMSource', remote: 'repository-url', credentialsId: 'gitlab-credentials'])
  ```

**Trigger Jenkins Job - Webhooks**

- Manage Jenkins>Manage Plugins => Install GitLab
- Get the GitLab API Token => GitLab>Preferences>Access Token
- GitLab>Settings>Integrations>Jenkins CI => enable integration, check trigger push, paste Jenkins URL, project name is the job name in Jenkins, username and password of the Jenkins
- Manage Jenkins>Configure System>GitLab
  - Connection name = your connection name
  - GitLab host URL = the URL to GitLab server e.g, https://gitlab.com/
  - Add credentials > GitLab API Token
- Pipeline>Configure>General>GitLab Connection => make sure build triggers, build when a change is pushed to GitLab, push events &s opened merge request events
- For multibranch pipeline
  - Manage Jenkins>Manage Plugins => Install Multibranch Scan Webhook Trigger
  - Multibranch pipeline>Configure>Scan Multibranch Pipeline Triggers>Scan by webhook => enter trigger token
  - GitLab>Settings>Webhooks => configure GitLab

**Versioning the Application**

- Increment Maven Version
  ```
  mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVerion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalMajorVerion} versions:commit
  ```
- In JenkinsFile

  ```groovy
  stage("increment version") {
    steps {
        script {
            sh "mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVerion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalMajorVerion} versions:commit"
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
            env.IMAGE_VERSION = "$version-$BUILD_NUMBER"
        }
    }
  }
  stage("build image") {
    steps {
        script {
            dockerBuild "[repository-host]/[image-name]:$IMAGE_VERSION"
        }
    }
  }
  stage("commit version update") {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                sh 'git config --global user.email "jenkins@example.com"'
                sh 'git config --global user.name "jenkins"' // better to ssh to jenkin server and set the configuration
                sh "git remote set-url origin https://$${USER}:${PASS}@[git-repository-url]"
                sh 'git add .'
                sh 'git commit -m "ci: version bump"'
                sh 'git push origin HEAD:jenkins-job'
            }
        }
    }
  }
  ```

- Manage Jenkins>Manage Plugins => Install Ignore Committer Strategy, Pipeline>Configure>Branch Sources>Git> Build Strategy>Ignore Committer Strategy => enter jenkins email and check the allow builds for other authors
- Modify the Dockerfile

  ```Dockerfile
  COPY ./target/java-maven-app-*.jar /user/app/

  CMD java -jar java-maven-app-*.jar
  ```
