# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Define the number of nodes for the fuzzing cluster
MAMBA_CLUSTER_SIZE = 2

# Define Fuzzing Target 
FUZZ_TARGET = "objdump"

# Definte Packages to Install
# For Ubuntu find out what package provides the target by:
# sudo apt-get -y install apt-file
# apt-file update
# apt-file search #{FUZZ_TARGET}
TARGET_PACKAGES = "binutils"

# Define Fuzzing Target options
FUZZ_TARG_OPTS = "cli#-x -D"

# Define the fuzzer type
FUZZ_TYPE = "SimpleGeneticAlgorithm"
FUZZER_NAME = "my-new-fuzzer"
FUZZER_TIMEOUT = "5"

# Genetic Parameters
CrossoverRate = "0.1"
MutationRate = "0.2"
MaxGens = "5"
PopSize = "1314"
FitnessFunction = "R"
# Note: Be sure to escape /'s
InitialPop = 'tests\/testset.zip'
Disassembly = "disassemblies/libbfd-2.24-system.so.fz"
# /usr/lib/libbfd-2.24-system.so
#
$chefUpdate = <<CHEF
  set -x
  apt-get update
  /opt/ruby/bin/gem update chef --no-ri --no-rdoc
  apt-get install -y gnupg
  apt-get install -y man-db
CHEF

$lldb = <<LLDB
set -x
# Install LLDB APT Repository
wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key | apt-key add -
echo "deb http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main" > /etc/apt/sources.list.d/llvm.list
echo "deb-src http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main" >> /etc/apt/sources.list.d/llvm.list
apt-get update

# Install llvm/lldb
apt-get -y install clang-3.4 clang-3.4-doc libclang-common-3.4-dev libclang-3.4-dev libclang1-3.4 libclang1-3.4-dbg libllvm-3.4-ocaml-dev libllvm3.4 libllvm3.4-dbg lldb-3.4 llvm-3.4 llvm-3.4-dev llvm-3.4-doc llvm-3.4-examples llvm-3.4-runtime clang-modernize-3.4 clang-format-3.4 python-clang-3.4 lldb-3.4-dev

# Fix lldb python issue 
ln -sf /usr/lib/llvm-3.4/lib/liblldb.so.1  /usr/lib/python2.7/dist-packages/lldb/_lldb.so

# Make a symlink for the lldb executable
ln -s /usr/bin/lldb-3.4 /usr/local/bin/lldb
LLDB

# Erlang installation
$erlang = <<ERLANG
set -x
# Grab the package for the repo
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
dpkg -i erlang-solutions_1.0_all.deb

# Install erlang
apt-get -y update 
apt-get -y install erlang
apt-get -y install zip
apt-get install -y xsltproc
ERLANG

# Mongodb installation
$mongodb = <<MONGODB
set -x
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' > /etc/apt/sources.list.d/mongodb.list
apt-get update
apt-get -y install mongodb-org
service mongod stop
MONGODB

# Python dependencies for mamba
$mambaPython = <<MAMBAPY
set -x
apt-get -y install git 
apt-get -y install python-pip
apt-get -y install python-dev
apt-get -y install libevent-dev
apt-get -y install libxml2-dev libxslt-dev python-dev

pip install uwsgi
pip install numpy
pip install lxml
MAMBAPY

$crashDetection = <<CRASH
set -x
if [ -f /etc/default/apport ] ; then
  # Disable apport
  sed -i.bak s/enabled=1/enabled=0/g /etc/default/apport
  service apport stop
else
  # Create the missing directory since apport not installed
  mkdir -p /var/crash
  chmod 777 /var/crash
fi

# Setup core files
echo "/var/crash/core.%e.%p.%h.%t" > /proc/sys/kernel/core_pattern
CRASH

$mambaInstall = <<MAMBAINST
set -x
gem install bundle
gem install rake-compiler
gem install jeweler
cd $HOME
git clone -b simple-ga-distrib https://github.com/rogwfu/mamba.git
cd mamba
bundle install
rake install
MAMBAINST

$rubyInstall = <<RVM
  set -x
  echo "install: --no-rdoc --no-ri" > ~/.gemrc 
  echo "update:  --no-rdoc --no-ri" >> ~/.gemrc
  gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
  sudo apt-get -y install curl
  curl -sSL https://get.rvm.io | bash
  source ~/.bash_profile
  rvm install 2.1.4
  rvm use 2.1.4
  rvm gemset create mamba
  rvm use --default 2.1.4@mamba
RVM

$fuzzer = <<FUZZER
  set -x
  export CLUSTER_SIZE=#{MAMBA_CLUSTER_SIZE}
  # Setup the new fuzzer
  sudo apt-get -y install #{TARGET_PACKAGES}
  
  # Check for the main fuzzer
  if [[ "$HOSTNAME" == "mamba-fuzzer-0" ]] ; then
	echo "Running inside the mamba-fuzzer-0"

	# Is this a distributed job?
	if [[ $CLUSTER_SIZE -gt 1 ]] ; then
	  mamba create -a `which #{FUZZ_TARGET}` -e '#{FUZZ_TARG_OPTS}' -y #{FUZZ_TYPE} -d #{FUZZER_NAME} -t #{FUZZER_TIMEOUT}
	else
	  mamba create -a `which #{FUZZ_TARGET}` -e '#{FUZZ_TARG_OPTS}' -y #{FUZZ_TYPE} #{FUZZER_NAME} -t #{FUZZER_TIMEOUT}
	fi

	cd #{FUZZER_NAME}

	# Copy the initial testset
	cp /vagrant/testcases/* tests/

	# Copy the disassembly
	cp /vagrant/disassemblies/* disassemblies/

	# Fixup the genetic parameters
	sed -i 's/Crossover Rate: 0.2/Crossover Rate: #{CrossoverRate}/' ./configs/*Genetic* 
	sed -i 's/Mutation Rate: 0.5/Mutation Rate: #{MutationRate}/' ./configs/*Genetic* 
	sed -i 's/Maximum Generations: 2/Maximum Generations: #{MaxGens}/' ./configs/*Genetic* 
	sed -i 's/Population Size: 4/Population Size: #{PopSize}/' ./configs/*Genetic* 
	sed -i 's/Fitness Function: As/Fitness Function: #{FitnessFunction}/' ./configs/*Genetic* 
#	sed -i 's/Initial Population: tests\/testset.zip/Initial Population: #{InitialPop}/' ./configs/*Genetic* 
	sed -i '8d' ./configs/*Genetic*
	sed -i '7i Disassembly: #{Disassembly}' ./configs/*Genetic* 

	# Package up the fuzzing configuration
  
	if [[ $CLUSTER_SIZE -gt 1 ]] ; then
	  thor fuzz:package
	  cp fz* /vagrant/
	  nohup thor distrib:start
	  sleep 10
	fi

	nohup thor fuzz:start
  else
	mamba create -a `which #{FUZZ_TARGET}` -e '#{FUZZ_TARG_OPTS}' -y #{FUZZ_TYPE} -d #{FUZZER_NAME} -t #{FUZZER_TIMEOUT}

	cd #{FUZZER_NAME}

	cp /vagrant/fz* .
	thor fuzz:unpackage fz*
	nohup thor fuzz:start
  fi

FUZZER

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "chef/ubuntu-14.04"

  config.vm.provider "vmware_fusion" do |v|
	  v.gui = false 
	  v.vmx["memsize"] = "4096"
	  v.vmx["numvcpus"] = "2"
  end

  config.vm.provision "shell", inline: $chefUpdate
  config.vm.provision "shell", inline: $lldb
  config.vm.provision "shell", inline: $erlang
  config.vm.provision "shell", inline: $mongodb
  config.vm.provision "shell", inline: $mambaPython
  config.vm.provision "shell", inline: $crashDetection

  # For now install mamba from source, no official release yet
  config.vm.provision "shell", :privileged => false, :inline => $rubyInstall
  config.vm.provision "shell", :privileged => false, :inline => $mambaInstall

  # Dynamically define multiple fuzzing boxes for the cluster 
  MAMBA_CLUSTER_SIZE.times do |nodeNum|
	config.vm.define "mamba-fuzzer-#{nodeNum}" do |node|
	  node.vm.hostname = "mamba-fuzzer-#{nodeNum}" 
	  node.vm.provision "shell", :privileged => false, :inline => $fuzzer
	end
  end

end
