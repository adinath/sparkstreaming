# -*- mode: ruby -*-
# vi: set ft=ruby :

$global = <<SCRIPT

PROVISIOING_DIR="/vagrant/provisioning"
ARCHIVE_DIR="${PROVISIOING_DIR}/archive/"

function resourceExists {
	FILE=$ARCHIVE_DIR/$1
	if [ -e $FILE ]
	then
		return 0
	else
		return 1
	fi
}


function installLocal {
	echo "install from local file $1"
	FILE=$ARCHIVE_DIR/$1

  echo "unzipping $FILE"
	tar -xzf $FILE
}

function installRemote {
	echo "install from remote file $2 with $3"
	wget $3 -O $ARCHIVE_DIR/$1 -L $2

  echo "unzipping $ARCHIVE_DIR/$1"
	tar -xzf $ARCHIVE_DIR/$1
}

function install {
  local resourceName=$1
  local remoteResourceURL=$2
  local wgetExtraOptions=$3
	if resourceExists $resourceName; then
		installLocal $resourceName
	else
    installRemote $resourceName $remoteResourceURL $wgetExtraOptions
	fi
}

function setupPasswordlessSSH() {
  cp /vagrant/provisioning/ssh/id_rsa /home/vagrant/.ssh/id_rsa
  chmod 0600 /home/vagrant/.ssh/id_rsa
  #allow ssh passwordless
  cat /vagrant/provisioning/ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
}

function setupHostsFile() {
  #populate /etc/hosts
  sudo cp /vagrant/provisioning/hosts /etc/hosts
}

function installJava() {
    
  if ! resourceExists "jdk-8u201-linux-x64.tar.gz"; then
    echo "getting jdk-8u201-linux-x64.tar.gz"
    wget -O $ARCHIVE_DIR/jdk-8u201-linux-x64.tar.gz --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz"
  fi
  echo "unzipping jdk-8u201-linux-x64.tar.gz"
  tar -xzf $ARCHIVE_DIR/jdk-8u201-linux-x64.tar.gz
  
  echo "moving jdk1.8.0_201 to jdk"
  rm -rf jdk
  mv -f jdk1.8.0_201 jdk
}

setupPasswordlessSSH
setupHostsFile

installJava

install "hadoop-2.6.0.tar.gz" "http://archive.apache.org/dist/hadoop/core/hadoop-2.6.0/hadoop-2.6.0.tar.gz"

echo "moving hadoop-2.6.0 to hadoop"
rm -rf hadoop
mv -f hadoop-2.6.0 hadoop

echo "copying configuration files"
cp -f ${PROVISIOING_DIR}/hadoop/*.xml ./hadoop/etc/hadoop/
cp -f ${PROVISIOING_DIR}/hadoop/slaves ./hadoop/etc/hadoop/

sed -i "s@\${JAVA_HOME}@/home/vagrant/jdk@g" /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh

echo "export JAVA_HOME=/home/vagrant/jdk" > /home/vagrant/.bashrc
echo "export PATH=$JAVA_HOME/bin:/home/vagrant/hadoop/bin:/home/vagrant/hadoop/sbin:$PATH" > /home/vagrant/.bashrc
source /home/vagrant/.bashrc

#PATH=/home/hadoop/hadoop/bin:/home/hadoop/hadoop/sbin:$PATH
#update hadoop-env.sh to include $JAVA_HOME
#end script


SCRIPT

#PATH=/home/vagrant/hadoop/bin:/home/vagrant/hadoop/sbin:$PATH
# export JAVA_HOME=/home/vagrant/jdk
# export PATH=$JAVA_HOME/bin:/home/vagrant/hadoop/bin:/home/vagrant/hadoop/sbin:$PATH
# export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
# cat /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh | sed -e "s@\${JAVA_HOME}@/home/vagrant/java" > /home/vagrant/hadoop/etc/hadoop/hadoop-env-temp.sh
# cp -f /home/vagrant/hadoop/etc/hadoop/hadoop-env-temp.sh /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh
# rm /home/vagrant/hadoop/etc/hadoop/hadoop-env-temp.sh
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
  config.vm.box = "ubuntu/trusty64"
  config.vm.provision "shell", privileged: false, inline: $global
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

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

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.define "namenode" do |web|
    web.vm.box = "ubuntu/trusty64"
    web.vm.hostname = 'namenode'

    web.vm.network :private_network, ip: "192.168.56.101"

    web.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", 2048]
      v.customize ["modifyvm", :id, "--name", "namenode"]
    end
  end

  config.vm.define "datanode1" do |db|
    db.vm.box = "ubuntu/trusty64"
    db.vm.hostname = 'datanode1'

    db.vm.network :private_network, ip: "192.168.56.102"

    db.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", 2038]
      v.customize ["modifyvm", :id, "--name", "datanode1"]
    end
  end

  config.vm.define "datanode2" do |db|
    db.vm.box = "ubuntu/trusty64"
    db.vm.hostname = 'datanode2'

    db.vm.network :private_network, ip: "192.168.56.103"

    db.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", 2038]
      v.customize ["modifyvm", :id, "--name", "datanode2"]
    end
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
