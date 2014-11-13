# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$lldb = <<LLDB
# Install LLDB APT Repository
wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key | apt-key add -
echo "deb http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main" > /etc/apt/llvm.list
echo "deb-src http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main" >> /etc/apt/llvm.list
apt-get update

# Install llvm
apt-get -y install clang-3.4 clang-3.4-doc libclang-common-3.4-dev libclang-3.4-dev libclang1-3.4 libclang1-3.4-dbg libllvm-3.4-ocaml-dev libllvm3.4 libllvm3.4-dbg lldb-3.4 llvm-3.4 llvm-3.4-dev llvm-3.4-doc llvm-3.4-examples llvm-3.4-runtime clang-modernize-3.4 clang-format-3.4 python-clang-3.4 lldb-3.4-dev

# Fix lldb python issue 
ln -sf /usr/lib/llvm-3.4/lib/liblldb.so.1  /usr/lib/python2.7/dist-packages/lldb/_lldb.so

# Make a symlink for the lldb executable
ln -s /usr/bin/lldb-3.4 /usr/local/bin/lldb
LLDB

# Erlang installation
$erlang = <<ERLANG
# Grab the package for the repo
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
dpkg -i erlang-solutions_1.0_all.deb

# Install erlang
apt-get -y update 
apt-get install erlang
apt-get install zip
ERLANG

# Mongodb installation
$mongodb = <<MONGODB
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' > /etc/apt/sources.list.d/mongodb.list
apt-get update
apt-get install -y mongodb-org
MONGODB

# Python dependencies for mamba
$mambaPython = <<MAMBAPY
apt-get -y install python-pip
apt-get -y install python-dev
apt-get -y install libevent-dev
apt-get -y install libxml2-dev libxslt-dev python-dev

pip install uwsgi
pip install numpy
pip install lxml
MAMBAPY

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "hashicorp/precise64"

  config.vm.provision "shell", inline: $lldb
  config.vm.provision "shell", inline: $erlang
  config.vm.provision "shell", inline: $mongodb
  config.vm.provision "shell", inline: $mambaPython

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  end
end