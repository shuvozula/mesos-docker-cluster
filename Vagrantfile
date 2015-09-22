# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

# Load config from config.yaml
current_dir    = File.dirname(File.expand_path(__FILE__))
yaml           = YAML.load_file("#{current_dir}/config.yaml")
configs        = yaml['mesos-cluster-config']
servers        = configs['servers']
zk_config      = configs['zookeeper']


Vagrant.configure(2) do |config|

  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  # Mesos-Node1 = Zookeeper1 + Mesos-Master-1 + Marathon.
  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  config.vm.define "mesos-node1" do |node1|

    # allocate resources
    node1.vm.provider "virtualbox" do |v|
      v.name = "mesos-node1"
      v.memory = 2048
      v.cpus = 2
    end

    # Boxes are available at https://atlas.hashicorp.com/search.
    node1.vm.box = "ubuntu/trusty64"
    node1.vm.hostname = "mesos-node1"
    node1.vm.network :private_network, ip: "#{servers['ip_1']}"

    # Create docker containers for Zookeeper1 + Mesos-Master-1 + Marathon
    node1.vm.provision "docker" do |d|

      # Zookeeper - 1
      d.run "mesos-zookeeper-1",
        image: "mesoscloud/zookeeper:3.4.6-ubuntu-14.04",
        daemonize: true,
        args: "-e SERVER_ID=1 " \
              "-e MYID=1 " \
              "-e HOST_IP_1=" + servers["ip_1"] + " " \
              "-e HOST_IP_2=" + servers["ip_2"] + " " \
              "-e ADDITIONAL_ZOOKEEPER_1=server.1=" + servers["ip_1"] + ":2888:3888 " \
              "-e ADDITIONAL_ZOOKEEPER_2=server.2=" + servers["ip_2"] + ":2888:3888 " \
              "--net=host"

      # Mesos Master - 1
      d.run "mesos-master-1",
        image: "mesoscloud/mesos-master:0.23.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MESOS_HOSTNAME=" + servers["ip_1"] + " " \
              "-e MESOS_IP=" + servers["ip_1"] + " " \
              "-e HOST_IP_1=" + servers["ip_1"] + " " \
              "-e HOST_IP_2=" + servers["ip_2"] + " " \
              "-e MESOS_ZK=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MESOS_PORT=5050 " \
              "-e MESOS_LOG_DIR=/var/log/mesos " \
              "-e MESOS_QUORUM=1 " \
              "-e MESOS_REGISTRY=in_memory " \
              "-e MESOS_WORK_DIR=/var/lib/mesos " \
              "--net=host"

      # Marathon - 1
      d.run "marathon",
        image: "mesoscloud/marathon:0.10.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MARATHON_HOSTNAME=" + servers["ip_1"] + " " \
              "-e MARATHON_HTTPS_ADDRESS=" + servers["ip_1"] + " " \
              "-e MARATHON_HTTP_ADDRESS=" + servers["ip_1"] + " " \
              "-e MARATHON_MASTER=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MARATHON_ZK=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/marathon " \
              "--net=host"

    end

    node1.vm.provision "shell", inline: <<-SHELL
      echo 'cd /vagrant' >> /home/vagrant/.bashrc
    SHELL

  end

  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  # Mesos-Node2 = Zookeeper2 + Mesos-Master-2 + Marathon.
  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  config.vm.define "mesos-node2" do |node2|

    # allocate resources
    node2.vm.provider "virtualbox" do |v|
      v.name = "mesos-node2"
      v.memory = 2048
      v.cpus = 2
    end

    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://atlas.hashicorp.com/search.
    node2.vm.box = "ubuntu/trusty64"
    node2.vm.hostname = "mesos-node2"
    node2.vm.network :private_network, ip: "#{servers['ip_2']}"

    # Create docker containers for Zookeeper1 + Mesos-Master-1 + Marathon
    node2.vm.provision "docker" do |d|

      d.run "mesos-zookeeper-2",
        image: "mesoscloud/zookeeper:3.4.6-ubuntu-14.04",
        daemonize: true,
        args: "-e SERVER_ID=2 " \
              "-e MYID=2 " \
              "-e HOST_IP_1=" + servers["ip_1"] + " " \
              "-e HOST_IP_2=" + servers["ip_2"] + " " \
              "-e ADDITIONAL_ZOOKEEPER_1=server.1=" + servers["ip_1"] + ":2888:3888 " \
              "-e ADDITIONAL_ZOOKEEPER_2=server.2=" + servers["ip_2"] + ":2888:3888 " \
              "--net=host"

      # Start Mesos Master 2
      d.run "mesos-master-2",
        image: "mesoscloud/mesos-master:0.23.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MESOS_HOSTNAME=" + servers['ip_2'] + " " \
              "-e MESOS_IP=" + servers['ip_2'] + " " \
              "-e HOST_IP_1=" + servers["ip_1"] + " " \
              "-e HOST_IP_2=" + servers["ip_2"] + " " \
              "-e MESOS_ZK=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MESOS_PORT=5050 " \
              "-e MESOS_LOG_DIR=/var/log/mesos " \
              "-e MESOS_QUORUM=1 " \
              "-e MESOS_REGISTRY=in_memory " \
              "-e MESOS_WORK_DIR=/var/lib/mesos " \
              "--net=host"

      d.run "chronos",
        image: "mesoscloud/chronos:2.3.4-ubuntu-14.04",
        daemonize: true,
        args: "-e CHRONOS_HTTP_PORT=4400 " \
              "-e CHRONOS_MASTER=zk://" + servers['ip_1'] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e CHRONOS_ZK_HOSTS=" + servers['ip_1'] + ":2181," + servers["ip_2"] + ":2181 " \
              "--net=host"

    end

    node2.vm.provision "shell", inline: <<-SHELL
      echo 'cd /vagrant' >> /home/vagrant/.bashrc
    SHELL

  end

  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  # Mesos-Node3 = Mesos Slaves
  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  config.vm.define "mesos-node3" do |node3|

    # allocate resources
    node3.vm.provider "virtualbox" do |v|
      v.name = "mesos-node3"
      v.memory = 2048
      v.cpus = 2
    end

    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://atlas.hashicorp.com/search.
    node3.vm.box = "ubuntu/trusty64"
    node3.vm.hostname = "mesos-node3"
    node3.vm.network :private_network, ip: "#{servers['ip_3']}"

    # Create docker containers for Zookeeper1 + Mesos-Master-1 + Marathon
    node3.vm.provision "docker" do |d|

      d.run "mesos-slave-1",
        image: "mesoscloud/mesos-slave:0.23.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MESOS_HOSTNAME=" + servers['ip_3'] + " " \
              "-e MESOS_IP=" + servers['ip_3'] + " " \
              "-e MESOS_LOG_DIR=/var/log/mesos " \
              "-e MESOS_LOGGING_LEVEL=INFO " \
              "-e MESOS_MASTER=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MESOS_PORT=5051 " \
              "-v /sys/fs/cgroup:/sys/fs/cgroup " \
              "-v /var/run/docker.sock:/var/run/docker.sock " \
              "--net=host --privileged" #--port=5051"

      d.run "mesos-slave-2",
        image: "mesoscloud/mesos-slave:0.23.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MESOS_HOSTNAME=" + servers['ip_3'] + " " \
              "-e MESOS_IP=" + servers['ip_3'] + " " \
              "-e MESOS_LOG_DIR=/var/log/mesos " \
              "-e MESOS_LOGGING_LEVEL=INFO " \
              "-e MESOS_MASTER=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MESOS_PORT=5052 " \
              "-v /sys/fs/cgroup:/sys/fs/cgroup " \
              "-v /var/run/docker.sock:/var/run/docker.sock " \
              "--net=host --privileged" # --port=5052"

      d.run "mesos-slave-3",
        image: "mesoscloud/mesos-slave:0.23.0-ubuntu-14.04",
        daemonize: true,
        args: "-e MESOS_HOSTNAME=" + servers['ip_3'] + " " \
              "-e MESOS_IP=" + servers['ip_3'] + " " \
              "-e MESOS_LOG_DIR=/var/log/mesos " \
              "-e MESOS_LOGGING_LEVEL=INFO " \
              "-e MESOS_MASTER=zk://" + servers["ip_1"] + ":2181," + servers["ip_2"] + ":2181/mesos " \
              "-e MESOS_PORT=5053 " \
              "-v /sys/fs/cgroup:/sys/fs/cgroup " \
              "-v /var/run/docker.sock:/var/run/docker.sock " \
              "--net=host --privileged" # --port=5053"

    end

    node3.vm.provision "shell", inline: <<-SHELL
      echo 'cd /vagrant' >> /home/vagrant/.bashrc
    SHELL

  end

end
