Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.boot_timeout = 600

  # ==========================================
  # DESATIVAR DHCP DO VIRTUALBOX
  # ==========================================
  config.trigger.before :up do |trigger|
    trigger.name = "Desativar DHCP do VirtualBox"
    trigger.ruby do |env, machine|
      if Gem.win_platform?
        vboxmanage = 'C:\Program Files\Oracle\VirtualBox\VBoxManage.exe'
        system(vboxmanage, 'dhcpserver', 'modify', '--interface=VirtualBox Host-Only Ethernet Adapter', '--disable', out: File::NULL, err: File::NULL)
        system(vboxmanage, 'dhcpserver', 'remove', '--interface=VirtualBox Host-Only Ethernet Adapter', out: File::NULL, err: File::NULL)
      else
        system('VBoxManage dhcpserver modify --netname HostInterfaceNetworking-vboxnet0 --disable >/dev/null 2>&1 || true')
        system('VBoxManage dhcpserver remove --netname HostInterfaceNetworking-vboxnet0 >/dev/null 2>&1 || true')
      end
    end
  end

  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
    vb.check_guest_additions = false
    vb.customize ["modifyvm", :id, "--paravirtprovider", "kvm"]
  end

  # ==========================================
  # ARQ - SERVIDOR DE ARQUIVOS, DHCP E DNS
  # ==========================================
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.emanuel.souza.devops"
    arq.vm.network "private_network", ip: "192.168.56.129"

    arq.vm.provider "virtualbox" do |vb|
      vb.name = "ARQ"
      vb.memory = 512
      vb.cpus = 1

      (1..3).each do |i|
        disk_file = "./arq-disk-#{i}.vdi"
        unless File.exist?(disk_file)
          vb.customize ["createmedium", "disk", "--filename", disk_file, "--size", 10240]
        end
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", disk_file]
      end
    end
  end

  # ==========================================
  # DB - SERVIDOR DE BANCO DE DADOS (DHCP via MAC)
  # ==========================================
  config.vm.define "db" do |db|
    db.vm.hostname = "db.emanuel.souza.devops"
    db.vm.network "private_network", type: "dhcp", mac: "080027000030"

    db.vm.provider "virtualbox" do |vb|
      vb.name = "DB"
      vb.memory = 512
      vb.cpus = 1
    end
  end

  # ==========================================
  # APP - SERVIDOR WEB (DHCP via MAC)
  # ==========================================
  config.vm.define "app" do |app|
    app.vm.hostname = "app.emanuel.souza.devops"
    app.vm.network "private_network", type: "dhcp", mac: "080027000031"

    app.vm.provider "virtualbox" do |vb|
      vb.name = "APP"
      vb.memory = 512
      vb.cpus = 1
    end
  end

  # ==========================================
  # CLI - MAQUINA CLIENTE (DHCP dinamico)
  # ==========================================
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.emanuel.souza.devops"
    cli.vm.network "private_network", type: "dhcp"

    cli.vm.provider "virtualbox" do |vb|
      vb.name = "CLI"
      vb.memory = 1024
      vb.cpus = 1
    end

    cli.vm.provision "shell", inline: <<-SHELL
      mkdir -p /home/vagrant/.ssh
      for m in arq db app; do
        cp "/vagrant/.vagrant/machines/$m/virtualbox/private_key" "/home/vagrant/.ssh/${m}_key"
        chmod 600 "/home/vagrant/.ssh/${m}_key"
      done
      chown -R vagrant:vagrant /home/vagrant/.ssh

      cat << 'EOF' > /tmp/hosts.ini
[todos]
arq ansible_host=10.0.2.2 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/.ssh/arq_key
db  ansible_host=10.0.2.2 ansible_port=2200 ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/.ssh/db_key
app ansible_host=10.0.2.2 ansible_port=2201 ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/.ssh/app_key
cli ansible_connection=local
EOF
    SHELL

    cli.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "site.yml"
      ansible.install = true
      ansible.install_mode = "default"
      ansible.limit = "all"
      ansible.raw_arguments = [
        "-i", "/tmp/hosts.ini",
        "--ssh-common-args='-o StrictHostKeyChecking=no'"
      ]
    end
  end
end
