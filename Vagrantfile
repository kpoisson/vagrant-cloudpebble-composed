# Huge thanks and credit to nrgestle work on Vagrant, here's his repo:
# https://github.com/ngerstle/cloudpebble-vagrantfile
#
include_vagrantfile = File.expand_path("../include/_Vagrantfile", __FILE__)
load include_vagrantfile if File.exist?(include_vagrantfile)

Vagrant.configure("2") do |config|
  config.vm.box = "cxtlabs/vagrant-ubuntu-16.04-mate"

  # Port forwardings are temporarily disabled because I was only able to make
  # Cloudpebble work on the guest OS
  #
  #config.vm.network "forwarded_port", guest: 80, host:80, protocol: "tcp"
  #config.vm.network "forwarded_port", guest: 80, host:80, protocol: "udp"
  #config.vm.network "forwarded_port", guest: 8002, host:8002, protocol: "tcp"
  #config.vm.network "forwarded_port", guest: 8003, host:8003, protocol: "udp"


  config.vm.provider "virtualbox" do |vb|
    vb.name = "cloudpebble"
    vb.memory = 2048
    vb.cpus = 2
    #vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
	vb.gui = true
  end

  # Preparing the Vagrant box using a provisioning script that will install CloudPebble and all its dependencies
  $provisioningscript = <<SCRIPT
cd ~/
sudo apt-get remove docker docker-engine docker.io || true
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
# Pebble emulator dependencies
sudo apt-get install libsdl1.2debian libfdt1 libpixman-1-0 -y
sudo apt-get install docker-ce -y
sudo docker run hello-world
sudo apt-get install docker-compose -y
sudo apt remove nodejs npm || true
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs

git clone https://github.com/ltpitt/vagrant-cloudpebble-composed.git cloudpebble-composed
cd cloudpebble-composed
git clone https://github.com/ltpitt/cloudpebble.git
git clone https://github.com/ltpitt/cloudpebble-qemu-controller.git
git clone https://github.com/ltpitt/cloudpebble-ycmd-proxy.git
cd cloudpebble/ext
git clone https://github.com/pebble/pebblejs.git

cd ~/cloudpebble-composed
#sed -i 's/10.0.2.15/172.17.0.1/g' docker-compose.yml
chmod u+x dev_setup.sh

#TODO need to update node version, nodejs gpg keys?
sed -i 's/NODE_VERSION=4.2.3/NODE_VERSION=4.7.0/g' cloudpebble/Dockerfile
echo "adding GPG keys"
sed -i 's/DD8F2338BAE7501E3DD5AC78C273792F7D83545D/DD8F2338BAE7501E3DD5AC78C273792F7D83545D C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 B9AE9905FFD7803F25714661B63B535A4C206CA9 56730D5401028683275BD23C23EFEFE93C4CFFFE 77984A986EBC2AA786BC0F66B01FBB92821C587A/g'  cloudpebble/Dockerfile
sudo ./dev_setup.sh
SCRIPT

  config.vm.provision "shell", run: "once" do |s|
    s.inline= $provisioningscript
  end

  # Always run docker-compose up on vagrant up
  config.vm.provision "shell", run: "always" do |s|
    s.inline = "cd ~/cloudpebble-composed && sudo docker-compose up -d"
  end

end
