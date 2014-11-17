# -*- mode: ruby -*-
# vi: set ft=ruby :

# Boostrap Script
$script = <<SCRIPT

# Update & Install
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list
apt-get update
apt-get install -y build-essential python-setuptools git curl mongodb-10gen=2.4.5 tcl8.5
easy_install pip && pip install setuptools --no-use-wheel --upgrade && pip install dotcloud

# Make build directory
mkdir ~/build; cd ~/build

# Build and install Redis
echo "Installing Redis..."
wget http://redis.googlecode.com/files/redis-2.4.11.tar.gz
tar xzvf redis-2.4.11.tar.gz
cd ~/build/redis-2.4.11/
make
make install
make test

sed -i 's|daemonize no|daemonize yes|g' redis.conf
sed -i 's|# bind 127.0.0.1|bind 127.0.0.1|g' redis.conf
sed -i 's|dir ./|dir /var/lib/redis|g' redis.conf
mkdir /etc/redis
mv redis.conf /etc/redis/redis.conf

wget https://gist.githubusercontent.com/lsbardel/257298/raw/d48b84d89289df39eaddc53f1e9a918f776b3074/redis-server-for-init.d-startup
sed -i 's|DAEMON=/usr/bin/redis-server|DAEMON=/usr/local/bin/redis-server|g' redis-server-for-init.d-startup
mv redis-server-for-init.d-startup /etc/init.d/redis-server
chmod +x /etc/init.d/redis-server

useradd redis
mkdir -p /var/lib/redis
mkdir -p /var/log/redis
chown redis.redis /var/lib/redis
chown redis.redis /var/log/redis

update-rc.d redis-server defaults
service redis-server start

# Vagratnt Environment Varaibles
echo "Setting environment variables..."

echo "export NODE_ENV=development" >> /home/vagrant/.bashrc
echo "export PORT_WWW=8080" >> /home/vagrant/.bashrc
echo "export MONGO_URI=mongodb://localhost:27017/test" >> /home/vagrant/.bashrc
echo "export DOTCLOUD_CACHE_REDIS_HOST=localhost" >> /home/vagrant/.bashrc
echo "export DOTCLOUD_CACHE_REDIS_PORT=6379" >> /home/vagrant/.bashrc
echo "\n\n" >> /home/vagrant/.bashrc

# NodeJS via NVM
echo "Installing NVM..."
export HOME=/home/vagrant
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
echo "source ~/.nvm/nvm.sh" >> /home/vagrant/.bashrc
source /home/vagrant/.nvm/nvm.sh
#nvm install 0.8
nvm install 0.10
#nvm install 0.11
chown -R vagrant:vagrant /home/vagrant/.nvm
export HOME=/home/root

# NPM package install
echo "Installing NPM packages..."
echo "PATH=$PATH:/vagrant/node_modules/.bin" >> /home/vagrant/.bashrc
PATH=$PATH:/vagrant/node_modules/.bin
cd /vagrant && rm -rf node_modules
[ -f package.json ] && npm install

echo "cd /vagrant" >> /home/vagrant/.bashrc

SCRIPT
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "ubuntu/trusty32"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # The shell provisioner allows you to upload and execute a script as the root
  # user within the guest machine.
  config.vm.provision "shell", inline: $script

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:

  config.vm.provider "virtualbox" do |vb|
    # Don't boot with headless mode
    #vb.gui = true

    # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "384", "--cpus", "2"]
  end

  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with CFEngine. CFEngine Community packages are
  # automatically installed. For example, configure the host as a
  # policy server and optionally a policy file to run:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.am_policy_hub = true
  #   # cf.run_file = "motd.cf"
  # end
  #
  # You can also configure and bootstrap a client to an existing
  # policy server:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.policy_server_address = "10.0.2.15"
  # end

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file default.pp in the manifests_path directory.
  #
  # config.vm.provision "puppet" do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "default.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
