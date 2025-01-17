# Vagrant Worklfow

[https://developer.hashicorp.com/vagrant]

## Installation 👍

virtualox :

- [https://www.virtualbox.org/wiki/Downloads]

docker :

- [https://docs.docker.com/engine/install/]
- [https://docs.docker.com/desktop/]

utm :

- [https://mac.getutm.app/]

vagrant :

- [https://developer.hashicorp.com/vagrant/install]

vagrant plugin :

- [https://naveenrajm7.github.io/vagrant_utm/]

```bash
vagrant --version
vagrant --help
# plugin list 👍
vagrant plugin install vagrant_utm
vagrant plugin install docker
vagrant plugin install virtualbox
vagrant plugin install vagrant-vmware-desktop

vagrant plugin list
vagrant autocomplete install --bash --zsh
```

## Configuration

### linux / windows

```bash
mkdir vagrant_getting_started
cd vagrant_getting_started
vagrant init hashicorp/bionic64 # génère le fichier Vagrantfile 👍
# vagrant box add hashicorp/bionic64 # optionel
```

### mac

```bash
mkdir vagrant_getting_started
cd vagrant_getting_started
vagrant init utm/bookworm # génère le fichier Vagrantfile 👍
# vagrant box add utm/bookworm # optionel
```

## Manage

```bash
cat Vagrantfile
vagrant validate
vagrant up
vagrant status
vagrant ssh-config
vagrant ssh
vagrant destroy
```

## Vagrantfile exemples

### Vagrantfile virtualbox

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provider "virtualbox" do |u|
    u.name = "debian"
    u.cpus = 2
    u.memory = 4096
  end
end
```

### Vagrantfile utm

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'utm'
Vagrant.configure("2") do |config|
  config.vm.box = "utm/bookworm"
  config.vm.provider "utm" do |u|
    u.name = "debian"
    u.cpus = 2
    u.memory = 4096
    u.notes = "Vagrant: For testing plugin development"
    u.directory_share_mode = "webDAV"
  end
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y
  SHELL
end
```

### Vagrantfile vmware_desktop

[https://developer.hashicorp.com/vagrant/docs/providers/vmware/configuration]

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'vmware_desktop'
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provider "vmware_desktop" do |u|
    u.gui = true
    u.name = "debian"
    u.cpus = 2
    u.memory = 4096
  end
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y
  SHELL
end
```

### Vagrantfile docker

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'
Vagrant.configure("2") do |config|
  config.vm.provider "docker" do |conteneur|
    conteneur.image = "nginx"
    conteneur.ports = ["80:80"]
    conteneur.name = "nginx-container"
  end
  config.vm.synced_folder ".", "/vagrant"
end
```

- Docker Image 👍
  - Create Dockerfile

  ```bash
  touch Dockerfile
  ```

  ```Dockerfile
  FROM ubuntu:focal

  RUN apt-get update; \
      apt-get -y install  openssh-server passwd sudo; \
      apt-get -qq clean; \
      rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

  RUN apt-get update && apt-get install -y iproute2 git curl iputils-ping net-tools wget curl
  RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
      rm -f /lib/systemd/system/multi-user.target.wants/*; \
      rm -f /etc/systemd/system/*.wants/*; \
      rm -f /lib/systemd/system/local-fs.target.wants/*; \
      rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
      rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
      rm -f /lib/systemd/system/basic.target.wants/*; \
      rm -f /lib/systemd/system/anaconda.target.wants/*;

  RUN systemctl enable ssh.service;

  RUN useradd --create-home -s /bin/bash vagrant; \
      echo -e "vagrant\nvagrant" | (passwd --stdin vagrant); \
      echo 'vagrant ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/vagrant; \
      chmod 440 /etc/sudoers.d/vagrant

  RUN mkdir -p /home/vagrant/.ssh; \
      chmod 700 /home/vagrant/.ssh
  ADD https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub /home/vagrant/.ssh/authorized_keys
  RUN chmod 600 /home/vagrant/.ssh/authorized_keys; \
      chown -R vagrant:vagrant /home/vagrant/.ssh

  VOLUME [ "/sys/fs/cgroup" ]

  CMD ["/usr/sbin/init" ]
  ```

  - Build image docker

  ```bash
  docker build -t bootcamp:vagrant .
  ```

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'
Vagrant.configure("2") do |config|
    config.vm.define "serveur_web" do |serveur_web|
    serveur_web.vm.network "forwarded_port", guest: 80, host: 9000
    serveur_web.vm.provider "docker" do |serveur_web|
    serveur_web.image = "bootcamp:vagrant"
    serveur_web.has_ssh = true
    serveur_web.privileged = true
    serveur_web.create_args = ["-v", "/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      serveur_web.name = "serveur_web"
   end
  end
end
```

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'

Vagrant.configure("2") do |config|
    config.vm.define "bootcamp_1" do |bootcamp_1|
    bootcamp_1.vm.network :private_network, type: "static", ip: "192.168.1.10", netmask: 24
    bootcamp_1.vm.hostname="bootcamp1"
    bootcamp_1.vm.provider "docker" do |bootcamp_1|
    bootcamp_1.image = "bootcamp:vagrant" # image docker
    bootcamp_1.has_ssh = true
    bootcamp_1.privileged = true
    bootcamp_1.create_args = ["-v", "/sys/fs/cgroup:/sys/fs/cgroup:ro"]
            bootcamp_1.name = "bootcamp_1"
   end
  end
end
Vagrant.configure("2") do |config|
    config.vm.define "bootcamp_2" do |bootcamp_2|
    bootcamp_2.vm.network :private_network,type: "static", ip: "192.168.1.11", netmask: 24
        bootcamp_2.vm.network :private_network, ip: "192.168.2.11", netmask: 24
        bootcamp_2.vm.hostname="bootcamp2"
    bootcamp_2.vm.provider "docker" do |bootcamp_2|
    bootcamp_2.image = "bootcamp:vagrant" # image docker
    bootcamp_2.has_ssh = true
    bootcamp_2.privileged = true
    bootcamp_2.create_args = ["-v", "/sys/fs/cgroup:/sys/fs/cgroup:ro"]
            bootcamp_2.name = "bootcamp_2"
   end
  end
end

Vagrant.configure("2") do |config|
    config.vm.define "bootcamp_3" do |bootcamp_3|
    bootcamp_3.vm.network :private_network, type: "static", ip: "192.168.2.10", netmask: 24
    bootcamp_3.vm.hostname="bootcamp3"
    bootcamp_3.vm.provider "docker" do |bootcamp_3|
    bootcamp_3.image = "bootcamp:vagrant" # image docker
    bootcamp_3.has_ssh = true
    bootcamp_3.privileged = true
    bootcamp_3.create_args = ["-v", "/sys/fs/cgroup:/sys/fs/cgroup:ro"]
            bootcamp_3.name = "bootcamp_3"
   end
  end
end
```
