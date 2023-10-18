![2048 K8S](https://github.com/shubnimkar/2048_React_K8S/assets/46809421/cc9befdb-6e10-4408-9d7c-6f2281a78c9b)

Find complete steps here in my blog [DevSecOps: Deploying the 2048 Game on Docker and Kubernetes with Jenkins CI/CD](https://medium.com/@shubnimkar/devsecops-deploying-the-2048-game-on-docker-and-kubernetes-with-jenkins-ci-cd-675a6fe5caa7)

## Steps

1. Launch an Ubuntu(20.04) AWS t2.large / GCP e2-highcpu-8
2. Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.
3. Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.
4. Create a Pipeline Project in Jenkins using a Declarative Pipeline
5. Install OWASP Dependency Check Plugins
6. Docker Image Build and Push
7. Deploy the image using Docker
8. Kubernetes master and slave setup on Ubuntu (20.04) t2.medium/e2-medium
9. Access the Game on Browser.

## Installation steps:
* On Main server

## Jenkins
  
      sudo apt install openjdk-11-jre
      
      curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
        /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
        https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
        /etc/apt/sources.list.d/jenkins.list > /dev/null
      
      sudo apt-get update -y 
      
      sudo apt-get install jenkins -y
      
      sudo systemctl enable jenkins
      
      sudo systemctl start jenkins
      
      sudo systemctl status jenkins
  
## Docker

    sudo apt-get update -y
    
    sudo apt-get install ca-certificates curl gnupg
    
    sudo install -m 0755 -d /etc/apt/keyrings
    
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    
    echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      
    sudo apt-get update -y
      
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      
    sudo usermod -aG docker $USER
    
    newgrp docker
    
    sudo chmod 666 /var/run/docker.sock
    
    sudo systemctl restart docker

## Trivy

    sudo apt-get install wget apt-transport-https gnupg lsb-release
    
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
    
    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
    
    sudo apt-get update
    
    sudo apt-get install trivy -y
    
* Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.

* Install Plugins like JDK, Sonarqube Scanner, NodeJS, OWASP Dependency Check, kubernetes
Go to Manage Jenkins →Plugins → Available Plugins →

* Install below plugins

    1 → Eclipse Temurin Installer (Install without restart)
    
    2 → Sonarqube Scanner (Install without restart)
    
    3 → NodeJS Plugin (Install Without restart)
    
    4 → OWASP Dependency-Check (Install Without restart)
    
    5 → Search for Docker and install these plugins (Install Without restart)

        Docker
        
        Docker Commons
        
        Docker Pipeline
        
        Docker API
        
        docker-build-step

    6 → Kubernetes (Install Without restart)

* After the docker installation, we create a Sonarqube container (Remember to add 9000 ports in the security group).

      docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Now our Sonarqube is up and running

## Kubernetes Setup:

Take Two Ubuntu 20.04 instances one for k8s master and the other one for worker.

* Install Kubectl on Jenkins machine also.

* Kubectl is to be installed on Jenkins also
  
      sudo apt update
      sudo apt install curl -y
      curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
      sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
      kubectl version --client
  
* Part 1 — — — — — Master Node — — — — — —

    sudo hostnamectl set-hostname K8s-Master

* — — — — — Worker Node — — — — — —

    sudo hostnamectl set-hostname K8s-Worker

* Part 2 — — — — — — Both Master & Node — — — — —

      sudo apt-get update 
      sudo apt-get install -y docker.io
      sudo usermod –aG docker Ubuntu
      newgrp docker
      sudo chmod 777 /var/run/docker.sock
      sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
      deb https://apt.kubernetes.io/ kubernetes-xenial main     # 3lines same command
      EOF
      sudo apt-get update
      sudo apt-get install -y kubelet kubeadm kubectl
      sudo snap install kube-apiserver

* Part 3 — — — — — — — — Master — — — — — — — -

      sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# in case your in root exit from it and run below commands

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

* — — — — — Worker Node — — — — — —

      sudo kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>

Copy the config file to Jenkins master or the local file manager and save it
copy it and save it in documents or another folder save it as secret-file.txt

Note: create a secret-file.txt in your file explorer save the config in it and use this at the Kubernetes credential section.

## Final Deployment on kubernetes

![kubernetes deployment](https://github.com/shubnimkar/2048_React_K8S/assets/46809421/01ced688-7ca2-4dc2-aa3c-6684ff4f4bc9)
