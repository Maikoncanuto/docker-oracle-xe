# -*- mode: ruby -*-
# vi: set ft=ruby ts=2 sw=2 expandtab:

Vagrant.configure(2) do |config|
  config.vm.box = "boxcutter/centos71"
  config.vm.hostname = "oracle-xe"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end

  config.vm.network :forwarded_port, guest: 1521, host: 1521

  config.vm.provision "docker" do |d|
    # docker build --rm -t oracle-xe .
    d.build_image "/vagrant", args: "-t oracle-xe"

    # docker run --name oracle-xe -d
    d.run "oracle-xe", image: "oracle-xe:latest", daemonize: true, args: "-p 1521:1521"
  end
end
