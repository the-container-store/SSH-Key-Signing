# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.4"

  # Read from the certificate authority
  config.vm.provision "shell", inline: <<-SHELL
    sshd_config_path=/etc/ssh/sshd_config
    capub_path=/etc/ssh/trusted-user-ca-keys.pem

    function update_sshd_config {
      grep -q "^${1}" "${sshd_config_path}" && \
        sed -i "s|^${1}.*|${1} ${2}|" "${sshd_config_path}" || \
        echo "${1} ${2}" >> "${sshd_config_path}"
    }

    cp /vagrant/trusted-user-ca-keys.pem "${capub_path}"
    update_sshd_config TrustedUserCAKeys "${capub_path}"
    update_sshd_config AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
    systemctl restart sshd
  SHELL

  # Set up some principals and security zones
  config.vm.provision "shell", inline: <<-SHELL
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
  config.vm.provision "shell", inline: <<-SHELL
    adduser ops
    passwd -d ops
  SHELL
end
