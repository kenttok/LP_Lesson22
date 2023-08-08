MACHINES = {
  :"pam" => {
              :box_name => "centos/7",
              :cpus => 2,
              :memory => 1024,
              :ip => "192.168.57.10",
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Отключаем сетевую папку
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Добавляем сетевой интерфейс
    config.vm.network "private_network", ip: boxconfig[:ip]
    # Применяем параметры, указанные выше
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      box.vm.provision "shell", inline: <<-SHELL
#Разрешаем подключение пользователей по SSH с использованием пароля
sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
#Перезапуск службы SSHD
systemctl restart sshd.service
# Скопируем файл-скрипт login.sh в /usr/local/bin/
cp /vagrant/login.sh /usr/local/bin/
# Добавим права на исполнение файла:
chmod +x /usr/local/bin/login.sh
# Создаём пользователя otusadm и otus:
useradd otusadmin && sudo useradd otus
# Создаём пользователям пароли:
echo "Otus2022!" | sudo passwd --stdin otusadmin && echo "Otus2022!" | sudo passwd --stdin otus
#Для примера мы указываем одинаковые пароли для пользователя otus и otusadm
# Создаём группу admin:
sudo groupadd -f admin
# Добавляем пользователей vagrant,root и otusadm в группу admin:
usermod otusadmin -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
# Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:
sed -i '/pam_nologin\.so$/a account    required     pam_exec.so \/usr\/local\/bin\/login\.sh' /etc/pam.d/sshd
  	  SHELL
    end
  end
end
