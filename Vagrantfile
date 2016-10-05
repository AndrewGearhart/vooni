# -*- mode: ruby -*-
# vi: set ft=ruby :
$ip = "192.168.18.55"
$script = <<SCRIPT
sudo yum update -y
sudo yum install git -y
curl -fsSL https://get.docker.com/ | sh
sudo service docker start
sudo groupadd docker
sudo usermod -aG docker $(whoami)
newgrp docker
sudo chkconfig docker on
cd $HOME
git clone https://github.com/open-oni/open-oni.git
cd open-oni
echo $APP_URL
export APP_HOST="$ip"
export APP_URL="http://$APP_HOST"
./docker/dev.sh
cd $HOME 
git clone https://github.com/open-oni/sample-data.git
echo 'Moving sample data to be processed'
mv $HOME/sample-data/batch_* $HOME/open-oni/docker/data/batches
echo 'logging sample batches to be processed'
ls -1 $HOME/open-oni/docker/data/batches > $HOME/batches-to-load.txt
cd $HOME/open-oni
echo 'Starting django build'
while [ -d "$HOME/open-oni/ENV/build" ]
	do 
		sleep 2
		echo 'Waiting for django to finish building'
	done
while read -r BATCH <&3
	do 
		echo $(date) $BATCH " - Batch Loading Starting" >> $HOME/load-$BATCH.log
		docker exec -i openoni-dev /load_batch.sh $BATCH 2>&1 | tee -a $HOME/load-$BATCH.log
		echo $(date) $BATCH " - Batch Loading Completed" >> $HOME/load-$BATCH.log
	done 3<$HOME/batches-to-load.txt
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: $ip
  config.vm.provision "shell", env: {"APP_URL" => "$APP_URL"}, inline: $script
end
