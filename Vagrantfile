# Require YAML module
require 'yaml'

# Read YAML file with box details
configuration = YAML.load_file('config.yaml')


Vagrant.require_version '>= 1.6.0'

FORWARDED_PORT_RANGE_CUSTOM = [8080,5000, 2375]

Vagrant.configure(2) do |config|
    config.vm.box = configuration["box"]
    config.vm.provision "docker"
    config.vm.provision "docker_compose"
    config.vm.hostname = configuration["hostname"]

    config.vm.network "private_network", ip: configuration["ip"]

    config.vm.provider configuration["provider"] do |vb|
        vb.name = configuration["name"]
        vb.memory = configuration["ram"]
        vb.gui = configuration["gui"]
        vb.cpus = configuration["cpus"]

        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    for i in FORWARDED_PORT_RANGE_CUSTOM
                config.vm.network "forwarded_port", guest: i, host: i
        end

    if Vagrant.has_plugin?("vagrant-cachier")
        config.cache.scope = :box
    end
    #Setting Share Dir
    config.vm.synced_folder 'DockerImages', "/home/vagrant/DockerImages", :mount_options => ["dmode=777","fmode=776"]
    config.vm.synced_folder 'workspace', "/workspace", :mount_options => ["dmode=777","fmode=776"]
end
