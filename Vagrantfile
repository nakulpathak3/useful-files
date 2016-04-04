# -*- mode: ruby -*-
# vi: set ft=ruby :

boxes = [
    {
        :name => "es01",
        :eth1 => "192.168.205.11",
        :mem => "768",
        :cpu => "2",
        :mac => "08002767EDD3"
    },
    {
        :name => "es02",
        :eth1 => "192.168.205.12",
        :mem => "768",
        :cpu => "2",
        :mac => "08002767EDD4"
    },
    {
        :name => "sensu",
        :eth1 => "192.168.205.13",
        :mem => "1024",
        :cpu => "2",
        :mac => "08002767EDC7"
    },

]

Vagrant.configure(2) do |config|

  config.vm.provider :virtualbox do |v, override|
    config.vm.box = "rchouinard/oracle-65-x64"
  end

  
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]

      if opts[:name] == "sensu"
        config.vm.network "forwarded_port", guest: 3000, host: 5000
      end

      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end
      
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      end

      config.timezone.value = "America/Toronto"
      # Configure networking & vagrant-hostmanager
      config.proxy.http     = "http://www-proxy.us.oracle.com:80"
      config.proxy.https    = "http://www-proxy.us.oracle.com:80"
      config.proxy.no_proxy =  boxes.map { |host| host[:name] }.join(',') + ',localhost'
      #config.proxy.no_proxy = "localhost,127.0.0.1,zoo01,zoo02,zoo03"
      config.vm.network :public_network, mac: opts[:mac], bridge: 'Intel(R) Ethernet Connection I217-LM'
      config.hostmanager.enabled = true

      config.hostmanager.manage_host = true
      config.hostmanager.ignore_private_ip = false
      config.hostmanager.include_offline = false

      config.hostmanager.ip_resolver = proc do |machine|
        result = ""
        machine.communicate.execute("ifconfig eth1") do |type, data|
          result << data if type == :stdout
        end
        (ip = /inet addr:(\d+\.\d+\.\d+\.\d+)/.match(result)) && ip[1]
      end

    end
  end
end
