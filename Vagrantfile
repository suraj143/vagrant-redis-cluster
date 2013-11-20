# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# How many redis servers. Max 255.
REDIS_SLAVE_INSTANCES = 1

BASE_BOX = "precise64"
BASE_BOX_URL = "http://files.vagrantup.com/precise64.box"

MASTER_IP_ADDRESS = "10.10.25.100"
MASTER_PORT = 6379

SLAVE_IP_PREFIX = "10.10.25.10"

def setup_node(config, ip_addr, options)
  config.vm.network :private_network, ip: ip_addr
  config.vm.provision "shell", path: "shell/main.sh"

  # Puppet modules are managed by librarian-puppet
  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = "puppet/manifests"
    puppet.manifest_file  = "redis.pp"
    puppet.options = [
      "--templatedir", "/tmp/vagrant-puppet/templates",
      "--verbose",
      "--debug"
    ]
    puppet.facter = options
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = BASE_BOX
  config.vm.box_url = BASE_BOX_URL
  config.vm.synced_folder 'puppet/templates/', '/tmp/vagrant-puppet/templates'
  # config.vm.provider :virtualbox do |vb|
  #   # This allows symlinks to be created within the /vagrant root directory, 
  #   # which is something librarian-puppet needs to be able to do. This might
  #   # be enabled by default depending on what version of VirtualBox is used.
  #   vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
  # end

  # Master configuration
  config.vm.define "master_node" do |master|
    options = {
      "node_id" => "master",
      "slave" => false,
      "port" => 6379,
      "slave_priority" => 100,
      "master_ip" => MASTER_IP_ADDRESS,
      "master_port" => MASTER_PORT
    } 
    setup_node(config, MASTER_IP_ADDRESS, options)
  end

  REDIS_SLAVE_INSTANCES.times do |idx|
    config.vm.define "slave_node_#{idx}" do |slave|
      options = {
        "node_id" => idx,
        "slave" => true,
        "port" => 6379 + idx,
        "master_ip" => MASTER_IP_ADDRESS,
        "slave_priority" => 100 + idx,
        "master_port" => MASTER_PORT
      }
      setup_node(slave, "#{SLAVE_IP_PREFIX}#{idx}", options)
    end
  end
end
