# -*- mode: ruby -*-
# vim: set ft=ruby :

# Указываем зеркало для скачивания образов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|

  # ============================================
  # 1. inetRouter (AlmaLinux 9)
  # ============================================
  config.vm.define "inetRouter" do |inet|
    inet.vm.box = "almalinux/9"
    inet.vm.hostname = "inetRouter"
    inet.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    inet.vm.network "private_network", ip: "192.168.255.1", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"
    inet.vm.network "private_network", ip: "192.168.61.10", adapter: 3, netmask: "255.255.255.0"
    inet.vm.network "private_network", ip: "192.168.255.5", adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"

    inet.vm.provision "shell",
      run: "always",
      inline: <<-SHELL
        sudo dnf install -y iptables-services
        sudo systemctl enable iptables
        sudo sysctl -w net.ipv4.ip_forward=1
        grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
        sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
        sudo iptables -A FORWARD -j ACCEPT
        sudo iptables-save | sudo tee /etc/sysconfig/iptables
        sudo nmcli connection modify "System eth1" +ipv4.routes "192.168.0.0/16 192.168.255.2"


        sudo nmcli con reload
        sudo nmcli con up 'System eth1'
        sudo nmcli con up 'System eth3'
      SHELL
  end

  # ============================================
  # 2. centralRouter
  # ============================================
  config.vm.define "centralRouter" do |central|
    central.vm.box = "almalinux/9"
    central.vm.hostname = "centralRouter"
    central.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    central.vm.network "private_network", ip: "192.168.255.2", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"
    central.vm.network "private_network", ip: "192.168.62.10", adapter: 3, netmask: "255.255.255.0"
    central.vm.network "private_network", ip: "192.168.255.9", adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "office1Router-net"
    central.vm.network "private_network", ip: "192.168.255.6", adapter: 5, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"

    central.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo sysctl -w net.ipv4.ip_forward=1
      grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.255.1"
      sudo nmcli connection modify "System eth3" +ipv4.routes "192.168.2.0/24 192.168.255.10"

      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
      sudo nmcli con up 'System eth2'
      sudo nmcli con up 'System eth3'
      sudo nmcli con up 'System eth4'
    SHELL
  end

  # ============================================
  # 3. office1Router
  # ============================================
  config.vm.define "office1Router" do |office1|
    office1.vm.box = "almalinux/9"
    office1.vm.hostname = "office1Router"
    office1.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    office1.vm.network "private_network", ip: "192.168.255.10", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1Router-net"
    office1.vm.network "private_network", ip: "192.168.63.10", adapter: 3, netmask: "255.255.255.0"
    office1.vm.network "private_network", ip: "192.168.56.1", adapter: 5, netmask: "255.255.255.0", virtualbox__intnet: "testLAN"

    office1.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo sysctl -w net.ipv4.ip_forward=1
      grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.255.9"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
      sudo nmcli con up 'System eth3'
    SHELL
  end

  # ============================================
  # 4. testclient1
  # ============================================
  config.vm.define "testclient1" do |cli1|
    cli1.vm.box = "almalinux/9"
    cli1.vm.hostname = "testclient1"
    cli1.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    cli1.vm.network "private_network", ip: "192.168.56.2", adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "testLAN"
    cli1.vm.network "private_network", ip: "192.168.64.10", adapter: 3, netmask: "255.255.255.0"

    cli1.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      # sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.2.129"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

  # ============================================
  # 5. testclient2
  # ============================================
  config.vm.define "testclient2" do |cli2|
    cli2.vm.box = "ubuntu/jammy64"
    cli2.vm.hostname = "testclient2"
    cli2.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    cli2.vm.network "private_network", ip: "192.168.56.3", adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "testLAN"
    cli2.vm.network "private_network", ip: "192.168.65.10", adapter: 3, netmask: "255.255.255.0"

#    cli2.vm.provision "shell",
#    run: "always",
#    inline: <<-SHELL
#      nmcli con modify 'eth0' ipv4.never-default yes
#      nmcli con reload
#      nmcli con up 'eth0'
#      # sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.2.129"
#      sudo nmcli con reload
#      sudo nmcli con up 'System eth1'
#    SHELL

cli2.vm.provision "file", source: "99-vlan-client2.yaml", destination: "/tmp/99-vlan.yaml"

cli2.vm.provision "shell", inline: <<-SHELL
  sudo cp /tmp/99-vlan.yaml /etc/netplan/99-vlan.yaml
  sudo netplan apply
SHELL

  end

  # ============================================
  # 6. testserver1
  # ============================================
  config.vm.define "testserver1" do |srv1|
    srv1.vm.box = "almalinux/9"
    srv1.vm.hostname = "testserver1"
    srv1.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    srv1.vm.network "private_network", ip: "192.168.56.4", adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "testLAN"
    srv1.vm.network "private_network", ip: "192.168.66.10", adapter: 3, netmask: "255.255.255.0"

    srv1.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      # sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.2.129"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

  # ============================================
  # 7. testserver2
  # ============================================
  config.vm.define "testserver2" do |srv2|
    srv2.vm.box = "ubuntu/jammy64"
    srv2.vm.hostname = "testserver2"
    srv2.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    srv2.vm.network "private_network", ip: "192.168.56.5", adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "testLAN"
    srv2.vm.network "private_network", ip: "192.168.67.10", adapter: 3, netmask: "255.255.255.0"

srv2.vm.provision "file", source: "99-vlan-server2.yaml", destination: "/tmp/99-vlan.yaml"

srv2.vm.provision "shell", inline: <<-SHELL
  sudo cp /tmp/99-vlan.yaml /etc/netplan/99-vlan.yaml
  sudo netplan apply
SHELL

#    srv2.vm.provision "shell",
#    run: "always",
#    inline: <<-SHELL
#     nmcli con modify 'eth0' ipv4.never-default yes
#      nmcli con reload
#      nmcli con up 'eth0'
#      # sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.2.129"
#      sudo nmcli con reload
#      sudo nmcli con up 'System eth1'
#    SHELL
  end

end
