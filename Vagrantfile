require 'yaml'
require 'pp'
require 'json'

vms = YAML.load_file('machines.yml')

Vagrant.configure('2') do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  # Create each machine.
  vms.each do |machine_name, machine_def|
    config.vm.define machine_name do |machine|
      machine.vm.box = machine_def['box']
      machine.vm.hostname = machine_name
      if machine_def.key?('networks')
        machine_def['networks'].each do |network|
          if network.key?('private_ip') && network['internal']
            machine.vm.network 'private_network',
                                ip: network['private_ip'],
                                virtualbox__intnet: network['private_net_name']
          end
          if network.key?('private_ip') && !network['internal']
            machine.vm.network 'private_network',
                                ip: network['private_ip']
          end
          if network.key?('public_ip')
            if network['public_ip'] == "dhcp"
              machine.vm.network 'public_network'
            else
              machine.vm.network 'public_network',
                                  ip: machine_def['public_ip']
            end
          end
        end
      end

      if machine_def.key?('ssh_port')
        machine.vm.network :forwarded_port, guest: 22,
                            host: machine_def['ssh_port'], id: 'ssh'
      end

      config.vm.provider "virtualbox" do |vb|
        if machine_def.key?('memory')
          vb.memory = machine_def['memory']
        end
        if machine_def.key?('cpus')
          vb.cpus = machine_def['cpus']
        end
        if machine_def.key?('gui')
          vb.gui = machine_def['gui']
        end
      end
      config.ssh.insert_key = false
      config.ssh.forward_agent = true
      vagrant_home_path = ENV["VAGRANT_HOME"] ||= "~/.vagrant.d"
      config.ssh.private_key_path = ["#{vagrant_home_path}/insecure_private_key", "~/.ssh/id_rsa"]
      config.vm.provision :shell, privileged: false do |shell_action|
        ssh_public_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        shell_action.inline = <<-SHELL
          echo #{ssh_public_key} >> /home/$USER/.ssh/authorized_keys
        SHELL
      end
      config.vm.provision "ansible" do |ansible|
        ansible.limit = machine_name
        ansible.inventory_path = "inventory/vlan"
        ansible.playbook = "playbook.yml"
        ansible.verbose = "-v"
        #ansible.compatibility_mode = "2.0"
      end
    end
  end
end
