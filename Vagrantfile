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
      ssh-keygen -s my-ca/ca -I 44398 -n zone-webservers -V +1w -z 1 /vagrant/id_rsa.pub
    SHELL
  end

  config.vm.define "ssh-host" do |box|
    # Read from the certificate authority
    box.vm.provision "shell", inline: <<-SHELL
      sshd_config_path=/etc/ssh/sshd_config
      capub_path=/etc/ssh/ca.pub

      function update_sshd_config {
        grep -q "^${1}" "${sshd_config_path}" && \
          sed -i "s|^${1}.*|${1} ${2}|" "${sshd_config_path}" || \
          echo "${1} ${2}" >> "${sshd_config_path}"
      }

      cp /vagrant/my-ca/ca.pub "${capub_path}"
      update_sshd_config TrustedUserCAKeys "${capub_path}"
      update_sshd_config AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
      systemctl restart sshd
    SHELL

    # Set up some principals and security zones
    box.vm.provision "shell", inline: <<-SHELL
      auth_principals_path=/etc/ssh/auth_principals
      mkdir -p "${auth_principals_path}"

      for principal in root ops
      do
        principal_file_path="${auth_principals_path}/${principal}"
        if [ "${principal}" == "root" ]; then
          echo root-everywhere > "${principal_file_path}"
        else
          echo -e 'zone-webservers\nzone-databases' > "${principal_file_path}"
        fi
      done
    SHELL

    # Make the ops user
    box.vm.provision "shell", inline: <<-SHELL
      adduser ops
      passwd -d ops
    SHELL
  end
end
