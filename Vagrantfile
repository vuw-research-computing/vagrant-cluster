
$script = <<-SCRIPT
  set -x
  if [[ ! -e /etc/.provisioned ]]; then
    echo "192.168.100.12    login" >> /etc/hosts
    echo "192.168.100.13    node01" >> /etc/hosts
    echo "192.168.100.14    node02" >> /etc/hosts
    # we only generate the key on one of the nodes
    if [[ ! -e /vagrant/id_rsa ]]; then
      ssh-keygen -t rsa -f /vagrant/id_rsa -N ""
    fi
    install -m 600 -o vagrant -g vagrant /vagrant/id_rsa /home/vagrant/.ssh/
    # the extra 'echo' is needed because Vagrant inserts its own key without a
    # newline at the end
    (echo; cat /vagrant/id_rsa.pub) >> /home/vagrant/.ssh/authorized_keys
    # add EPEL and CAD RPM repos and install slurm
    yum -y install epel-release
    [[ -e /slurm/cad.repo ]] && cp /slurm/cad.repo /etc/yum.repos.d/
    yum -y install slurm
    # we only generate the munge key once
    if [[ ! -e /vagrant/munge.key ]]; then
      /usr/sbin/create-munge-key
      cp /etc/munge/munge.key /vagrant
    else
      cp /vagrant/munge.key /etc/munge
    fi
    chown munge.munge /etc/munge/munge.key
    sudo systemctl restart munge
    touch /etc/.provisioned
  fi
SCRIPT


Vagrant.configure("2") do |cluster|
  #basic setup
  cluster.vm.box = "centos/7"
  cluster.vm.provider 'virtualbox' do |vb|
      vb.memory = 256
      vb.cpus = 1
  end
  cluster.vm.synced_folder "slurm/", "/slurm"
  cluster.vm.provision "shell", inline: $script
  #define nodes
  cluster.vm.define "login" do |login|
      login.vm.hostname = "login"
      login.vm.network "private_network", ip: "192.168.100.12", virtualbox_intnet: "clusternet"
  end
  cluster.vm.define "node01" do |node01|
      node01.vm.hostname = "node01"
      node01.vm.network "private_network", ip: "192.168.100.13", virtualbox_intnet: "clusternet"
  end
  cluster.vm.define "node02" do |node02|
      node02.vm.hostname = "node02"
      node02.vm.network "private_network", ip: "192.168.100.14", virtualbox_intnet: "clusternet"
  end
end
