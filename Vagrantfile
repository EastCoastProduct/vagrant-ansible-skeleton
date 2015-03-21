# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.2"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # by default, calculate the amount of cpu and memory to give a vm
  host = RbConfig::CONFIG['host_os']

  # Give VM 1/4 system memory & access to all cpu cores on the host
  if host =~ /darwin/
    cpus = `sysctl -n hw.ncpu`.to_i
    # sysctl returns Bytes and we need to convert to MB
    mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
  elsif host =~ /linux/
    cpus = `nproc`.to_i
    # meminfo shows KB and we need to convert to MB
    mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
  else # sorry Windows folks, I can't help you
    cpus = 2
    mem = 1024
  end

  config.user.defaults = {
    "vm" => {
      "memory" => mem,
      "cpus" => cpus
    }
  }

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "hashicorp/precise64"

  config.ssh.forward_agent = true

  # disable the default /vagrant share
  config.vm.synced_folder "./", "/vagrant", disabled: true

  # https://github.com/mitchellh/vagrant/issues/1673#issuecomment-28288042
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

#  config.vm.provision :host_shell do |host_shell|
#    host_shell.inline = 'ansible-galaxy install --ignore-errors -r #provisioning/all.roles'
#  end

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", config.user.vm.memory]
    vb.customize ["modifyvm", :id, "--cpus", config.user.vm.cpus]
  end


  # ====================================================
  # App Server
  # (composite web, api, db, etc)
  # ====================================================

  config.vm.define "app", primary: true do |app|
    app.vm.post_up_message = <<-EOS
App server running on 192.168.50.4
To ssh to vm use 'vagrant ssh app'
EOS

    ip = "192.168.50.4"

    app.vm.network "private_network", ip: ip

    #app.vm.hostname = "app.local"

    [
      'express-api-skeleton'
    ].each { |project|
      app.vm.synced_folder "../#{project}", "/usr/local/#{project}/src", type: "nfs"
    }

    # needed to compile binaries
    # app.vm.synced_folder "../red5-live", "/src/red5-live", type: "nfs"

    app.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/vagrant.yml"
      ansible.verbose = 'vvvv'
      ansible.groups = {
        "common-server" => ["app"],
        "skeleton-app" => ["app"]
      }
    end

    # Always restart supervisorctl because of race condition with shared directories
    app.vm.provision "shell", run: "always" do |shell|
      shell.inline = <<-SHELL
while [ ! -S /var/run/supervisor.sock ]
do
  sleep 2
done
sudo supervisorctl restart all
SHELL
    end
  end

end
