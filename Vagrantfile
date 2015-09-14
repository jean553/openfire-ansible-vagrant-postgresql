# -*- mode: ruby -*-
# vi: set ft=ruby :

app_vars = {
  OPENFIRE_URL: 'http://download.igniterealtime.org/openfire/openfire_3.10.2_all.deb',
  OPENFIRE_PACKAGE: '/opt/openfire.deb',
  DB_USER: 'openfire',
  DB_PASSWORD: 'openfire',
  DB_NAME: 'openfire' 
}

#ansible_verbosity = 'vvvv'

#Fix for people with strange locale settings
ENV.keys.each {|k| k.start_with?('LC_') && ENV.delete(k)}

def host_box_is_unixy?
  (RUBY_PLATFORM !~ /cygwin|mswin|mingw|bccwin|wince|emx/)
end

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  verbosity_arg = if defined? ansible_verbosity then ansible_verbosity else '' end
  if host_box_is_unixy?
    config.vm.synced_folder "./", "/vagrant", type: "nfs"
    config.vm.provision :ansible do |ansible|
      ansible.playbook = 'provisioning/server.yml'
      ansible.extra_vars = app_vars
    end
  else
    config.vm.synced_folder "./", "/vagrant", type: "smb", mount_options: ['ip=192.168.50.1'] #host side of :private_network
    extra_vars_arg = '{' + app_vars.map{|k,v| '"' + k.to_s + '":"' + v.to_s + '"'}.join(',') + '}'

    config.vm.provision :shell, :inline => <<-END
set -e
if ! which ansible-playbook ; then
  echo "Windows host environment: the devbox will install Ansible and self-provision itself" >&2
  sudo apt-get update
  sudo apt-get -y install python-software-properties
  sudo add-apt-repository ppa:ansible/ansible
  sudo apt-get update
  sudo apt-get -y install ansible
fi
PYTHONUNBUFFERED=1 ansible-playbook --connection=local -i "[default] $(hostname)," \\
--extra-vars='#{ extra_vars_arg }' #{ verbosity_arg.empty? ? '' : '-'+ansible_verbosity } /vagrant/provisioning/server.yml
END
  end
  
  config.vm.network "forwarded_port", guest: 9090, host: 8081
  config.vm.network "forwarded_port", guest: 9091, host: 8082
  config.vm.network "forwarded_port", guest: 5222, host: 6222
  config.vm.network "forwarded_port", guest: 5223, host: 6223
  config.vm.network "private_network", ip: "192.168.50.21"

end
