# -*- mode: ruby -*-
# vi: set ft=ruby :


#Workaround to fix DHCP errors
# class VagrantPlugins::ProviderVirtualBox::Action::Network
#   def dhcp_server_matches_config?(dhcp_server, config)
#     true
#   end
# end

require 'yaml'

# get details of boxes to build
boxes = YAML.load_file('./boxes.yaml')

# API version
VAGRANTFILE_API_VERSION = 2

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  boxes.each do |boxes|
    config.vm.define boxes['name'] do |srv|
      if boxes['name'] == "dev"
        srv.vm.network "private_network", ip: "192.168.42.1"
      elsif boxes['name'] == "tm-infra"
        srv.vm.network "private_network", ip: "192.168.42.2"
      elsif boxes['name'] == "master"
        srv.vm.network "private_network", ip: "192.168.42.3"
      elsif boxes['name'] == "node"
        srv.vm.network "private_network", ip: "192.168.42.4"
      end    

      # OS and hostname
      srv.vm.box = boxes['box']
      if boxes['box_version']
        srv.vm.box_version = boxes['box_version']
      end
      srv.vm.hostname = boxes['name']

      # Networking.  By default a NAT interface is added.
      # Add an internal network like this:
      #   srv.vm.network 'private_network', type: 'dhcp', virtualbox__intnet: true
      # Add a bridged network
      # if boxes['public_network']
        # if boxes['public_network']['ip']
          # srv.vm.network 'public_network', bridge: boxes['public_network']['bridge'], ip: boxes['public_network']['ip']
        # else
          # srv.vm.network 'public_network', bridge: boxes['public_network']['bridge']
        # end
      # end

      if boxes['ssh_port']
        srv.vm.network :forwarded_port, guest: 22, host: boxes['ssh_port'], host_ip: '127.0.0.1', id: 'ssh'
      end

      # Shared folders

      srv.vm.synced_folder '.', '/vagrant', disabled: "true"
      srv.vm.synced_folder './build', '/opt/treadmill', id: 'tm_venv', type: "sshfs"
      srv.vm.synced_folder './rpms', '/opt/rpms', type: "sshfs"

	  config.vbguest.auto_update = false

	  #config.vbguest.no_remote = true
	  # config.vbguest.auto_reboot = true
      
	  
	  srv.vm.provider 'virtualbox' do |v|
		v.customize ['setextradata', :id, 'VBoxInternal2/SharedFoldersEnableSymlinksCreate/tm_venv', '1']
	  end
	

      srv.vm.provider 'virtualbox' do |vb|
        vb.customize ['modifyvm', :id, '--cpus', boxes['cpus']]
        vb.customize ['modifyvm', :id, '--cpuexecutioncap', boxes['cpu_execution_cap']]
        vb.customize ['modifyvm', :id, '--memory', boxes['ram']]
        vb.customize ['modifyvm', :id, '--name', boxes['name']]
        vb.customize ['modifyvm', :id, '--description', boxes['description']]
        #vb.customize ['modifyvm', :id, '--groups', '/vagrant']
      end

      # Copy cloud-init files to tmp and provision
      if boxes['provision']
        srv.vm.provision :file, :source => boxes['provision']['meta-data'], :destination => '/tmp/vagrant/cloud-init/nocloud-net/meta-data'
        srv.vm.provision :file, :source => boxes['provision']['user-data'], :destination => '/tmp/vagrant/cloud-init/nocloud-net/user-data'
        srv.vm.provision :shell, :path => boxes['provision']['cloud-init']
      end
    end
  end
  
  config.vm.provider :virtualbox do |vb|
	 vb.linked_clone = true
	 #vb.gui = true
  end

  # config.ssh.insert_key = false
  # config.ssh.private_key_path = '~/.ssh/id_rsa'

end