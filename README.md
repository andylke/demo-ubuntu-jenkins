# Setup Jenkins

## Install JDK-11

`sudo apt install openjdk-11-jdk-headless`

`java --version`

Output:

```
openjdk 11.0.16 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8-post-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 11.0.16+8-post-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

## Install Jenkins on Ubuntu (Use Weekly Release)

Add the repository key to the system:

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

Append the Debian package repository address to the server’s sources.list

```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Run update so that apt will use the new repository.

`sudo apt-get update`

Install Jenkins and its dependencies.

`sudo apt install jenkins`

## Start Jenkins

Start using systemctl

`sudo systemctl start jenkins.service`

Verify started successfully:

`sudo systemctl status jenkins`

Open Jenkins at `http://192.168.1.10:8080/`

Get Jenkins initial admin password
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Setup Jenkins admin
username: admin
password: admin@123

Install Plugins

1. Pipeline
2. Pipeline: Stage View
3. Pipeline Utility Steps
4. Git
5. Maven Integration
6. Docker
7. Docker Pipeline
8. Config File Provider

## Install YQ - YAML file processor (Use in Pipeline)

`snap install yq`

## Change Jenkins Home Directory

```
sudo service jenkins stop
sudo mv /var/lib/jenkins /home
sudo chown jenkins:jenkins /home/jenkins
sudo usermod -d /home/jenkins jenkins
sudo nano /etc/default/jenkins
```

Change JENKINS_HOME to `/home/jenkins`

```
# jenkins home location
JENKINS_HOME=/home/jenkins
```

`sudo nano /lib/systemd/system/jenkins.service`

Change Environment & WorkingDirectory to `/home/jenkins`

```
# Directory where Jenkins stores its configuration and workspaces
Environment="JENKINS_HOME=/home/jenkins"
WorkingDirectory=/home/jenkins
```

```
sudo systemctl daemon-reload
sudo service jenkins start
```

```
sudo groupadd docker
sudo usermod -aG docker jenkins
```

## Configure SSL on Jenkins

Convert SSL keys to PKCS12 format

```
openssl pkcs12 -export -out jenkins.p12 \
-passout 'pass:admin@123' -inkey server.key \
-in server.crt -certfile ca.crt -name server.example.com
```

Convert PKCS12 to JKS format

```
keytool -importkeystore -srckeystore jenkins.p12 \
-srcstorepass 'admin@123' -srcstoretype PKCS12 \
-srcalias server.example.com -deststoretype JKS \
-destkeystore jenkins.jks -deststorepass 'admin@123' \
-destalias server.example.com
```

Move JKS to `/etc/jenkins`

```
sudo mkdir -p /etc/jenkins
sudo cp jenkins.jks /etc/jenkins/
sudo chown -R jenkins: /etc/jenkins
sudo chmod 700 /etc/jenkins
sudo chmod 644 /etc/jenkins/jenkins.jks
```

Modify Jenkins Configuration

`sudo nano /lib/systemd/system/jenkins.service`

```
Environment="JENKINS_PORT=-1"
Environment="JENKINS_HTTPS_PORT=8080"
Environment="JENKINS_HTTPS_KEYSTORE=/etc/jenkins/jenkins.jks"
Environment="JENKINS_HTTPS_KEYSTORE_PASSWORD=admin@123"
```

Restart Jenkins

```
sudo systemctl daemon-reload
sudo systemctl restart jenkins.service
```

## Grant Docker.sock read access for non root user/group

`sudo chmod 666 /var/run/docker.sock`

## Grant microk8s to admin

```
sudo usermod -a -G microk8s jenkins
sudo chown -f -R jenkins ~/.kube
```
