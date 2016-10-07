# -*- mode: ruby -*-
# vi: set ft=ruby :
ip="192.168.18.55"
APP_URL="http://#{ip}"
# print "There's an APP_URL of #{APP_URL} ... hope this works. \n"

$script_install = <<SCRIPT
echo $APP_URL
echo 'export APP_URL='$APP_URL >> .bash_profile
curl -fsSL https://get.docker.com/ | sudo sh
sudo service docker start
sudo groupadd docker
user=$(whoami)
sudo usermod -aG docker root
sudo usermod -aG docker vagrant
newgrp - docker << DSPACE
echo "Let's check our groups for the user $(whoami)"
echo "It looks like $(whoami) is in the following groups: "
groups
echo "APP_URL is set to be: "$APP_URL
echo "I'll let that sink in for a moment."
sudo yum update -y
sudo yum install git -y
sudo chkconfig docker on
service docker status
cd /home/vagrant
git clone https://github.com/open-oni/open-oni.git
cd /home/vagrant/open-oni
echo 'export APP_URL='$APP_URL >> .bash_profile
echo "================================================================================================="
echo "Let's setup some containers. Here's our APP_URL: $APP_URL"
echo "================================================================================================="
./docker/dev.sh
echo "================================================================================================="
echo "Alright.... The hard part is done. We're in the home stretch! Let's load some sample data."
echo "================================================================================================="
cd /home/vagrant
git clone https://github.com/open-oni/sample-data.git
echo 'Moving sample data to be processed'
sudo chown -R vagrant /home/vagrant/
mv /home/vagrant/sample-data/batch_* /home/vagrant/open-oni/docker/data/batches
echo 'logging sample batches to be processed'
ls -1 /home/vagrant/open-oni/docker/data/batches > /home/vagrant/batches-to-load.txt
cd /home/vagrant/open-oni
echo 'Starting django build'
DSPACE
while [ -d "/home/vagrant/open-oni/ENV/build" ]
	do 
		sleep 2
		echo 'Waiting for django. Packages left to build:' $(ls -d1 /home/vagrant/open-oni/ENV/build/*/ | grep -c .)
	done
sleep 10
while read -r BATCH <&3
	do 
		echo $(date) $BATCH" - Batch Loading Starting" >> /home/vagrant/load-$BATCH.log
		sg - docker "docker exec -i openoni-dev /load_batch.sh ""\$BATCH"" 2>&1|tee -a /home/vagrant/load-\$BATCH.log"
		echo $(date) $BATCH" - Batch Loading Completed" >> /home/vagrant/load-$BATCH.log
	done 3</home/vagrant/batches-to-load.txt
SCRIPT

Vagrant.configure(2) do |config|
  # config.vbguest.auto_update = false
  # if Vagrant.has_plugin?("vagrant-cachier")
  #   config.cache.scope = :box
  # end
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: "#{ip}"
  config.vm.provision "shell", env: {"APP_URL" => "#{APP_URL}"}, privileged: FALSE, inline: $script_install
end
