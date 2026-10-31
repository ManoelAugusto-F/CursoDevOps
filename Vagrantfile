Vagrant.configure("2") do |config|

  config.vm.box = "shekeriev/debian-12"

  # Provisionamento comum de rede (firewall liberado, iptables coerente)
  COMMON_NET_PROVISION = <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update -qq
    apt-get install -y -qq iptables nftables >/dev/null 2>&1 || true
    # Garante que nftables não bloqueie tráfego vindo do host-only (enp0s8) e da rede interna (enp0s9)
    if command -v nft >/dev/null 2>&1; then
      nft add rule inet filter input iifname "enp0s8" accept 2>/dev/null || true
      nft add rule inet filter input iifname "enp0s9" accept 2>/dev/null || true
    fi
  SHELL

  # ====================================================
  # controle
  # ====================================================
  config.vm.define "controle" do |controle|
    controle.vm.hostname = "controle"

    # Host-Only: comunicação com web e db
    controle.vm.network "private_network", ip: "192.168.56.100"

    # Rede Interna: acesso entre as VMs (substitui o Bridge)
    controle.vm.network "private_network", ip: "192.168.1.100",
      virtualbox__intnet: "intnet"

    # Port forwarding (fallback confiável)
    controle.vm.network "forwarded_port",
      guest: 8080, host: 8080, host_ip: "127.0.0.1",
      auto_correct: true

    controle.vm.provider "virtualbox" do |vb|
      vb.name   = "controle"
      vb.memory = "6000"
      vb.cpus   = 4
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end

    controle.vm.provision "ansible_local" do |al|
      al.install_mode = "apt"
      al.playbook     = "installdocker.yml"
    end
    controle.vm.provision "ansible_local" do |al|
      al.playbook = "installjenkins.yml"
      al.install_mode = "apt"
    end
    controle.vm.provision "shell", inline: COMMON_NET_PROVISION
  end

  # ====================================================
  # web
  # ====================================================
  config.vm.define "web" do |web|
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.56.101"
    
    # Rede Interna
    web.vm.network "private_network", ip: "192.168.1.101",
      virtualbox__intnet: "intnet"

    web.vm.provider "virtualbox" do |vb|
      vb.name   = "web"
      vb.memory = "512"
      vb.cpus   = 2
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end

    web.vm.provision "shell", inline: COMMON_NET_PROVISION
  end

  # ====================================================
  # db
  # ====================================================
  config.vm.define "db" do |db|
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "192.168.56.102"
    
    # Rede Interna
    db.vm.network "private_network", ip: "192.168.1.102",
      virtualbox__intnet: "intnet"

    db.vm.provider "virtualbox" do |vb|
      vb.name   = "db"
      vb.memory = "512"
      vb.cpus   = 2
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end

    db.vm.provision "shell", inline: COMMON_NET_PROVISION
  end

end
