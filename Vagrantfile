# -*- mode: ruby -*-
# vi: set ft=ruby :
ip="192.168.18.55"
APP_URL="http://#{ip}"
# print "There's an APP_URL of #{APP_URL} ... hope this works. \n"

$script_install = <<SCRIPT
echo 'export APP_URL='$APP_URL >> .bash_profile
curl -fsSL https://get.docker.com/ | sudo sh
sudo service docker start
sudo groupadd docker
sudo usermod -aG docker root
sudo usermod -aG docker vagrant
newgrp - docker << DSPACE
groups
sudo chkconfig docker on
DSPACE
sudo yum update -y
sudo yum install git -y
cd /home/vagrant
git clone https://github.com/open-oni/open-oni.git
echo 'export APP_URL='$APP_URL >> /home/vagrant/.bash_profile
SCRIPT

$script_dev_run = <<DEVRUN
cd /home/vagrant/open-oni
echo "================================================================================================="
echo "Let's setup some containers. Here's our APP_URL: $APP_URL"
echo "================================================================================================="
sg - docker "./docker/dev.sh"
DEVRUN

$script_sample_load = <<SAMPLELOAD
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

echo "================================================================================================="
echo 'Verifying Django Build'
echo "================================================================================================="
while [[ -d "/home/vagrant/open-oni/ENV/build" ]] || [[ $COUNTER -gt 151 ]]
	do 
		sleep 2
		echo 'Waiting for Django. Packages left to build:' "$(ls -d1 "/home/vagrant/open-oni/ENV/build/*/" | grep -c .)"
		let COUNTER=COUNTER+1
	done
echo "Let's give Django a moment. (10 seconds)"
sleep 10
while read -r BATCH <&3
	do 
		echo $(date) $BATCH" - Batch Loading Starting" >> /home/vagrant/load-$BATCH.log
		sg - docker "docker exec -i openoni-dev /load_batch.sh ""\$BATCH"" 2>&1|tee -a /home/vagrant/load-\$BATCH.log"
		echo $(date) $BATCH" - Batch Loading Completed" >> /home/vagrant/load-$BATCH.log
	done 3</home/vagrant/batches-to-load.txt
SAMPLELOAD

Vagrant.configure(2) do |config|
  # config.vbguest.auto_update = false
  # if Vagrant.has_plugin?("vagrant-cachier")
  #   config.cache.scope = :box
  # end
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: "#{ip}"
  config.vm.provision "install", 
  	type: "shell",
  	env: {"APP_URL" => "#{APP_URL}"}, 
  	privileged: FALSE, 
  	inline: $script_install
  config.vm.provision "devrun",
    type: "shell", 
  	env: {"APP_URL" => "#{APP_URL}"}, 
  	privileged: FALSE, 
  	inline: $script_dev_run,
    run: "always"
  config.vm.provision "sampleload", 
    type: "shell",
  	env: {"APP_URL" => "#{APP_URL}"}, 
  	privileged: FALSE, 
  	inline: $script_sample_load
end
