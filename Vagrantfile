Vagrant.configure("2") do | config|
   config.vm.box = "centos/7"
   config.vm.box_version = "2004.01"

   config.vm.provider :virtualbox do |v|
      v.memory = 1024
      v.cpus = 2
   end

   boxes = [
      { :name => "web",
        :ip => "192.168.50.10",
      },
      { :name => "log",
        :ip => "192.168.50.15",
      }
   ]

   boxes.each do |opts|
      config.vm.define opts[:name] do |config|
         config.vm.hostname = opts[:name]
         config.vm.network "private_network", ip: opts[:ip]
         config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/me.pub"
         config.vm.provision "shell", inline: <<-SHELL
            cat /home/vagrant/.ssh/me.pub >> /home/vagrant/.ssh/authorized_keys
            #Разрешаем подключение пользователей по SSH с использованием пароля
            sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
            #Перезапуск службы SSHD
            systemctl restart sshd.service
         SHELL
      end
   end
   config.vm.synced_folder '.', '/vagrant', disabled: true
end
