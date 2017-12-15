#    Vagrant topology for ALT Linux build for Mellanox Spectrum Switch and RHEL/CentOS servers
#    Github: https://github.com/kvadrage/altdemo-vagrant.git
#    Credits: https://github.com/CumulusNetworks/cldemo-vagrant
#
#
#    NOTE: in order to use this Vagrantfile you will need:
#       -Vagrant(v1.8.1+) installed: http://www.vagrantup.com/downloads
#       -the "helper_scripts" directory, which includes bootstrap scripts for VMs
#       -the "oob-provision" directory, which includes ansible playbook to provision mgmt VM
#       -Virtualbox installed: https://www.virtualbox.org/wiki/Downloads


$script = <<-SCRIPT
echo "Running post-init script here..."
reboot
SCRIPT

Vagrant.configure("2") do |config|
  wbid = 3
  offset = 100

  config.vm.provider "virtualbox" do |v|
    v.gui=false
  end

  ##### DEFINE VM for oob-mgmt-server #####
  config.vm.define "oob-mgmt-server" do |device|
    device.vm.hostname = "oob-mgmt-server"
    device.vm.box = "boxcutter/ubuntu1604"


    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_oob-mgmt-server"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth1 --> oob-mgmt-switch:swp1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt01", auto_config: false , :mac => "443839000001"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']

      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype2", "virtio"]

end

      # Fixes "stdin: is not a tty" and "mesg: ttyname failed : Inappropriate ioctl for device"  messages --> https://github.com/mitchellh/vagrant/issues/1673
      device.vm.provision :shell , inline: "(grep -q 'mesg n' /root/.profile 2>/dev/null && sed -i '/mesg n/d' /root/.profile  2>/dev/null && echo 'Ignore the previous error, fixing this now...') || true;"

      # Shorten Boot Process - Applies to Ubuntu Only - remove \"Wait for Network\"
      device.vm.provision :shell , inline: "sed -i 's/sleep [0-9]*/sleep 1/' /etc/init/failsafe.conf || true"

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_oob_server.sh"


      extravars = {wbench_hosts: {
          leaf01: {ip: "10.35.81.201", mac: "a0:00:00:00:00:11"},
          leaf02: {ip: "10.35.81.202", mac: "a0:00:00:00:00:12"},
          leaf03: {ip: "10.35.81.203", mac: "a0:00:00:00:00:13"},
          leaf04: {ip: "10.35.81.204", mac: "a0:00:00:00:00:14"},
          spine01: {ip: "10.35.81.205", mac: "a0:00:00:00:00:15"},
          spine02: {ip: "10.35.81.206", mac: "a0:00:00:00:00:16"},
          server01: {ip: "10.35.81.221", mac: "a0:00:00:00:00:21"},
          server02: {ip: "10.35.81.222", mac: "a0:00:00:00:00:22"},
          server03: {ip: "10.35.81.223", mac: "a0:00:00:00:00:23"},
          server04: {ip: "10.35.81.224", mac: "a0:00:00:00:00:24"},
          prod: {ip: "10.35.81.225", mac: "a0:00:00:00:00:25"},
          }}
      device.vm.provision :shell , inline: "sudo apt-get update"
      device.vm.provision :shell , inline: "sudo apt-get install software-properties-common vim lldpd git -y"
      device.vm.provision :shell , inline: "sudo apt-add-repository ppa:ansible/ansible -y"
      device.vm.provision :shell , inline: "sudo apt-get update"
      device.vm.provision :shell , inline: "sudo apt-get install ansible -qy"
      config.vm.provision "file", source: "./oob-provision", destination: "~/oob-provision"
      device.vm.provision :shell , inline: "ansible-playbook oob-provision/site.yml --extra-vars '#{extravars.to_json}'  --connection=local -i localhost,"

      # Clone altdemo-provision playbook
      device.vm.provision :shell , inline: "git clone https://github.com/kvadrage/altdemo-provision.git"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:00:01 eth1"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm --vagrant-name=eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for oob-mgmt-switch #####
  config.vm.define "oob-mgmt-switch" do |device|
    device.vm.hostname = "oob-mgmt-switch"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_oob-mgmt-switch"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 256
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for swp1 --> oob-mgmt-server:eth1
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt01", auto_config: false , :mac => "443839000101"

      # link for swp2 --> leaf01:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt02", auto_config: false , :mac => "443839000102"

      # link for swp3 --> leaf02:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt03", auto_config: false , :mac => "443839000103"

      # link for swp4 --> leaf03:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt04", auto_config: false , :mac => "443839000104"

      # link for swp5 --> leaf04:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt05", auto_config: false , :mac => "443839000105"

      # link for swp6 --> spine01:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt06", auto_config: false , :mac => "443839000106"

      # link for swp7 --> spine02:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt07", auto_config: false , :mac => "443839000107"

      # link for swp8 --> server01:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt08", auto_config: false , :mac => "443839000108"

      # link for swp9 --> server02:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt09", auto_config: false , :mac => "443839000109"

      # link for swp10 --> server03:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt10", auto_config: false , :mac => "443839000110"

      # link for swp11 --> server04:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt11", auto_config: false , :mac => "443839000111"

      # link for swp12 --> prod:eth0
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt12", auto_config: false , :mac => "443839000112"

      # additional mgmt ports
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt13", auto_config: false , :mac => "443839000113"
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt14", auto_config: false , :mac => "443839000114"
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt15", auto_config: false , :mac => "443839000115"
      device.vm.network "private_network", nic_type: "virtio", virtualbox__intnet: "#{wbid}_mgmt16", auto_config: false , :mac => "443839000116"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc8', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc9', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc10', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc11', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc12', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc13', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc14', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc15', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc16', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc17', 'allow-all']

end
      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_oob_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:01 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:02 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:03 swp3"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:04 swp4"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:05 swp5"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:06 swp6"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:07 swp7"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:08 swp8"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:09 swp9"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:10 swp10"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:11 swp11"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:12 swp12"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:13 swp13"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:14 swp14"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:15 swp15"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:39:00:01:16 swp16"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm -nv"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for leaf01 #####
  config.vm.define "leaf01" do |device|
    device.vm.hostname = "leaf01"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_leaf01"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp2
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt02", auto_config: false , :mac => "a00000000011"

      # link for swp13 --> spine01:swp1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_1_0", auto_config: false , :mac => "443840000011"

      # link for swp14 --> spine02:swp1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_1_0", auto_config: false , :mac => "443840000021"

      # link for swp1s0 --> server01:eth11
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_11_0", auto_config: false , :mac => "443840000111"

      # link for swp1s1 --> server03:eth11
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_21_0", auto_config: false , :mac => "443840000211"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:11 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:11 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:21 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:11 swp5s0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:11 swp5s1"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for leaf02 #####
  config.vm.define "leaf02" do |device|
    device.vm.hostname = "leaf02"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_leaf02"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp3
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt03", auto_config: false , :mac => "a00000000012"

      # link for swp13 --> spine01:swp2
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_2_0", auto_config: false , :mac => "443840000012"

      # link for swp14 --> spine02:swp2
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_2_0", auto_config: false , :mac => "443840000022"

      # link for swp1s0 --> server01:eth12
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_12_0", auto_config: false , :mac => "443840000112"

      # link for swp1s1 --> server03:eth12
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_22_0", auto_config: false , :mac => "443840000212"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:12 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:12 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:22 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:12 swp5s0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:12 swp5s1"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for leaf03 #####
  config.vm.define "leaf03" do |device|
    device.vm.hostname = "leaf03"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_leaf03"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp4
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt04", auto_config: false , :mac => "a00000000013"

      # link for swp13 --> spine01:swp3
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_3_0", auto_config: false , :mac => "443840000013"

      # link for swp14 --> spine02:swp3
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_3_0", auto_config: false , :mac => "443840000023"

      # link for swp1s0 --> server02:eth11
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_13_0", auto_config: false , :mac => "443840000113"

      # link for swp1s0 --> server04:eth11
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_23_0", auto_config: false , :mac => "443840000213"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:13 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:13 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:23 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:13 swp5s0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:13 swp5s1"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for leaf04 #####
  config.vm.define "leaf04" do |device|
    device.vm.hostname = "leaf04"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_leaf04"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp5
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt05", auto_config: false , :mac => "a00000000014"

      # link for swp13 --> spine01:swp4
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_4_0", auto_config: false , :mac => "443840000014"

      # link for swp14 --> spine02:swp4
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_4_0", auto_config: false , :mac => "443840000024"

      # link for swp1s0 --> server02:eth12
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_14_0", auto_config: false , :mac => "443840000114"

      # link for swp1s0 --> server04:eth12
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_24_0", auto_config: false , :mac => "443840000214"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:14 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:14 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:00:24 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:14 swp5s0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:14 swp5s1"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for spine01 #####
  config.vm.define "spine01" do |device|
    device.vm.hostname = "spine01"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_spine01"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp6
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt06", auto_config: false , :mac => "a00000000015"

      # link for swp1 --> leaf01:swp13
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_1_0", auto_config: false , :mac => "443840000101"

      # link for swp2 --> leaf02:swp13
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_2_0", auto_config: false , :mac => "443840000102"

      # link for swp3 --> leaf03:swp13
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_3_0", auto_config: false , :mac => "443840000103"

      # link for swp4 --> leaf04:swp13
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_1_4_0", auto_config: false , :mac => "443840000104"

      # link for swp5 --> prod:swp1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_3_1_0", auto_config: false , :mac => "443840001301"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:15 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:01 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:02 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:03 swp3"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:04 swp4"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:13:01 swp5"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for spine02 #####
  config.vm.define "spine02" do |device|
    device.vm.hostname = "spine02"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_spine02"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp7
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt07", auto_config: false , :mac => "a00000000016"

      # link for swp1 --> leaf01:swp14
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_1_0", auto_config: false , :mac => "443840000201"

      # link for swp2 --> leaf02:swp14
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_2_0", auto_config: false , :mac => "443840000202"

      # link for swp3 --> leaf03:swp14
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_3_0", auto_config: false , :mac => "443840000203"

      # link for swp4 --> leaf04:swp14
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_2_4_0", auto_config: false , :mac => "443840000204"

      # link for swp5 --> prod:swp2
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_3_2_0", auto_config: false , :mac => "443840002301"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc7', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:16 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:01 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:02 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:03 swp3"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:04 swp4"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:23:01 swp5"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for server01 #####
  config.vm.define "server01" do |device|
    device.vm.hostname = "server01"
    device.vm.box = "centos/7"


    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_server01"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 256
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp8
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt08", auto_config: false , :mac => "a00000000021"

      # link for eth11 --> leaf01:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_11_0", auto_config: false , :mac => "443840000121"

      # link for eth12 --> leaf02:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_12_0", auto_config: false , :mac => "443840000122"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_server_centos.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:21 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:21 eth11"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:22 eth12"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for server02 #####
  config.vm.define "server02" do |device|
    device.vm.hostname = "server02"
    device.vm.box = "centos/7"


    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_server02"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 256
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp9
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt09", auto_config: false , :mac => "a00000000022"

      # link for eth11 --> leaf03:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_13_0", auto_config: false , :mac => "443840000123"

      # link for eth12 --> leaf04:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_14_0", auto_config: false , :mac => "443840000124"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_server_centos.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:22 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:23 eth11"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:01:24 eth12"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for server03 #####
  config.vm.define "server03" do |device|
    device.vm.hostname = "server03"
    device.vm.box = "centos/7"


    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_server03"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 256
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp10
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt10", auto_config: false , :mac => "a00000000023"

      # link for eth11 --> leaf01:swp1s1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_21_0", auto_config: false , :mac => "443840000221"

      # link for eth12 --> leaf02:swp1s1
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_22_0", auto_config: false , :mac => "443840000222"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_server_centos.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:23 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:21 eth11"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:22 eth12"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for server04 #####
  config.vm.define "server04" do |device|
    device.vm.hostname = "server04"
    device.vm.box = "centos/7"


    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_server04"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 256
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp11
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt11", auto_config: false , :mac => "a00000000024"

      # link for eth11 --> leaf03:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_23_0", auto_config: false , :mac => "443840000223"

      # link for eth12 --> leaf04:swp1s0
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_0_24_0", auto_config: false , :mac => "443840000224"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_server_centos.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:24 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:23 eth11"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:02:24 eth12"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

  ##### DEFINE VM for prod #####
  config.vm.define "prod" do |device|
    device.vm.hostname = "prod"
    device.vm.box = "alt_spectrum"
    config.vbguest.auto_update = false

    device.vm.provider "virtualbox" do |v|
      v.name = "#{wbid}_prod"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 512
    end
    device.vm.synced_folder ".", "/vagrant", disabled: true

      # NETWORK INTERFACES
      # link for eth0 --> oob-mgmt-switch:swp7
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_mgmt12", auto_config: false , :mac => "a00000000025"

      # link for swp1 --> spine01:swp5
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_3_1_0", auto_config: false , :mac => "443840000301"

      # link for swp2 --> spine02:swp5
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_net10_3_2_0", auto_config: false , :mac => "443840000302"

      # link for swp3 --> reserve
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_reserve1", auto_config: false , :mac => "443840000303"

      # link for swp4 --> reserve
      device.vm.network "private_network", virtualbox__intnet: "#{wbid}_reserve2", auto_config: false , :mac => "443840000304"


    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc5', 'allow-all']
      vbox.customize ['modifyvm', :id, '--nicpromisc6', 'allow-all']

  end

      # Run Any Extra Config
      device.vm.provision :shell , path: "./helper_scripts/config_switch_alt.sh"

      # Apply the interface re-map
      device.vm.provision "file", source: "./helper_scripts/apply_udev.py", destination: "/home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "chmod 755 /home/vagrant/apply_udev.py"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a a0:00:00:00:00:25 eth0"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:03:01 swp1"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:03:02 swp2"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:03:03 swp3"
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -a 44:38:40:00:03:04 swp4"

      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -vm "
      device.vm.provision :shell , inline: "/home/vagrant/apply_udev.py -s"
      device.vm.provision :shell , :inline => $script

  end

end
