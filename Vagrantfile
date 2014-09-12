# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "hashicorp/precise64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 4321
  config.vm.network "forwarded_port", guest: 8800, host: 4322

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  # avoids 'stdin: is not a tty' error.
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Use this specific, not-default-for-Vagrant Chef version
  config.omnibus.chef_version = "11.12.8"

  # The file nodes/vagrant.json contains the Chef attributes,
  # plus a run_list.

  json_erb_path = File.join(File.dirname(__FILE__), "nodes", "all_nodes.json.erb")
  eruby = Erubis::Eruby.new File.read(json_erb_path)

  # TODO: add Vagrant-specific node file and merge it over top of all_nodes.json.erb
  chef_json = JSON.parse eruby.result({})
  raise "Can't read JSON file for vagrant Chef node!" unless chef_json

  # TODO: test on Windows
  home_dir = ENV['HOME'] || ENV['userprofile']
  creds_dir = File.join(home_dir, '.deploy_credentials')

  # Read local credentials and pass them to Chef
  #chef_json['ssh_public_key'] = File.read File.join(creds_dir, 'id_rsa_4096.pub')
  chef_json['authorized_keys'] = File.read File.join(creds_dir, 'authorized_keys')

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = ["site-cookbooks", "cookbooks"]
    chef.roles_path = "roles"
    chef.data_bags_path = "data_bags"
    chef.provisioning_path = "/tmp/vagrant-chef"

    # WORKAROUND: This is to prevent a nasty SSL and HTTP warning
    chef.custom_config_path = "Vagrantfile.chef"

    # You may also specify custom JSON attributes:
    run_list = chef_json.delete 'run_list'
    chef.json = chef_json
    chef.run_list = run_list
  end
end
