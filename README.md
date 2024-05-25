# Setting Up CI/CD Pipeline with Vagrant, Docker, and Jenkins

## Prerequisites
Operating System: This guide assumes you're using a Linux-based operating system.
Basic Terminal Skills: Familiarity with terminal commands.

## 1. Install Vagrant
Vagrant simplifies the setup of development environments. Follow these steps to install Vagrant:

Download the appropriate installer from Vagrant Downloads.
Install Vagrant by running the installer.
OR
Create a Vagrantfile and paste
```
Vagrant.configure("2") do |config|

  ### Master VM ####
  config.vm.define "cicd" do |cicd|
    cicd.vm.box = "ubuntu/jammy64"
    cicd.vm.hostname = "cicd"
    cicd.vm.network "private_network", ip: "192.168.56.20"
    cicd.vm.provider "virtualbox" do |vb|
      vb.memory = "3072"
      vb.cpus = 2
    end
  end
  
end

```
