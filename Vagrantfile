
$script = <<-SCRIPT
  set -x
  # Only need to do this once, so check for lock file
  if [[ ! -e /etc/CLUSTER_VAGRANT_UP ]]; then
    echo "192.168.100.12    login" >> /etc/hosts
    echo "192.168.100.13    node01" >> /etc/hosts
    echo "192.168.100.14    node02" >> /etc/hosts
    # we only generate the key on one of the nodes
    if [[ ! -e /scratch/id_rsa ]]; then
      ssh-keygen -t rsa -f /scratch/id_rsa -N ""
    fi
    install -m 600 -o vagrant -g vagrant /scratch/id_rsa /home/vagrant/.ssh/
    # the extra 'echo' is needed because Vagrant inserts its own key without a
    # newline at the end
    (echo; cat /scratch/id_rsa.pub) >> /home/vagrant/.ssh/authorized_keys
    # add EPEL
    yum -y install epel-release
    # add our CAD repo
    [[ -e /slurm/cad.repo ]] && cp /slurm/cad.repo /etc/yum.repos.d/
    # install Slurm, Ansible & git
    yum -y install slurm ansible git
    # we only generate the munge key once
    if [[ ! -e /scratch/munge.key ]]; then
      /usr/sbin/create-munge-key
      cp /etc/munge/munge.key /scratch
    else
      cp /scratch/munge.key /etc/munge
    fi
    chown munge.munge /etc/munge/munge.key
    sudo systemctl restart munge
    # clone our repositories
    [[ ! -e /scratch/cluster_create ]] && \
      git clone https://github.com/eResearchSandpit/cadclusterhigh-ansible.git /scratch/cluster_create
    [[ ! -e /scratch/cluster_config ]] && \
      git clone https://github.com/eResearchSandpit/cadClusterConfig.git /scratch/cluster_config
    # create lock file
    touch /etc/CLUSTER_VAGRANT_UP
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
  cluster.vm.synced_folder "scratch/", "/scratch", type: "nfs", mount_options: ['rw', 'vers=3', 'tcp', 'fsc', 'actimeo=1'] # for macos
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
