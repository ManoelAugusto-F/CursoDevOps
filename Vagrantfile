Vagrant.configure("2") do |config|

  config.vm.box = "shekeriev/debian-12"

  # Provisionamento comum de rede (firewall liberado, iptables coerente)
  COMMON_NET_PROVISION = <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update -qq
    apt-get install -y -qq iptables nftables >/dev/null 2>&1 || true
    # Garante que nftables não bloqueie tráfego vindo do host-only
    if command -v nft >/dev/null 2>&1; then
      nft add rule inet filter input iifname "enp0s8" accept 2>/dev/null || true
    fi
  SHELL

  # ====================================================
  # controle
  # ====================================================
  config.vm.define "controle" do |controle|
    controle.vm.hostname = "controle"

    # Host-Only: comunicação com web e db
    controle.vm.network "private_network", ip: "192.168.56.100"

    # Bridge: acesso do host e da rede 192.168.1.X
    controle.vm.network "public_network",
      ip: "192.168.1.100",
      bridge: "enp4s0"

    # Port forwarding (fallback confiável)
    controle.vm.network "forwarded_port",
      guest: 8080, host: 8080, host_ip: "127.0.0.1",
      auto_correct: true

    controle.vm.provider "virtualbox" do |vb|
      vb.name   = "controle"
      vb.memory = "1024"
      vb.cpus   = 2
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end

    controle.vm.provision "ansible_local" do |al|
      al.install_mode = "apt"
      al.playbook     = "installdocker.yml"
    end

    controle.vm.provision "shell", inline: COMMON_NET_PROVISION
  end

  # ====================================================
  # web
  # ====================================================
  config.vm.define "web" do |web|
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.56.101"
    web.vm.network "public_network",
      ip: "192.168.1.101",
      bridge: "enp4s0"

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
    db.vm.network "public_network",
      ip: "192.168.1.102",
      bridge: "enp4s0"

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
