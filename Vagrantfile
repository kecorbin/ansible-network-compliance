# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  domain_name = 'ansible-network-compliance.test'

  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

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
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  vms = {
    'services' => {
      ip: '192.168.89.100',
    },
    'legacy' => {
      ip: '192.168.89.101',
      box: 'ubuntu/trusty64',
    },
    'crazy-old' => {
      ip: '192.168.89.102',
      box: 'ubuntu/precise64',
    },
    'weird' => {
      ip: '192.168.89.103',
      # Warning: centos7 doesn't work w/ vagrant 1.8.5
      # see https://github.com/mitchellh/vagrant/issues/7610
      box: 'centos/7',
    },
  }

  vms.each do |name, options|
    config.vm.define name do |node|
      node.vm.hostname = "#{name}.#{domain_name}"
      node.vm.network 'private_network', {ip: options.fetch(:ip)}
      node.vm.box = options.fetch(:box, 'ubuntu/xenial64')
      node.vm.provider 'virtualbox' do |v|
        v.cpus = options.fetch(:cpus, 1)
        v.memory = options.fetch(:memory, 512)
      end

      # default user for ubuntu instances
      user = 'ubuntu'

      if node.vm.box.start_with?('centos/')
        # CentOS boxes don't bring up eth1 automatically; and ifup eth1 doesn't
        # work consistently. If reboot doesn't work, I give up.
        node.vm.provision :reload

        # Default user for CentOS is vagrant.
        user = 'vagrant'
      end

      node.vm.provision 'shell' do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        # ensure we can SSH directly to the VM
        s.inline = <<-SHELL
          set -ex
          echo #{ssh_pub_key} >> /home/#{user}/.ssh/authorized_keys
        SHELL
      end

      # xenial only ships with Python 3 by default, ansible requires
      # Python 2.7
      if node.vm.box == 'ubuntu/xenial64'
        node.vm.provision 'shell' do |s|
          s.inline = <<-SHELL
            set -ex
            export DEBIAN_FRONTEND=noninteractive
            apt-get update
            apt-get install -y python
          SHELL
        end
      end
    end
  end
end
