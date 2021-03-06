# -*- mode: ruby -*-
# vi: set ft=ruby :
require "yaml"

# Load in external config
config_file = "#{File.dirname(__FILE__)}/vm_config.yml"
default_config_file = "#{File.dirname(__FILE__)}/.vm_config_default.yml"

if !File.file? config_file
  puts "Creating vm_config.yml..."
  FileUtils.cp default_config_file, config_file
end

vm_external_config = YAML.load_file(config_file)

# Install required Vagrant plugins
missing_plugins_installed = false
required_plugins = %w(vagrant-cachier vagrant-hostsupdater vagrant-librarian-puppet)

required_plugins.each do |plugin|
  if !Vagrant.has_plugin? plugin
    system "vagrant plugin install #{plugin}"
    missing_plugins_installed = true
  end
end

# If any plugins were missing and have been installed, re-run vagrant
if missing_plugins_installed
  exec "vagrant #{ARGV.join(" ")}"
end

# Configure Vagrant
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: vm_external_config["ip"]
  config.vm.hostname = vm_external_config["hostname"]
  
  # node-inspector and vscode debug port
  config.vm.network "forwarded_port", guest: 5858, host:5858

  config.vm.synced_folder vm_external_config["app_path"], "/home/vagrant/code/app", :nfs => true

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", vm_external_config["memory"]]
  end

  config.cache.scope = :box

  config.librarian_puppet.placeholder_filename = ".gitkeep"
  config.librarian_puppet.puppetfile_dir = "provisioning"
  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = "provisioning/manifests"
    puppet.manifest_file = "init.pp"
    puppet.module_path = "provisioning/modules"
  end
end
