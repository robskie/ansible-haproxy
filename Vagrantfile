boxes = {
  "wheezy64" => {
    :url => "https://atlas.hashicorp.com/debian/boxes/wheezy64",
    :memory => "128",
  },
  "jessie64" => {
    :url => "https://atlas.hashicorp.com/debian/boxes/jessie64",
    :memory => "128",
  },
  "precise64" => {
    :url => "https://atlas.hashicorp.com/ubuntu/boxes/precise64",
    :memory => "512",
  },
  "trusty64" => {
    :url => "https://atlas.hashicorp.com/ubuntu/boxes/trusty64",
    :memory => "512",
  },
  "xenial64" => {
    :url => "https://atlas.hashicorp.com/ubuntu/boxes/xenial64",
    :memory => "512",
  }
}

$script = <<SCRIPT
apt-get update -qq
apt-get install python -qq
SCRIPT

VAGRANTFILE_CONF_VERSION = "2"
ANSIBLE_TAGS = ENV['ANSIBLE_TAGS']
ANSIBLE_ROLE_NAME = "ansible-haproxy"

Vagrant.configure(VAGRANTFILE_CONF_VERSION) do |config|
  boxes.each do |box_name, box|
    config.vm.define box_name do |machine|
      machine.vm.box = box_name
      machine.vm.box_url = box[:url]
      machine.vm.synced_folder ".", "/vagrant", disabled: true

      machine.vm.provider "virtualbox" do |vb|
        vb.name = "%s-%s" % [ANSIBLE_ROLE_NAME, box_name]
        vb.memory = box[:memory]
      end

      machine.vm.provision "shell", inline: $script
      machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "tests/test.yml"
        ansible.tags = ANSIBLE_TAGS
      end
    end
  end
end
