# -*- mode: ruby -*-
# vi: set ft=ruby :

# required plugins check
if ARGV[0] != 'plugin'
  required_plugins = [
    'vagrant-reload',
    'vagrant-secret'
  ]

  plugins_to_install = required_plugins.select {
    |plugin| not Vagrant.has_plugin? plugin }
  
  if not plugins_to_install.empty?
    raise "\n[IMPORTANT] Vagrant plugins missing: #{plugins_to_install.join(' ')}\nrun: vagrant plugin install (...)"
  end
end

$vagrantfile_api_version = "2"
$synced_folder = "syncedfolder"
$provider = "hyperv"
$ubuntu = "bento/ubuntu-16.04"
$centos = "bento/centos-7.5"

# user to be created on control node
$control_node_username = Secret.ctrluser
$control_node_password = Secret.ctrluserpassword

# smb synced folder username && password
# from Secret file
$synced_folder_username = Secret.smbusername
$synced_folder_password = Secret.smbpassword

# add user && add to sudo group && create the user group
$ctrluserscript = <<SCRIPT
useradd -m -p $(openssl passwd -1 #{$control_node_password}) -s /bin/bash -G sudo #{$control_node_username}
adduser #{$control_node_username} sudo
adduser #{$control_node_username} #{$control_node_username}
SCRIPT

Vagrant.configure($vagrantfile_api_version) do |config|
  config.vm.provider $provider do |hv|
    hv.linked_clone = "true"
  end
  
  config.vm.synced_folder $synced_folder, "/vagrant",
    type: "smb",
    smb_username: $synced_folder_username,
    smb_password: $synced_folder_password
  
  # config.vm.network "private_network", type: "dhcp"

  # ansible controller
  config.vm.define "ubuntu-c" do |ubuntuc|
    ubuntuc.vm.box = "bento/ubuntu-16.04"
    ubuntuc.vm.hostname = "ubuntu-c"
    ubuntuc.vm.provision "shell", path: "update_system.sh"
    # use Vagrant Reload Provisioner; "to do a reload on a VM during provisioning"
    # https://github.com/aidanns/vagrant-reload/blob/master/README.md
    ubuntuc.vm.provision "reload"
    ubuntuc.vm.provision "shell", path: "install_ansible.sh"
    ubuntuc.vm.provision "shell", inline: $ctrluserscript
  end

  # managed nodes: ubuntu
  # ubuntu x3
  (1..3).each do |i|
    config.vm.define "ub-mg-#{i}" do |umg|
      umg.vm.box = $ubuntu
      umg.vm.hostname = "ub-managed-#{i}"
    end
  end
  
  # dnsmasq x1
  config.vm.define "experimental" do |dnsmasq|
    dnsmasq.vm.box = $ubuntu
    dnsmasq.vm.hostname = "experimental"
  end

  # centos x3
  (1..3).each do |i|
    config.vm.define "ce-mg-#{i}" do |cemg|
      cemg.vm.box = $centos
      cemg.vm.hostname = "ce-managed-#{i}"
      cemg.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end
end
