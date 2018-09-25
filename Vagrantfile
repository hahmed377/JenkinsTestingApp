# Install required plugins
# include plugin for "vagrant-berkshelf"


required_plugins = ["vagrant-hostsupdater",'vagrant-berkshelf']
required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    Vagrant::Plugin::Manager.instance.install_plugin plugin
    # User vagrant plugin manager to install plugin, which will automatically refresh plugin list afterwards
  end
end


def set_env vars
  command = <<~HEREDOC
      echo "Setting Environment Variables"
      source ~/.bashrc
  HEREDOC

  vars.each do |key, value|
    command += <<~HEREDOC
      if [ -z "$#{key}" ]; then
          echo "export #{key}=#{value}" >> ~/.bashrc
      fi
    HEREDOC
  end

  return command
end

Vagrant.configure("2") do |config|
  config.vm.define "app" do |app|
    app.vm.box = "ubuntu/xenial64"
    app.vm.network "private_network", ip: "192.168.10.100"
    app.hostsupdater.aliases = ["development.local"]
    app.vm.synced_folder "app", "/home/ubuntu/app"
    app.vm.synced_folder "environment/app", "/home/ubuntu/environment"
    app.vm.provision "chef_solo" do |chef|
      chef.add_recipe "node::default"
    end
    app.vm.provision "shell", inline: set_env({ DB_HOST: "mongodb://192.168.10.150:27017/posts" }), privileged: false
    # app.vm.provision "chef_solo" do |chef|
    #   chef.cookbooks_path = ["cookbooks", "node"]
    # end
  end

  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"
    db.vm.network "private_network", ip: "192.168.10.150"
    db.hostsupdater.aliases = ["database.local"]
    db.vm.synced_folder "environment/db", "/home/ubuntu/environment"
    db.vm.provision "chef_solo" do |chef|
      chef.add_recipe "mongo::default"
    end
    db.vm.provision "shell", path: "environment/db/provision.sh", privileged: false
  end
end
