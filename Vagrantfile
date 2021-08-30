# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.ssh.insert_key = false
    config.vm.provider "virtualbox" do |vb|
      
    end
    config.vm.define "docker" do |docker|
      docker.vm.box="shekeriev/centos-8-minimal"
      docker.vm.hostname = "docker.dof.lab"
      docker.vm.network "private_network", ip: "192.168.99.100"
      docker.vm.network "forwarded_port", guest: 80, host: 80, auto_correct: true
      docker.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true
      docker.vm.network "forwarded_port", guest: 3000, host: 3000, auto_correct: true
      docker.vm.network "forwarded_port", guest: 9090, host: 9090, auto_correct: true


      docker.vm.synced_folder "vagrant/", "/vagrant"
      docker.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "3048"]
          vb.customize ["modifyvm", :id, "--cpus", "2"] 
      end
      docker.vm.provision "shell", inline: <<EOS
  echo "*Dobavqme hosta "
  echo "192.168.99.100 docker.dof.lab docker" >> /etc/hosts
  echo nameserver 8.8.8.8 > /etc/resolv.conf
  echo "* Add Docker repository ..."
  dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  
  echo "* Instalirame Docker ..."
  dnf install -y docker-ce docker-ce-cli
  
  echo "* Donaglasqme Dockera"
  mkdir -p /etc/docker
  echo '{ "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"] }' | tee /etc/docker/daemon.json
  mkdir -p /etc/systemd/system/docker.service.d/
  echo [Service] | tee /etc/systemd/system/docker.service.d/docker.conf
  echo ExecStart= | tee -a /etc/systemd/system/docker.service.d/docker.conf
  echo ExecStart=/usr/bin/dockerd | tee -a /etc/systemd/system/docker.service.d/docker.conf
  
  echo "* Startirame Dockera"
  systemctl enable docker
  systemctl start docker
  
  echo "* Otvarqme Portovete"
  firewall-cmd --add-port=80/tcp --permanent
  firewall-cmd --add-port=2375/tcp --permanent
  firewall-cmd --add-port=8080/tcp --permanent
  firewall-cmd --add-port=3000/tcp --permanent
  firewall-cmd --add-port=9090/tcp --permanent
  echo "Prezarejdame Rulvoete"
  firewall-cmd --reload
  
  usermod -aG docker vagrant
  yum install git -y
  cp /vagrant/bkirov.yml /home/vagrant
  sudo yum install -y epel-release
  sudo yum install -y ansible
  sudo ansible-playbook bkirov.yml
  #sudo yum install -y mysql-server mysql
  #sudo systemctl start mysqld.service
  #sudo systemctl start mariadb - ako iskash da gi pusnesh
  #sudo systemctl start httpd - ako iskash da gi pusnesh
  cp  -r /vagrant/html /home/vagrant 
  cp /vagrant/docker-volumes.sh /home/vagrant
  cp /vagrant/Jenkins-server-volumes.tar /home/vagrant
  sudo yum install -y nano
  docker run -d   -it   --name WebServer3   -v /home/vagrant/html:/usr/share/nginx/html   -p 80:80   nginx:latest
  docker run -d --name JenkinsServer  -p 8080:8080  jenkins/jenkins
  docker stop JenkinsServer
  docker run -dt --name CentOS1 bkirov/jenkins-agent:latest
  docker run -dt --name CentOS4 bkirov/jenkins-agent:latest
  ./docker-volumes.sh JenkinsServer load Jenkins-server-volumes.tar
  docker start JenkinsServer
  docker network create --driver bridge victor-net
  docker network connect victor-net JenkinsServer
  docker network connect victor-net CentOS1
  docker network connect victor-net CentOS4
  wget https://github.com/vegasbrianc/prometheus/archive/refs/heads/master.zip
  unzip master.zip
  cd prometheus-master
  docker swarm init --advertise-addr=192.168.99.100
  sudo cp /vagrant/docker-stack.yml /home/vagrant/prometheus-master
  HOSTNAME=$(hostname) docker stack deploy -c docker-stack.yml prom
  docker run -dt -p 90:8080 --name Jira2  atlassian/jira-software
EOS
    end
   
end
  