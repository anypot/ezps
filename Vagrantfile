Vagrant.configure("2") do |config|
  (1..3).each do |i|
    config.vm.define "ezpsnode#{i}" do |node|
      node.vm.box = "centos/atomic-host"
      node.vm.hostname = "ezpsnode#{i}"

      # node.vm.network :private_network, ip: "192.168.56.10#{i}"
      node.vm.network :public_network, ip: "192.168.1.9#{i}"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", "ezpsnode#{i}"]
      end
    end
  end
end

