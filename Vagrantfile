#require 'yaml'

PRIVATE_IP = "192.168.77.31"
OS = "centos/7"
MEMORIA_RAM = 4096
NUM_CPU = 1
NODE_VERSION="setup_10.x"
NODE_URL = "https://rpm.nodesource.com/#{NODE_VERSION}"
REGION_AWS = "eu-west-1"
GRAFANA_RPM = "grafana-5.4.2-1.x86_64.rpm"

Vagrant.configure("2") do |configLS|

 configLS.vm.provision "file", source: File.expand_path("../id_rsa.pub", __FILE__), destination: "~/.ssh/authorized_keys"
 configLS.vm.provision "file", source: File.expand_path("~/.aws/credentials", __FILE__), destination: "~/.aws/credentials"
 configLS.ssh.insert_key = false
 configLS.ssh.private_key_path = [File.expand_path("../id_rsa", __FILE__), "~/.vagrant.d/insecure_private_key"]

# current_dir    = File.dirname(File.expand_path(__FILE__))
# configs        = YAML.load_file("#{current_dir}/config/vars.yaml")
# vagrant_config = configs['variables'][configs['variables']['use']]

 configLS.vm.box = OS

 configLS.vm.define "serverless-grafana"
 configLS.vm.hostname = "serverlessgrafana"
 configLS.vm.network :private_network, ip:PRIVATE_IP
 configLS.vm.box_check_update = false
 configLS.vm.provider "virtualbox" do |v|
  v.memory = MEMORIA_RAM
  v.cpus = NUM_CPU
 end

 $script = <<-SCRIPT
  yum -y update
  yum -y install initscripts fontconfig wget urw-fonts
  yum -y install make python python-devel
  curl -sL #{NODE_URL} | sudo bash -
  yum -y install nodejs
  yum -y install docker
  if [ ! -f get-pip-py ]
  then
    curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
  fi
  python get-pip.py
  if [ ! -f #{GRAFANA_RPM} ]
  then
    wget https://dl.grafana.com/oss/release/#{GRAFANA_RPM}
    rpm -Uvh #{GRAFANA_RPM}
  fi
  service grafana-server start
  /sbin/chkconfig --add grafana-server
  npm install -g serverless
  echo "***INSTALLING LOCALSTACK***"
  pip install localstack --user
  sudo chown -R vagrant:vagrant /root/.config
  sudo chown -R vagrant:vagrant /root/.local
  echo "***EXPORTING VARIABLES***"
  echo export SERVICES="es" > /etc/profile.d/servicesenv.sh
  echo export DEFAULT_REGION=#{REGION_AWS} > /etc/profile.d/regionenv.sh
  echo export FORCE_NONINTERACTIVE=''>/etc/profile.d/noninteractive.sh
  echo export DATA_DIR='/vagrant'>/etc/profile.d/datadir.sh
  echo export PATH=$PATH:/root/.local/bin > /etc/profile.d/changepath.sh
  chmod 0755 /etc/profile.d/servicesenv.sh
  chmod 0755 /etc/profile.d/regionenv.sh
  chmod 0755 /etc/profile.d/noninteractive.sh
  chmod 0755 /etc/profile.d/datadir.sh
  chmod 0755 /etc/profile.d/changepath.sh
  localstack infra stop
  localstack start --docker
  localstack web
 SCRIPT

 configLS.vm.provision "shell", inline: $script

 for port in 4567..4582 do
  configLS.vm.network "forwarded_port", guest: port, host: port
 end
end
