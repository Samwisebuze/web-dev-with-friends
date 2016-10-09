# -*- mode: ruby -*-
# vi: set ft=ruby :

$ORIGIN    = "git@github.com:QuickFixes"
$REPO_NAME = "web-dev-with-friends"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "webdev-jessie"
  config.vm.define :debian_jessie

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine.
  config.vm.network "forwarded_port", guest: 80, host: 9980, id: "http"
  config.vm.network "forwarded_port", guest: 81, host: 9981, id: "http-alt"
  # See https://github.com/mitchellh/vagrant/issues/3232#issuecomment-48994417
  config.vm.network "forwarded_port", guest: 22, host: 9922, id: "ssh"
  config.vm.network "forwarded_port", guest: 5000, host: 55000, id: "flask-app"

  # Forward SSH authentication credentials to the VM; disabled for now because
  # it gets confusing when things like 'git clone' work inside the VM when you
  # didn't expect them to.
  # See http://docs.vagrantup.com/v2/vagrantfile/ssh_settings.html
  #config.ssh.forward_agent = true

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
  config.vm.synced_folder "todolist", "/var/www/todolist"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider "virtualbox" do |vb|
    # Uncomment to run with a GUI (so you can see what's going on):
    # See also: https://docs.vagrantup.com/v2/virtualbox/configuration.html
    #vb.gui = true
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
    # Might want these if you were, say, running a memory hog Java app, or X
    # with a graphical desktop environment
    #vb.memory = "1024"
    #vb.customize ["modifyvm", :id, "--vram", "32"]
  end

  # Provision from within the VM itself by copying the GitLab deploy key pair
  # to the guest (fixing permissions on the private key), then cloning the
  # GitLab repo and running the 'self-provision.sh' script on the VM.
  files = {
    # This seems somewhat fruitless because of the new hashed host keys
    'provisioning/files/origin_host_keys'            => '~/.ssh/known_hosts',
    'provisioning/files/id_rsa-webdev-vm-deploy'     => '~/.ssh/id_rsa',
    'provisioning/files/id_rsa-webdev-vm-deploy.pub' => '~/.ssh/id_rsa.pub'
  }
  files.each { |s,d| config.vm.provision "file", source: s, destination: d }

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # OpenSSH gets (rightfully) upset if everyone can read your private key,
    # and in fact the 'git clone' will fail as a result unless you are
    # running a key agent on the host OS, and that public key has already
    # been added to your GitHub profile.
    chmod 0600 ~/.ssh/id_rsa
    cd ~vagrant
    test -d devel || mkdir devel && cd devel

    # Check if the repo for the VM configuration (where this Vagrantfile comes
    # from) already exists *on* the VM; clone it if not.
    if [ ! -d #$REPO_NAME ]; then
        git clone -q #$ORIGIN/#$REPO_NAME || exit $?
        cd #$REPO_NAME
    else
        cd #$REPO_NAME
        git pull --quiet || exit $?
    fi

    echo "==> Running provisioning playbooks with Ansible <=="
    bin/self-provision.sh
  SHELL

end
# vim: sw=2 ts=2 expandtab