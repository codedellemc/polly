# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'fileutils'
require 'shellwords'

Vagrant.require_version ">= 1.8.0"

# ensure that the authentication module for the VirtualBox web server has
# been disabled and that the web server is online
if ARGV[0] == "up"
  unless `ps alx | grep [v]boxwebsrv` != ""
    printf "starting virtualbox web server\n"
    print `VBoxManage setproperty websrvauthlibrary null && vboxwebsrv --background`
  end
end

$default_install = "stable"
if ENV['DEPLOY_TYPE']
  $default_install = ENV['DEPLOY_TYPE']
  printf("Changing the deploy type to %s\n", $default_install)
end

# node info
$node0_name = "node0"
$node0_ip   = "192.168.56.10"
$node0_mem  = "1024"

$node1_name = "node1"
$node1_ip   = "192.168.56.11"
$node1_mem  = "512"

# Golang information
$goos   = "linux"
$goarch = "amd64"
$gover  = "1.6.2"
$gotgz  = "go#{$gover}.#{$goos}-#{$goarch}.tar.gz"
$gourl  = "https://storage.googleapis.com/golang/#{$gotgz}"
$gopath = "/opt/go"

# the script to provision golang
$provision_golang = <<SCRIPT
echo installing go#{$gover}.#{$goos}-#{$goarch}
wget -q #{$gourl.shellescape}
tar -C /usr/local -xzf #{$gotgz.shellescape}
mkdir -p #{$gopath.shellescape}
rm -f #{$gotgz.shellescape}
SCRIPT

# the script to provision docker
$provision_docker = <<SCRIPT
apt-get update
apt-get install apt-transport-https ca-certificates
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 \
            --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > \
     /etc/apt/sources.list.d/docker.list
apt-get update
apt-get purge lxc-docker
apt-get install linux-image-extra-$(uname -r)
apt-get install apparmor
apt-get update
apt-get install -y docker-engine
usermod -a -G docker vagrant
service docker start
docker run hello-world
SCRIPT

# go-bindata info
$go_bindata_dir = "#{$gopath}/src/github.com/jteeuwen/go-bindata"
$go_bindata_url = "https://github.com/akutz/go-bindata"
$go_bindata_ref = "feature/md5checksum"

# the script to build go-bindata
$build_go_bindata = <<SCRIPT
mkdir -p #{$go_bindata_dir.shellescape}
cd #{$go_bindata_dir.shellescape}
git clone #{$go_bindata_url.shellescape} .
git checkout #{$go_bindata_ref.shellescape}
go get ./...
go install ./...
SCRIPT

# rex-ray repo and branch information
$rexray_dir = "#{$gopath}/src/github.com/emccode/rexray"
$rexray_url = "https://github.com/emccode/rexray"
$rexray_ref = "master"
$rexray_bin = "/usr/bin/rexray"
$rexray_cfg = "/etc/rexray/config.yml"

if $default_install == "custom" && ENV['REXRAY_REF']
  $rexray_ref = ENV['REXRAY_REF']
  printf("Changing the REX-Ray checkout ref to %s\n", $rexray_ref)
end

# the script to build rex-ray
$build_rexray = <<SCRIPT
mkdir -p #{$rexray_dir.shellescape}
cd #{$rexray_dir.shellescape}
git clone #{$rexray_url.shellescape} .
git checkout #{$rexray_ref.shellescape}
make deps
make
SCRIPT

# polly repo and branch information
$polly_dir = "#{$gopath}/src/github.com/emccode/polly"
$polly_url = "https://github.com/emccode/polly"
$polly_ref = "master"
$polly_bin = "/usr/bin/polly"
$polly_cfg = "/etc/polly/config.yml"

if $default_install == "custom" && ENV['POLLY_REF']
  $polly_ref = ENV['POLLY_REF']
  printf("Changing the Polly checkout ref to %s\n", $polly_ref)
end

# the script to build polly
$build_polly = <<SCRIPT
mkdir -p #{$polly_dir.shellescape}
cd #{$polly_dir.shellescape}
git clone #{$polly_url.shellescape} .
git checkout #{$polly_ref.shellescape}
make deps
make
SCRIPT

# volume_path is a valid directory path on the local, host system for storing
# virtualbox volumes. ensure it exists as well
$volume_path = "#{File.dirname(__FILE__)}/.vagrant/volumes"
FileUtils::mkdir_p $volume_path

# the script to write node0's rex-ray config file. the 'virtualbox.volumePath'
# property should be replaced with a valid directory path on the virtualbox
# host system
$write_rexray_config_node0 = <<SCRIPT
mkdir -p #{File.dirname($rexray_cfg).shellescape}
cat << EOF > #{$rexray_cfg.shellescape}
rexray:
  logLevel: warn
libstorage:
  host:     tcp://127.0.0.1:7981
  service:  virtualbox
EOF
SCRIPT

# the script to write node1's rex-ray config file. the 'virtualbox.volumePath'
# property should be replaced with a valid directory path on the virtualbox
# host system
$write_rexray_config_node1 = <<SCRIPT
mkdir -p #{File.dirname($rexray_cfg).shellescape}
cat << EOF > #{$rexray_cfg.shellescape}
rexray:
  logLevel: warn
libstorage:
  host:    tcp://#{$node0_ip}:7981
  service: virtualbox
EOF
SCRIPT

# the script to write node0's polly config file. the 'virtualbox.volumePath'
# property should be replaced with a valid directory path on the virtualbox
# host system
$write_polly_config_node0 = <<SCRIPT
mkdir -p #{File.dirname($polly_cfg).shellescape}
cat << EOF > #{$polly_cfg.shellescape}
polly:
  host: tcp://127.0.0.1:7978
  store:
    type: boltdb
    endpoints: /tmp/boltdb
    bucket: MyBoltDb_test
libstorage:
  host:     tcp://127.0.0.1:7981
  embedded: false
  service:  virtualbox
  server:
    endpoints:
      public:
        address: tcp://:7981
    services:
      virtualbox:
        driver: virtualbox
virtualbox:
  volumePath: #{$volume_path.shellescape}
EOF
SCRIPT

# POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS
# the script to write node1's polly config file. the 'virtualbox.volumePath'
# property should be replaced with a valid directory path on the virtualbox
# host system
#$write_polly_config_node1 = <<SCRIPT
#mkdir -p #{File.dirname($polly_cfg).shellescape}
#cat << EOF > #{$polly_cfg.shellescape}
#polly:
#  host: tcp://[TODO]:7978
#libstorage:
#  host:    tcp://[TODO]:7981
#  service: virtualbox
#EOF
#SCRIPT
# POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS

#write the MOTD to help users get started
$motd_file = "/etc/motd"

$write_motd_config = <<SCRIPT
mkdir -p #{File.dirname($motd_file).shellescape}
cat << EOF > #{$motd_file.shellescape}

----------------------------------------------
This host is running:
polly @ /usr/bin/polly
rexray @ /usr/bin/rexray

Sample commands for REX-Ray (can run on node0 or node1):
----------------------------------------------
Create a volume:
rexray volume create --volumename=node0vol --size=1

List volumes:
rexray volume


Sample commands for Polly (run on node0 only + must sudo su):
----------------------------------------------
Create a volume:
polly volume create --servicename=virtualbox --name=pollyvol --size=1

List volumes:
polly volume ls

Delete volume:
polly volume remove --volumeid=<Volume UUID>
----------------------------------------------

EOF
SCRIPT

# init the environment variables used when building go source
$build_env_vars = Hash[
    "GOPATH" => $gopath.shellescape,
    "PATH" => "#{$gopath.shellescape}/bin:/usr/local/go/bin:$PATH"
]

# node_dir returns the directory for a given node
def node_dir(name)
    return "#{File.dirname(__FILE__)}/.vagrant/machines/#{name}"
end

# is_first_up returns a flag indicating whether or not this is the first time
# 'vagrant up' has been called on a specific node
def is_first_up(name)
  return Dir.glob("#{node_dir(name)}/*/id").empty?
end

# init_node initializes the node information
def init_node(node, name, ip)
  node.vm.box = "ubuntu/trusty64"
  node.vm.hostname = name
  node.vm.network :private_network, ip: ip
end

# init_virtualbox initializes the virtualbox settings for a VM
def init_virtualbox(vb, ram)
  # set the VM's RAM size. must be at least 1GB.
  # see https://github.com/beego/wetalk/issues/32
  # https://groups.google.com/forum/#!topic/golang-nuts/0qUdADqqsDs
  vb.memory = ram

  # renamed the SATA controller to be the default name for a VirtualBox
  # SATA controller, `SATA`
  vb.customize ["storagectl", :id, "--name", "SATAController", "--rename", "SATA"]

  # set the SATA controller's port count so it's greater than 1. if this
  # step is omitted it is not possible to attach new volumes to this VM.
  vb.customize ["storagectl", :id, "--name", "SATA", "--portcount", "25"]

  # enable NAT DNS
  vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

  # make sure the first NIC has a "random" MAC address to ensure it is not
  # the same MAC address as the other node
  vb.customize ["modifyvm", :id, "--macaddress1", "auto"]

end

Vagrant.configure("2") do |config|

  # configure node0
  config.vm.define $node0_name do |node|

    # initialize the node information
    init_node node, $node0_name, $node0_ip

    # only proceed if this is the first time 'vagrant up' has been called
    # on this node
    if is_first_up node.vm.hostname

      # initialize virtualbox
      node.vm.provider :virtualbox do |vb|
        init_virtualbox vb, $node0_mem
      end

      # provision docker - don't use docker provisioner - see
      # https://github.com/mitchellh/vagrant/issues/7161
      node.vm.provision "shell" do |s|
        s.name =   "docker"
        s.inline = $provision_docker
      end

      #on the non-stable version of polly and rexray, we need to install
      #and build each component
      if $default_install != "stable" && $default_install != "staged" && $default_install != "unstable"

        # provision golang
        node.vm.provision "shell" do |s|
          s.name   = "golang"
          s.inline = $provision_golang
        end

         # build go-bindata
         node.vm.provision "shell" do |s|
           s.name   = "go-bindata"
           s.env    = $build_env_vars
           s.inline = $build_go_bindata
         end

         # build polly
         node.vm.provision "shell" do |s|
           s.name   = "build polly"
           s.env    = $build_env_vars
           s.inline = $build_polly
         end

         # copy polly to /usr/bin
         node.vm.provision "shell" do |s|
           s.name   = "copy polly"
           s.inline = "cp #{$gopath.shellescape}/bin/polly " +
                      "#{$polly_bin.shellescape}"
         end

         # build rex-ray
         node.vm.provision "shell" do |s|
           s.name   = "build rex-ray"
           s.env    = $build_env_vars
           s.inline = $build_rexray
         end

         # copy rex-ray to /usr/bin
         node.vm.provision "shell" do |s|
           s.name   = "copy rex-ray"
           s.inline = "cp #{$gopath.shellescape}/bin/rexray " +
                      "#{$rexray_bin.shellescape}"
         end

      else

        # install polly
        node.vm.provision "shell" do |s|
          s.name       = "install polly"
          s.inline     = "curl -sSL https://dl.bintray.com/emccode/polly/install | sh -s #{$default_install}"
        end

        # install rexray
        node.vm.provision "shell" do |s|
          s.name       = "install rexray"
          s.inline     = "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh -s #{$default_install}"
        end

      end

       # write polly config file
       node.vm.provision "shell" do |s|
         s.name       = "config polly"
         s.inline     = $write_polly_config_node0
       end

       # write rex-ray config file
       node.vm.provision "shell" do |s|
         s.name       = "config rex-ray"
         s.inline     = $write_rexray_config_node0
       end

       # install polly
       node.vm.provision "shell" do |s|
         s.name   = "polly install"
         s.inline = "polly install"
       end

       # install rex-ray
       node.vm.provision "shell" do |s|
         s.name   = "rex-ray install"
         s.inline = "rexray install"
       end

       # start polly as a service
       node.vm.provision "shell" do |s|
         s.name   = "start polly"
         s.inline = "/etc/init.d/polly start"
       end

      # start rex-ray as a service
      node.vm.provision "shell" do |s|
        s.name   = "start rex-ray"
        s.inline = "/etc/init.d/rexray start"
      end

      # write MOTD
      node.vm.provision "shell" do |s|
        s.name   = "write MOTD"
        s.inline = $write_motd_config
      end

    end # if is_first_up node.vm.hostname

    # list volume mapping with rex-ray to verify configuration
    node.vm.provision "shell", run: "always" do |s|
      s.name       = "rex-ray volume map"
      s.privileged = false
      s.inline     = "rexray volume map"
    end

  end # configure node0

  # configure node1
  config.vm.define $node1_name do |node|

    # initialize the node information
    init_node node, $node1_name, $node1_ip

    # only proceed if this is the first time 'vagrant up' has been called
    # on this node
    if is_first_up node.vm.hostname

      # initialize virtualbox
      node.vm.provider :virtualbox do |vb|
        init_virtualbox vb, $node1_mem
      end

      # provision docker - don't use docker provisioner - see
      # https://github.com/mitchellh/vagrant/issues/7161
      node.vm.provision "shell" do |s|
        s.name =   "docker"
        s.inline = $provision_docker
      end

      # copy node0's private ssh key to node1 so node1 can ssh to node0 without
      # being prompted for a password
      node.vm.provision              "file",
                        source:      "#{node_dir($node0_name)}" +
                                     "/virtualbox/private_key".shellescape,
                        destination: '"$HOME"/.ssh' +
                                     "/#{$node0_name.shellescape}.key"

      #on the non-stable version of polly and rexray, we need to install
      #and build each component
      if $default_install != "stable" && $default_install != "staged" && $default_install != "unstable"

        # scp rex-ray from node0 to node1
        node.vm.provision "shell" do |s|
          s.name =   "scp rexray"
          s.inline = "scp -q -i " +
                     "/home/vagrant/.ssh/#{$node0_name}.key".shellescape + " " +
                     "-o StrictHostKeyChecking=no " +
                     "vagrant@#{$node0_ip}:#{$rexray_bin.shellescape} " +
                     "#{$rexray_bin.shellescape}"
        end

        # POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS
        # scp polly from node0 to node1
        #node.vm.provision "shell" do |s|
        #  s.name =   "scp polly"
        #  s.inline = "scp -q -i " +
        #              "/home/vagrant/.ssh/#{$node0_name}.key".shellescape + " " +
        #              "-o StrictHostKeyChecking=no " +
        #              "vagrant@#{$node0_ip}:#{$polly_bin.shellescape} " +
        #              "#{$polly_bin.shellescape}"
        #end

      else

        # install polly
        node.vm.provision "shell" do |s|
          s.name       = "install polly"
          s.inline     = "curl -sSL https://dl.bintray.com/emccode/polly/install | sh -s #{$default_install}"
        end

        # install rexray
        node.vm.provision "shell" do |s|
          s.name       = "install rexray"
          s.inline     = "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh -s #{$default_install}"
        end

      end

      # write rex-ray config file
      node.vm.provision "shell" do |s|
        s.name       = "config rex-ray"
        s.inline     = $write_rexray_config_node1
      end

      # POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS
      # write polly config file
      #node.vm.provision "shell" do |s|
      #  s.name       = "config polly"
      #  s.inline     = $write_polly_config_node1
      #end

      # install rex-ray
      node.vm.provision "shell" do |s|
        s.name   = "rex-ray install"
        s.inline = "rexray install"
      end

      # POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS
      # install polly
      #node.vm.provision "shell" do |s|
      #  s.name   = "polly install"
      #  s.inline = "polly install"
      #end

      # start rex-ray as a service
      node.vm.provision "shell" do |s|
        s.name   = "start rex-ray"
        s.inline = "/etc/init.d/rexray start"
      end

      # POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS
      # start polly as a service
      #node.vm.provision "shell" do |s|
      #  s.name   = "start polly"
      #  s.inline = "/etc/init.d/polly start"
      #end
      # POLLY IN A REMOTE MODE IS CURRENTLY NOT SUPPORTED, UNCOMMENT WHEN IT IS

      # write MOTD
      node.vm.provision "shell" do |s|
        s.name   = "write MOTD"
        s.inline = $write_motd_config
      end

    end # if is_first_up node.vm.hostname

    # list volume mapping with rex-ray to verify configuration
    node.vm.provision "shell", run: "always" do |s|
      s.name       = "rex-ray volume map"
      s.privileged = false
      s.inline     = "rexray volume map"
    end

  end # configure node1

end
