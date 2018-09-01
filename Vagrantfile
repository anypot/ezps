# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
VAGRANT_API_VERSION = '2'

config_file = File.expand_path(File.join(File.dirname(__FILE__), 'vagrant_vars.yml'))
settings = YAML.load_file(config_file)

HOSTNAME_PREFIX		= settings['hostname_prefix'] ? settings['hostname_prefix'] + '-' : ""
BOX			= settings['vagrant_box']
BOX_URL			= settings['vagrant_box_url']
PUBLIC_SUBNET		= settings['public_subnet']
MEMORY			= settings['memory']
USER			= settings['ssh_username']
SYNC_DIR		= settings['vagrant_sync_dir']
DISABLE_SYNCED_FOLDER	= settings.fetch('vagrant_disable_synced_folder', false)
SKIP_TAGS		= settings['skip_tags']
DEBUG			= settings['debug']

ASSIGN_STATIC_IP = !(BOX == 'openstack' or BOX == 'anothercloudprovider')

ansible_provision = proc do |ansible|
  ansible.playbook = 'site.yml'
  ansible.limit = 'all'

  ansible.extra_vars = {
    public_network: "#{PUBLIC_SUBNET}.0/24",
    private_iface: settings['private_iface'],
    disks: settings['disks']
  }

  ansible.groups = {
	  'ezps' => (1..3).map{ |j| "#{HOSTNAME_PREFIX}node#{j}" }
  }

  if SKIP_TAGS then
    ansible.skip_tags = SKIP_TAGS
  end

  if DEBUG then
    ansible.verbose = '-vvvvv'
  end
end

Vagrant.configure(VAGRANT_API_VERSION) do |config|
  config.vm.box = BOX
  config.vm.box_url = BOX_URL
  config.ssh.username = USER
  config.ssh.private_key_path = settings['ssh_private_key_path']

  if DISABLE_SYNCED_FOLDER
    config.vm.provider :virtualbox do |v,override|
      override.vm.synced_folder '.', SYNC_DIR, disabled: true
    end
  end

  (1..3).each do |i|
    config.vm.define "#{HOSTNAME_PREFIX}node#{i}" do |node|
      node.vm.hostname = "#{HOSTNAME_PREFIX}node#{i}"

      if ASSIGN_STATIC_IP
        node.vm.network :private_network, ip: "#{PUBLIC_SUBNET}.9#{i}"
      end

      node.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--memory", "#{MEMORY}"]
        vb.customize ["modifyvm", :id, "--name", "#{HOSTNAME_PREFIX}node#{i}"]
      end
      node.vm.provision "ansible", &ansible_provision if i == 3
    end
  end
end

