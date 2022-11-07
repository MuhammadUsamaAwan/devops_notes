For Nexus you need sever with more memory, like 8GB.<br>

**Installing Nexus**

- You need to install java 8
- `cd /opt/`
- Download the Nexus tar file from [nexus download page](https://help.sonatype.com/repomanager3/product-information/download)
- `tar –xvzf <file-name>` = untar the file, you will get a nexus folder which contains runtime and applications of nexus and sonatype-work which contains config for nexus and data

**Create a Nexus Service User**

- `addUser nexus`
- `chown -r nexus:nexus <nexus-folder>`
- `chown -r nexus:nexus sonatype-work`
- `vim <nexus-folder>/bin/nexus.rc`
  ```
  run_as_user="nexus"
  ```
- `/opt/<nexus-folder>/bin/nexus start` = start nexus
- `ps aux | grep nexus` = to check nexus is running
- Nexus runs on port 8081, open 8081 in firewall for all sources
- Open up nexus on your browser

**Nexus Repositories**

- **Proxy Repository** = link to a remote repository
- **Hosted Repository** = repository for organizational internal snapshots and releases
- **Group Repository** = collection of other repositories

**Create a Nexus User**

- Settings>Security>User>Create Local User
- Roles>Create Role
- Give role to the user

**Upload Gradle Project to Nexus**

- Config gradle project

  ```java
  apply plugin: 'naven-publish'

  publishing {
    publications {
        maven(MavenPublication) {
          artifact('builds/libs/my-app-$version'+'.jar') {
            extension jar
          }
        }
    }

    repositories {
      maven {
        name 'nexus'
        url 'http://<nexus-ip>:<nexus-port>/repository/maven-snapshots/'
        credientials {
          username project.repoUser
          password project.repoPassword
        }
      }
    }
  }
  ```

  gradle.properties

  ```properties
  repoUser = xxxx
  repoPassword = xxxx
  ```

  settings.gradle

  ```java
  rootProject.name = 'my-app'
  ```

- `./gradlew build` = to create a build
- `./gradlew publish` = to publish the artifact to nexus

**Upload Maven Project to Nexus**

- Inside build tag
  ```xml
  <pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-deploy-plugin</artifactId>
        <version>2.8.2</version>
      </plugin>
    </plugins>
  </pluginManagement>
  ```
  ```xml
  <distributionManagement>
   <snapshotRepository>
      <id>nexus-snapshots</id>
      <url>http://[nexus-ip]:[nexus-port]/repository/maven-snapshots/</url>
   </snapshotRepository>
  </distributionManagement>
  ```
- Setup user crendials. In home directory

  - `cd .m2`
  - `vim settings.xml`

  ```xml
  <servers>
   <server>
      <id>nexus-snapshots</id>
      <username>xxxx</username>
      <password>xxxx</password>
   </server>
  </servers>
  ```

- `mvn package` = to create a build
- `mvn deploy` = to publish the artifact to nexus

**Nexus API**

- `curl -u user:pwd -X GET 'http://<nexus-ip>:<nexus-port>/service/rest/v1/repositories'`= get all repositories
- `curl -u user:pwd -X GET 'http://<nexus-ip>:<nexus-port>/service/rest/v1/components?repository=<repository-name>'`= get all components of repository
- `curl -u user:pwd -X GET 'http://<nexus-ip>:<nexus-port>/service/rest/v1/components/<component-id>'`= get a component of repository

**Nexus Blob Store**<br>
A binary large object (blob) storage, or blobstore, is the folder or network location for where Nexus Repository will store everthing uploaded to or proxied from a repository, including basic metadata for the object. A blob may contain one or more repositories or repository groups. You can move the blob but can't move the repository to another blob. By default, Nexus Repository creates a file blob store named default in the $data-dir directory during installation. You can configure new blob stores by navigating to Administration → Repository → Blob Stores in Nexus Repository. You will need nx-all or nx-blobstore privileges to access this section of Nexus Repository.

**Components vs Assets**<br>
Component = Abstract, high level definition of what we are uploading<br>
Assets = Actual physical package or files, 1 component = 1 or more assets, docker layer = asset

**Cleanup Policies**<br>

- Cleanup Policies> New Cleanup Policies
- Repositories>Select a repository>Apply Cleanup Policy
- You can see the cleanup schedule in the task tab
- Cleaup policy soft deletes the repository to delete permanently run a task schedule for combat blob store
