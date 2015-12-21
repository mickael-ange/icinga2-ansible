    boxes = [
        { :name => "icinga2-web2",       :eth1 => "172.16.1.2" }
    ]

Vagrant.configure("2") do |config|
    config.vm.box = "jhcook/centos7"
    config.vm.box_url = "https://atlas.hashicorp.com/jhcook/boxes/centos7"
    config.vm.synced_folder ".", "/vagrant", disabled: true

    boxes.each do |opts|
        config.vm.define opts[:name] do |node|
            node.vm.hostname = opts[:name]
            node.vm.network "private_network", ip: opts[:eth1]
            node.vm.provision "ansible" do |ansible|
                ansible.limit = opts[:name]
                ansible.inventory_path = "hosts"

                # Set playbook
                if opts[:playbook]
                    ansible.playbook = "playbooks/" + opts[:playbook]
                else
                    # Default playbook name
                    ansible.playbook = "playbooks/" + opts[:name] + "-setup.yml"
                end

                # Set verbose level
                if opts[:verbose]
                    ansible.verbose = opts[:verbose]
                end

                # Set extra_vars
                if opts[:extra_vars]
                    ansible.extra_vars = opts[:extra_vars]
                end
            end
        end
    end
end
