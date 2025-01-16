# Vagrant Worklfow

[https://developer.hashicorp.com/vagrant]

## Installation

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
vagrant --help
vagrant plugin install vagrant_utm
vagrant plugin install docker
vagrant plugin install virtualbox

vagrant plugin list
vagrant autocomplete install --bash --zsh
```

## Configuration

### linux / windows

```bash
mkdir vagrant_getting_started
cd vagrant_getting_started
vagrant init hashicorp/bionic64 # génère le fichier Vagrantfile
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

## Vagrantfile exemple

### Vagrantfile docker

```ruby
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'
Vagrant.configure("2") do |config|
  config.vm.provider "docker" do |conteneur|
    conteneur.image = "nginx"
    conteneur.ports = ["80:80"]
    conteneur.name = "nginx-container"
  end
  config.vm.synced_folder ".", "/vagrant" # Nous définissons un volume utilisé dans le conteneur "/vagrant" qui sera synchronisé avec le répertoire courant ou se trouve le Vagrantfile
end
```

### Vagrantfile virtualbox

```ruby
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
