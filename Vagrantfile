# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "almalinux/9"
  
  config.vm.provider "parallels" do |prl|
    prl.update_guest_tools = true
    prl.memory = 4096
    prl.cpus = 4

    # Avoiding request to grant permission to microphone on macOS
    prl.customize "post-import", ["set", :id, "--device-del", "sound0"]
    # Create "docker" hdd, which is mounted on /var/lib/docker
    prl.customize "post-import", ["set", :id, "--device-add", "hdd", "--type", "expand", "--size", "8192", "--split", "--iface", "sata"]
  end

  # Perform update during provisioning
  config.vm.provision "update", type: "shell", inline: <<-UPDATE
    dnf update -y
  UPDATE

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "#{ENV['ANSIBLE_INFRA_SETUP_PATH']}/init2.yml"
    ansible.compatibility_mode = "2.0"
    ansible.raw_arguments = ["--connection=paramiko"]
    ansible.host_vars = {
      "default" => { "layer" => "dev" }
    }
  end

  config.vm.provision "docker", type: "shell", inline: <<-DOCKER
    yum install -y yum-utils
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    usermod -a -G docker vagrant
    systemctl enable docker
  DOCKER

  config.vm.provision "postgres", type: "shell", inline: <<-PSQL
    dnf install -y postgresql
  PSQL

  config.vm.provision "nodejs", type: "shell", inline: <<-NODEJS
    dnf module install -y nodejs:20/development
    npm install -g typescript ts-node newman
  NODEJS

  # Define synced folder
  config.vm.synced_folder ENV['DEVELOPER_PATH'], "/developer"

  # Map ports on developers's machine
  #config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"

  # Avoid ssh-rsa algorithm error while using 'vagrant ssh'
  config.ssh.disable_deprecated_algorithms = true
end

# Parallels Provider: https://parallels.github.io/vagrant-parallels/docs/configuration.html
# Parallels Command-Line Reference: https://download.parallels.com/desktop/v16/docs/en_US/Parallels%20Desktop%20Pro%20Edition%20Command-Line%20Reference.pdf
