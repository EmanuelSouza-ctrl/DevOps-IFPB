
Vagrant.configure("2") do |config|

  # ==========================================
  # CONFIGURAÇÕES GERAIS
  # ==========================================

  config.vm.box = "debian/bookworm64"
  config.vm.boot_timeout = 600
  config.ssh.insert_key = false

  # ==========================================
  # DESATIVAR DHCP DO VIRTUALBOX
  # ==========================================

  config.trigger.before :up do |trigger|
    trigger.name = "Desativar DHCP do VirtualBox"

    trigger.ruby do |env, machine|
      vboxmanage = 'C:\Program Files\Oracle\VirtualBox\VBoxManage.exe'

      unless system(
        vboxmanage,
        'dhcpserver',
        'modify',
        '--interface=VirtualBox Host-Only Ethernet Adapter',
        '--disable'
      )
        raise "Falha ao desativar DHCP do VirtualBox"
      end
    end
  end

  # ==========================================
  # CONFIGURAÇÕES DO VIRTUALBOX
  # ==========================================

  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
    vb.check_guest_additions = false
  end

  # ==========================================
  # ARQ - SERVIDOR DE ARQUIVOS
  # IP: 192.168.56.129
  # ==========================================

  config.vm.define "arq" do |arq|

    arq.vm.hostname = "arq"

    arq.vm.network "private_network",
      ip: "192.168.56.129",
      name: "VirtualBox Host-Only Ethernet Adapter"

    arq.vm.provider "virtualbox" do |vb|
      vb.name = "ARQ"
      vb.memory = 512
      vb.cpus = 1
    end

    # Três discos adicionais de 10 GB
    (1..3).each do |i|
      arq.vm.disk :disk,
        size: "10GB",
        name: "arq-disk-#{i}"
    end

  end

  # ==========================================
  # DB - SERVIDOR DE BANCO DE DADOS
  # IP reservado: 192.168.56.130
  # ==========================================

  config.vm.define "db" do |db|

    db.vm.hostname = "db"

    db.vm.network "private_network",
      ip: "192.168.56.130",
      name: "VirtualBox Host-Only Ethernet Adapter",
      auto_config: false

    db.vm.provider "virtualbox" do |vb|
      vb.name = "DB"
      vb.memory = 512
      vb.cpus = 1
    end

  end

  # ==========================================
  # APP - SERVIDOR WEB
  # IP reservado: 192.168.56.131
  # ==========================================

  config.vm.define "app" do |app|

    app.vm.hostname = "app"

    app.vm.network "private_network",
      ip: "192.168.56.131",
      name: "VirtualBox Host-Only Ethernet Adapter",
      auto_config: false

    app.vm.provider "virtualbox" do |vb|
      vb.name = "APP"
      vb.memory = 512
      vb.cpus = 1
    end

  end

  # ==========================================
  # CLI - MÁQUINA CLIENTE
  # IP provisório da interface: 192.168.56.132
  # Endereço efetivo será recebido via DHCP
  # ==========================================

  config.vm.define "cli" do |cli|

    cli.vm.hostname = "cli"

    cli.vm.network "private_network",
      ip: "192.168.56.132",
      name: "VirtualBox Host-Only Ethernet Adapter",
      auto_config: false

    cli.vm.provider "virtualbox" do |vb|
      vb.name = "CLI"
      vb.memory = 1024
      vb.cpus = 1
    end

  end

end
