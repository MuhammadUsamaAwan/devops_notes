**Server Setup on Digital Ocean**

- Create a new droplet on digital ocean
- Add your public SSH key

**Setup firewall**

- Droplets>your droplet>access>firewall>edit>create firewall>open port 22 for your
- Add this firewall to your droplet

**Deploy Your Apllication**

- `ssh root@<server-ip>`
- Onstall any dependencies
- `scp <source< <target>:<location>` = eg, `scp test.sh root@159.89.14.94:/rooot`
- Start your application = eg, `java -jar my-app.jar &` & = detach mode
- `ps aux | grep java` = to check your running java application
- `netstat` = to check which service is running on which port
- Open port in firewall for all sources

**Create a Service User**

- `addUser <username>`
- `usermod -aG sudo <username>` = give sudo privileges
- `su - <username>` = login user
- `mkdir .ssh` = in home directory of service user
- `sudo vim .ssh/authorized_keys` = copy your public SSh key, without this you can't access the service directly
- `ssh <username>@<server-ip>`
