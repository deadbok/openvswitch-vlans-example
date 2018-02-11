require 'pp'

Vagrant.configure('2') do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  # This is in array of all the machines that gets created.
  vms = [
    {
      :name => 'firewall',
      :box => 'debian/stretch64',
      :memory => 256,
      :cpus => 1,
      :gui => false,
      :networks => [
        {
          :private_ip => "192.168.0.1",
          :private_net_name => "firewall",
        }
      ],
      :host_vars_file => "firewall.yml"
    },
    {
      :name => 'vswitch',
      :box => 'debian/stretch64',
      :memory => 512,
      :cpus => 1,
      :gui => false,
      :networks => [
        {
          :private_ip => "192.168.0.2",
          :private_net_name => "firewall",
        },
        {
          :private_ip => "192.168.10.2",
          :private_net_name => "media",
        },
        {
          :private_ip => "192.168.11.2",
          :private_net_name => "work",
        },
        {
          :private_ip => "192.168.12.2",
          :private_net_name => "wifi",    
        }
      ],
      #:host_vars_file => "vswitch.yml"
    },
    {
      :name => 'srv-media',
      :box => 'debian/stretch64',
      :memory => 512,
      :cpus => 1,
      :gui => false,
      :networks => [
        {
          :private_ip => "192.168.10.3",
          :private_net_name => "media",
        },
      ],
      #:host_vars_file => "srv-media.yml"
    },
    {
      :name => 'workstation',
      :box => 'debian/stretch64',
      :memory => 512,
      :cpus => 1,
      :gui => false,
      :networks => [
        {
          :private_ip => "192.168.11.3",
          :private_net_name => "work",
        },
      ],
      #:host_vars_file => "workstation.yml"
    },
    {
      :name => 'wlan-workstation',
      :box => 'debian/stretch64',
      :memory => 512,
      :cpus => 1,
      :gui => false,
      :networks => [
        {
          :private_ip => "192.168.12.3",
          :private_net_name => "wifi",
        },
      ] ,
      #:host_vars_file => "wlan-workstation.yml"
    }
  ]

  # Create each machine.
  vms.each do |machine_def|
    config.vm.define machine_def[:name] do |machine|
      machine.vm.box = machine_def[:box]
      machine.vm.hostname = machine_def[:name]
      if machine_def.key?(:networks)
        machine_def[:networks].each do |interface|
          if interface.key?(:private_ip)
            machine.vm.network 'private_network',
                                ip: interface[:private_ip],
                                virtualbox__intnet: interface[:private_net_name]
          end
          if machine_def.key?(:public_ip)
            if machine_def[:public_ip] == "dhcp"
              machine.vm.network 'public_network'
            else
              machine.vm.network 'public_network',
                                  ip: machine_def[:public_ip]
            end
          end
        end
      end

      if machine_def.key?(:ssh_port)
        machine.vm.network :forwarded_port, guest: 22,
                            host: machine_def[:ssh_port], id: 'ssh'
      end

      # Only do provider settings if there are any in the definitions.
      config.vm.provider "virtualbox" do |vb|
        if machine_def.key?(:memory)
          vb.memory = machine_def[:memory]
        end
        if machine_def.key?(:cpus)
          vb.cpus = machine_def[:cpus]
        end
        if machine_def.key?(:gui)
          vb.gui = machine_def[:gui]
        end
      end
      config.vm.provision "ansible" do |ansible|
        ansible.limit = machine_def[:name]
        ansible.host_vars = {
          "firewall" => {
            "interfaces" => [
              "eth1" => {
                "method" => "static",
                "ipv4" => "192.168.0.1/24",
                "gateway_ipv4" => "192.168.0.1",
              },
              "eth110" => {
                "method" => "static",
                "ipv4" => "192.168.10.1/24",
                "gateway_ipv4" => "192.168.10.1",
                "vlan-raw-device" => "eth1",
              },
              "eth111" => {
                "method" => "static",
                "ipv4" => "192.168.11.1/24",
                "gateway_ipv4" => "192.168.11.1",
                "vlan-raw-device" => "eth1",
              },
              "eth112" => {
                "method" => "static",
                "ipv4" => "192.168.12.1/24",
                "gateway_ipv4" => "192.168.12.1",
                "vlan-raw-device" => "eth1",
              }
            ]
          },
          "vswitch" => {},
          "srv-media" => {
            "interfaces" => {
              "eth1" => {
                "method" => "static",
                "ipv4" => "192.168.0.3/24",
                "gateway_ipv4" => "192.168.0.1",
              },
              "eth1.10" => {
                "method" => "static",
                "ipv4" => "192.168.10.3/24",
                "gateway_ipv4" => "192.168.10.1",
                "vlan-raw-device" => "eth1",
              }
            }
          },
          "workstation" => {
            "interfaces" => {
              "eth1" => {
                "method" => "static",
                "ipv4" => "192.168.0.4/24",
                "gateway_ipv4" => "192.168.0.1",
              },
              "eth1.11" => {
                "method" => "static",
                "ipv4" => "192.168.11.3/24",
                "gateway_ipv4" => "192.168.11.1",
                "vlan-raw-device" => "eth1",
              }
            }
          },
          "wlan-workstation" => {
            "interfaces" => {
              "eth1" => {
                "method" => "static",
                "ipv4" => "192.168.0.5/24",
                "gateway_ipv4" => "192.168.0.1",
              },
              "eth1.12" => {
                "method" => "static",
                "ipv4" => "192.168.12.3/24",
                "gateway_ipv4" => "192.168.12.1",
                "vlan-raw-device" => "eth1",
              }
            }
          }
        }
        ansible.playbook = "playbook.yml"
        ansible.verbose ="-v"
      end
    end
  end
end
