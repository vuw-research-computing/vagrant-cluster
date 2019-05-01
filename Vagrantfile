
$script = <<-SCRIPT
  set -x
  # Only need to do this once, so check for lock file
  if [[ ! -e /etc/CLUSTER_VAGRANT_UP ]]; then
    echo "192.168.100.12    login" >> /etc/hosts
    echo "192.168.100.13    node01" >> /etc/hosts
    echo "192.168.100.14    node02" >> /etc/hosts
    echo "192.168.100.15    node03" >> /etc/hosts
    # we only generate the key on one of the nodes
    if [[ ! -e /vagrant/slurm/scratch/id_rsa ]]; then
      ssh-keygen -t rsa -f /vagrant/slurm/scratch/id_rsa -N ""
    fi
    install -m 600 -o vagrant -g vagrant /vagrant/slurm/scratch/id_rsa /home/vagrant/.ssh/
    # the extra 'echo' is needed because Vagrant inserts its own key without a
    # newline at the end
    (echo; cat /vagrant/slurm/scratch/id_rsa.pub) >> /home/vagrant/.ssh/authorized_keys
    # add EPEL
    yum -y install epel-release
    # add our CAD repo
    [[ -e /vagrant/slurm/cad.repo ]] && cp /vagrant/slurm/cad.repo /etc/yum.repos.d/
    # install required pkgs
    yum -y install slurm ansible git vim htop
    # we only generate the munge key once
    if [[ ! -e /vagrant/slurm/scratch/munge.key ]]; then
      /usr/sbin/create-munge-key
      cp /etc/munge/munge.key /vagrant/slurm/scratch
    else
      cp /vagrant/slurm/scratch/munge.key /etc/munge
    fi
    chown munge.munge /etc/munge/munge.key
    sudo systemctl restart munge
    # create lock file
    touch /etc/CLUSTER_VAGRANT_UP
  fi
SCRIPT

$clone = <<-CLONE
    # clone our repo
    [[ ! -e /home/vagrant/cluster_ansible ]] && \
      git clone --single-branch --branch vagrant  \
      https://github.com/eResearchSandpit/cadclusterhigh-ansible.git /home/vagrant/cluster_ansible && \
      chown -R vagrant.vagrant /home/vagrant/cluster_ansible
CLONE

Vagrant.configure("2") do |cluster|
  #basic setup
  cluster.vm.box = "centos/7"
  cluster.vm.provider 'virtualbox' do |vb|
      vb.memory = 256
      vb.cpus = 1
  end
  # slurm dir is for sharing common data
  cluster.vm.provision "shell", inline: $script
  #define nodes
  cluster.vm.define "login" do |login|
      login.vm.hostname = "login"
      login.vm.network "private_network", ip: "192.168.100.12", virtualbox_intnet: "clusternet"
      # clone our repos on the login node
      login.vm.provision "shell", inline: $clone
  end
  cluster.vm.define "node01" do |node01|
      node01.vm.hostname = "node01"
      node01.vm.network "private_network", ip: "192.168.100.13", virtualbox_intnet: "clusternet"
  end
  cluster.vm.define "node02" do |node02|
      node02.vm.hostname = "node02"
      node02.vm.network "private_network", ip: "192.168.100.14", virtualbox_intnet: "clusternet"
  end
  cluster.vm.define "node03" do |node03|
      node03.vm.hostname = "node03"
      node03.vm.network "private_network", ip: "192.168.100.15", virtualbox_intnet: "clusternet"
  end
end
