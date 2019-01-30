Vagrant.configure("2") do |configLS|

 configLS.vm.provision "file", source: File.expand_path("../id_rsa.pub", __FILE__), destination: "~/.ssh/authorized_keys"
 configLS.vm.provision "file", source: File.expand_path("~/.aws/credentials", __FILE__), destination: "~/.aws/credentials"
 configLS.ssh.insert_key = false
 configLS.ssh.private_key_path = [File.expand_path("../id_rsa", __FILE__), "~/.vagrant.d/insecure_private_key"]

 configLS.vm.box = "centos/7"

 configLS.vm.define "serverless-grafana"
 configLS.vm.hostname = "serverlessgrafana"
 configLS.vm.network :private_network, ip:"192.168.77.31"
 configLS.vm.box_check_update = false
 configLS.vm.provider "virtualbox" do |v|
  v.memory = 4096
  v.cpus = 1
 end

 $script = <<-SCRIPT
  yum -y update
  curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
  yum -y install nodejs
  yum -y install initscripts fontconfig
  wget https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
  rpm -Uvh grafana-5.4.2-1.x86_64.rpm
  service grafana-server start
  /sbin/chkconfig --add grafana-server
  npm install -g serverless
 SCRIPT

 configLS.vm.provision "shell", inline: $script

 for port in 4567..4582 do
  configLS.vm.network "forwarded_port", guest: port, host: port
 end
end
