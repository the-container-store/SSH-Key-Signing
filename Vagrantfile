# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.4"

  # Read from the certificate authority
  config.vm.provision "shell", env: {VAULT_ADDR:ENV['VAULT_ADDR']}, inline: <<-SHELL
    [ -z "${VAULT_ADDR}" ] && echo "Need to set VAULT_ADDR" && exit 1;
    sshd_config_path=/etc/ssh/sshd_config
    capub_path=/etc/ssh/trusted-user-ca-keys.pem

    function update_sshd_config {
      grep -q "^${1}" "${sshd_config_path}" && \
        sed -i "s|^${1}.*|${1} ${2}|" "${sshd_config_path}" || \
        echo "${1} ${2}" >> "${sshd_config_path}"
    }

    curl "${VAULT_ADDR}/v1/ssh-client-signer/public_key" -o "${capub_path}" -s
    update_sshd_config TrustedUserCAKeys "${capub_path}"
    systemctl restart sshd
  SHELL

  # Make the ops user
  config.vm.provision "shell", inline: <<-SHELL
    adduser ops
    passwd -d ops
  SHELL
end
