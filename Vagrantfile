# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.4"

  config.vm.define "ca-server" do |box|
    # Make a certificate authority
    box.vm.provision "shell", inline: <<-SHELL
      umask 77
      mkdir -p /vagrant/my-ca && cd /vagrant/my-ca
      yes y | ssh-keygen -C CA -f ca -q -P ""
    SHELL

    # Sign my personal pubkey
    box.vm.provision "shell", inline: <<-SHELL
      cd /vagrant
      ssh-keygen -s my-ca/ca -I 44398 -n root -V +1w -z 1 /vagrant/id_rsa.pub
    SHELL
  end

  config.vm.define "ssh-host" do |box|
    # Read from the certificate authority
    box.vm.provision "shell", inline: <<-SHELL
      cp /vagrant/my-ca/ca.pub /etc/ssh/ca.pub
      grep -q '^TrustedUserCAKeys' /etc/ssh/sshd_config && sed -i 's|^TrustedUserCAKeys.*|TrustedUserCAKeys /etc/ssh/ca.pub|' /etc/ssh/sshd_config || echo 'TrustedUserCAKeys /etc/ssh/ca.pub' >> /etc/ssh/sshd_config
      systemctl restart sshd
    SHELL
  end
end
