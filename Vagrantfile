MACHINES = {
	:router1 => {
		:box_name => "centos7",
		:vm_name => "router1",
		:net => [
			{ip: '10.0.10.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
			{ip: '10.0.12.1', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
			{ip: '192.168.10.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net1"},
			{ip: '192.168.50.10', adapter: 5},
				]
				},
	:router2 => {
		:box_name => "centos7",
		:vm_name => "router2",
		:net => [
			{ip: '10.0.10.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
			{ip: '10.0.11.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
			{ip: '192.168.20.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
			{ip: '192.168.50.11', adapter: 5},
				]
				},
	:router3 => {
		:box_name => "centos7",
		:vm_name => "router3",
		:net => [
			{ip: '10.0.11.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
			{ip: '10.0.12.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
			{ip: '192.168.30.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net3"},
			{ip: '192.168.50.12', adapter: 5},
				]
				}
			}
Vagrant.configure("2") do |config|
	MACHINES.each do |boxname, boxconfig|
		config.vm.define boxname do |box|
			box.vm.box = boxconfig[:box_name]
			box.vm.host_name = boxconfig[:vm_name]
			  box.vm.provision "ansible" do |ansible|
				ansible.playbook = "ospf_asim_routering/provision.yml" #Изменить в соответсвии с пунктом ДЗ
			  end
			boxconfig[:net].each do |ipconf|
				box.vm.network "private_network", ipconf
			end
		end
	end
end
