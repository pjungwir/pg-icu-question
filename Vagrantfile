# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.ssh.forward_agent = true

  config.vm.define "pg15" do |host|
    host.vm.network :private_network, ip: "192.168.48.11"
    host.vm.provider :virtualbox do |vb|
      vb.name = "pg-locale-test-15"
    end

    host.vm.provision :shell, privileged: false do |host|
      host.inline = <<-EOS
        sudo apt-get update
        sudo apt-get install -y gnupg2 wget vim
        sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
        curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
        sudo apt-get update
        sudo apt-get install -y postgresql-15 postgresql-contrib-15

        sudo locale-gen en_IN.UTF-8

        sudo pg_dropcluster --stop 15 main
      EOS
    end
  end

  config.vm.define "pg16" do |host|
    host.vm.network :private_network, ip: "192.168.48.12"
    host.vm.provider :virtualbox do |vb|
      vb.name = "pg-locale-test-16"
    end

    host.vm.provision :shell, privileged: false do |host|
      host.inline = <<-EOS
        sudo apt-get update
        sudo apt-get install -y gnupg2 wget vim
        sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
        curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
        sudo apt-get update
        sudo apt-get install -y postgresql-16 postgresql-contrib-16

        sudo locale-gen en_IN.UTF-8

        sudo pg_dropcluster --stop 16 main
      EOS
    end
  end
end
