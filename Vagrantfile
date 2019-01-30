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

 configLS.vm.provision "shell" do |configShell|
  configShell.inline = "yum -y update"
  configShell.inline = "curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -"
  configShell.inline = "yum install nodejs"
  configShell.inline = "yum -y install initscripts fontconfig"
  configShell.inline = "yum install https://grafana.com/grafana/download?platform=linux"
  configShell.inline = "service grafana-server start"
  configShell.inline = "/sbin/chkconfig --add grafana-server"
  configShell.inline = "npm install -g serverless"
 end
 for port in 4567..4582 do
  configLS.vm.network "forwarded_port", guest: port, host: port
 end
end
