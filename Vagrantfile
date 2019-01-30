require 'yaml'

current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/vars.yaml")
vagrant_config = configs['variables'][configs['variables']['use']]

Vagrant.configure("2") do |configLS|

 configLS.vm.provision "file", source: File.expand_path("../id_rsa.pub", __FILE__), destination: "~/.ssh/authorized_keys"
 configLS.vm.provision "file", source: File.expand_path("~/.aws/credentials", __FILE__), destination: "~/.aws/credentials"
 configLS.ssh.insert_key = false
 configLS.ssh.private_key_path = [File.expand_path("../id_rsa", __FILE__), "~/.vagrant.d/insecure_private_key"]

 configLS.vm.box = vagrant_config['OS']

 configLS.vm.define "serverless-grafana"
 configLS.vm.hostname = "serverlessgrafana"
 configLS.vm.network :private_network, ip:vagrant_config['PRIVATE_IP']
 configLS.vm.box_check_update = false
 configLS.vm.provider "virtualbox" do |v|
  v.memory = vagrant_config['MEMORIA_RAM']
  v.cpus = 1
 end

 $script = <<-SCRIPT
  yum -y update
  curl -sL vagrant_config['NODE_URL'] | sudo bash -
  yum -y install nodejs
  yum -y install initscripts fontconfig wget urw-fonts
  yum -y install make python python-pip python-devel java mvn
  if [ ! -f get-pip-py ]
  then
    curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
  fi
  python get-pip.py
  if [ ! -f grafana-5.4.2-1.x86_64.rpm ]
  then
    wget https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
  fi
  rpm -Uvh grafana-5.4.2-1.x86_64.rpm
  service grafana-server start
  /sbin/chkconfig --add grafana-server
  npm install -g serverless
  pip install localstack
  export SERVICES=es
  export DEFAULT_REGION=eu-west-1
 SCRIPT

 configLS.vm.provision "shell", inline: $script

 for port in 4567..4582 do
  configLS.vm.network "forwarded_port", guest: port, host: port
 end
end
