# ****Первые шаги с Ansible**** #

### Описание домашнего задания ###

Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible необходимо развернуть nginx со следующими условиями:
1. Необходимо использовать модуль yum/apt
2. Конфигурационный файлы должны быть взяты из шаблона jinja2 с переменными
3. После установки nginx должен быть в режиме enabled в systemd
4. Должен быть использован notify для старта nginx после установки
5. Cайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible

### **Выполнение** ###

Задание выполняется на рабочей станции с ОС Ubuntu 22.04.4 LTS с заранее установленными Vagrant 2.4.1 и VirtualBox 7.0. Перед выполнением предварительно подготовлен репозиторий <https://github.com/ConstantaNF/Ansible> по методичке.

### **Установка Ansible** ###

Так как README.md пишу на этапе, когда домашнее задание уже выполнено, в выводе команд уже присутствуют заранее подготовленный каталог `/home/adminkonstantin/Ansible/`, а также путь до файла ansible.cfg `/home/adminkonstantin/Ansible/ansible.cfg`.
Установку произвожу по инструкции из оффициальной документации <https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu>.
Проверяю версию установленного ПО:

`adminkonstantin@2OSUbuntu:~/Ansible$ ansible --version`

Результат:

`ansible [core 2.16.4]
  config file = /home/adminkonstantin/Ansible/ansible.cfg
  configured module search path = ['/home/adminkonstantin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/adminkonstantin/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True`

### **Подготовка окружения** ###

Для развёртывания управляемой ВМ посредством Vagrant использую Vagrantfile из методички <https://drive.google.com/file/d/17MEtg20TFSjKil6ih7PvPez7jmCvo6fb/view?usp=share_link>.
Данный Vagrantfile кладу в заранее подготовленный каталог Ansible `/home/adminkonstantin/Ansible/`.
Так как VPN не использую, буду качать необходимый box вручную из Vagrant Cloud <https://app.vagrantup.com/boxes/search>.
Смотрим содержимое Vagrantfile, предварительно перейдя в целевой каталог:

`adminkonstantin@2OSUbuntu:~$ cd Ansible/`
`adminkonstantin@2OSUbuntu:~/Ansible$ nano Vagrantfile`

Результат:

`MACHINES = {
  :nginx => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "nginx",
        :net => [
           ["192.168.11.150",  2, "255.255.255.0", "mynet"],
        ]
  }
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      box.vm.provider "virtualbox" do |v|
        v.memory = 768
        v.cpus = 1
       end
      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
      end
      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
    end
  end
end`

Из файла видно, что нам нужен box generic/ubuntu2204. Ищем его в Vagrant Cloud <https://app.vagrantup.com/generic/boxes/ubuntu2204> и качаем версию под VirtualBox:

`adminkonstantin@2OSUbuntu:~/Ansible$ wget https://app.vagrantup.com/generic/boxes/ubuntu2204/versions/4.3.12/providers/virtualbox/amd64/vagrant.box`

Результат:

`--2024-03-16 10:22:49--  https://app.vagrantup.com/generic/boxes/ubuntu2204/versions/4.3.12/providers/virtualbox/amd64/vagrant.box
Распознаётся app.vagrantup.com (app.vagrantup.com)… 54.204.238.15, 54.209.91.188, 54.221.251.148, ...
Подключение к app.vagrantup.com (app.vagrantup.com)|54.204.238.15|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 302 Found
Адрес: https://app.vagrantup.com/generic/boxes/ubuntu2204/versions/4.3.12/providers/virtualbox/amd64/download/vagrant.box [переход]
--2024-03-16 10:22:50--  https://app.vagrantup.com/generic/boxes/ubuntu2204/versions/4.3.12/providers/virtualbox/amd64/download/vagrant.box
Повторное использование соединения с app.vagrantup.com:443.
HTTP-запрос отправлен. Ожидание ответа… 302 Found
Адрес: https://archivist.vagrantup.com/v1/object/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJib3hlcy8yN2U2MjRhZC01ZDk2LTQzODAtODQ2ZS1iOGFlMGE4ZjQ5MDYiLCJtb2RlIjoiciIsImV4cGlyZSI6MTcxMDU3NDY3MH0.LKEKinebtErWM0wfG4cn45MtB9yKpg4dHUJ6dmpXBao [переход]
--2024-03-16 10:22:50--  https://archivist.vagrantup.com/v1/object/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJib3hlcy8yN2U2MjRhZC01ZDk2LTQzODAtODQ2ZS1iOGFlMGE4ZjQ5MDYiLCJtb2RlIjoiciIsImV4cGlyZSI6MTcxMDU3NDY3MH0.LKEKinebtErWM0wfG4cn45MtB9yKpg4dHUJ6dmpXBao
Распознаётся archivist.vagrantup.com (archivist.vagrantup.com)… 52.204.242.176, 54.157.58.70, 54.162.128.250, ...
Подключение к archivist.vagrantup.com (archivist.vagrantup.com)|52.204.242.176|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 307 Temporary Redirect
Адрес: https://vagrantcloud-files-production.s3-accelerate.amazonaws.com/archivist/boxes/27e624ad-5d96-4380-846e-b8ae0a8f4906?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIA6NDPRW4BYXO575GF%2F20240316%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240316T072251Z&X-Amz-Expires=900&X-Amz-Security-Token=FwoGZXIvYXdzEKn%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDDYC9BAR95k2gsL4KiK3AVS95mfYzxaPdcJLu4gBSO10J6eHO0Y4LrL6YXIW%2FbLt2wu7Bk35TOvJZ55TxttLZc06BeUaDf5CiscDT4dnBGJfryws47nxCGc3Ut8Xj2kyJ4nnoi9Lj7eSxIwzqzlilIbZnPicMUpkFDxGD6BsF1rw%2BtKKFw1trJ2bX8BiJOoQb4eiHH%2B2yZ%2BYDGLBuBgKZL3negOpZa89ffu3sDF%2BBD7JTazj0YFh2zvaMB6Ap4rF%2BVlDdijq5SijjNWvBjItBWHlOjw0rzXTCvoR09BMNnjyc8nhhNMIIlXbj9BBkW%2FhDxNRc5wKVZycn7%2FI&X-Amz-SignedHeaders=host&X-Amz-Signature=f48e74c12ec2d75ef7aff16b237d39391994dd4e4f896e1b5f3da567c3dcb664 [переход]
--2024-03-16 10:22:51--  https://vagrantcloud-files-production.s3-accelerate.amazonaws.com/archivist/boxes/27e624ad-5d96-4380-846e-b8ae0a8f4906?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIA6NDPRW4BYXO575GF%2F20240316%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240316T072251Z&X-Amz-Expires=900&X-Amz-Security-Token=FwoGZXIvYXdzEKn%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDDYC9BAR95k2gsL4KiK3AVS95mfYzxaPdcJLu4gBSO10J6eHO0Y4LrL6YXIW%2FbLt2wu7Bk35TOvJZ55TxttLZc06BeUaDf5CiscDT4dnBGJfryws47nxCGc3Ut8Xj2kyJ4nnoi9Lj7eSxIwzqzlilIbZnPicMUpkFDxGD6BsF1rw%2BtKKFw1trJ2bX8BiJOoQb4eiHH%2B2yZ%2BYDGLBuBgKZL3negOpZa89ffu3sDF%2BBD7JTazj0YFh2zvaMB6Ap4rF%2BVlDdijq5SijjNWvBjItBWHlOjw0rzXTCvoR09BMNnjyc8nhhNMIIlXbj9BBkW%2FhDxNRc5wKVZycn7%2FI&X-Amz-SignedHeaders=host&X-Amz-Signature=f48e74c12ec2d75ef7aff16b237d39391994dd4e4f896e1b5f3da567c3dcb664
Распознаётся vagrantcloud-files-production.s3-accelerate.amazonaws.com (vagrantcloud-files-production.s3-accelerate.amazonaws.com)… 52.85.48.12
Подключение к vagrantcloud-files-production.s3-accelerate.amazonaws.com (vagrantcloud-files-production.s3-accelerate.amazonaws.com)|52.85.48.12|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 1977638250 (1,8G) [binary/octet-stream]
Сохранение в: ‘vagrant.box’
vagrant.box                                     100%[=======================================================================================================>]   1,84G  6,23MB/s    за 5m 42s  
2024-03-16 10:28:33 (5,51 MB/s) - ‘vagrant.box’ сохранён [1977638250/1977638250]`

Разворачиваю box:

`adminkonstantin@2OSUbuntu:~/Ansible$ vagrant box add --name "generic/ubuntu2204" ./vagrant.box`

Результат:

`==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'generic/ubuntu2204' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/adminkonstantin/Ansible/vagrant.box
==> box: Successfully added box 'generic/ubuntu2204' (v0) for ''!`

Стартую ВМ:

`vagrant up`

Результат:

`Bringing machine 'nginx' up with 'virtualbox' provider...
==> nginx: Clearing any previously set forwarded ports...
==> nginx: Clearing any previously set network interfaces...
==> nginx: Preparing network interfaces based on configuration...
    nginx: Adapter 1: nat
    nginx: Adapter 2: intnet
==> nginx: Forwarding ports...
    nginx: 22 (guest) => 2222 (host) (adapter 1)
==> nginx: Running 'pre-boot' VM customizations...
==> nginx: Booting VM...
==> nginx: Waiting for machine to boot. This may take a few minutes...
    nginx: SSH address: 127.0.0.1:2222
    nginx: SSH username: vagrant
    nginx: SSH auth method: private key
    nginx: Warning: Connection reset. Retrying...
    nginx: Warning: Remote connection disconnect. Retrying...
==> nginx: Machine booted and ready!
==> nginx: Checking for guest additions in VM...
    nginx: The guest additions on this VM do not match the installed version of
    nginx: VirtualBox! In most cases this is fine, but in rare cases it can
    nginx: prevent things such as shared folders from working properly. If you see
    nginx: shared folder errors, please make sure the guest additions within the
    nginx: virtual machine match the version of VirtualBox you have installed on
    nginx: your host and reload your VM.
    nginx: 
    nginx: Guest Additions Version: 6.1.38
    nginx: VirtualBox Version: 7.0
==> nginx: Setting hostname...
==> nginx: Configuring and enabling network interfaces...
==> nginx: Machine already provisioned. Run 'vagrant provision' or use the '--provision'
==> nginx: flag to force provisioning. Provisioners marked to run always will still run.`

Проверяю доступность созданной ВМ по ssh:

`adminkonstantin@2OSUbuntu:~/Ansible$ vagrant ssh`

Результат:

`vagrant@nginx:~$`

ВМ доступна по ssh.

Далее выполняю последовательность действий из методички по созданию inventory, ansible.cfg и плейбука nginx.yml для установки и настройки web-сервера NGINX.
Для проверки работоспособности созданного web-сервера NGINX подключаюсь к ВМ, развёрнутой в Vagrant:

`adminkonstantin@2OSUbuntu:~/Ansible$ vagrant ssh`

Проверяю доступность сайта на порту 8080 через консоль:

`vagrant@nginx:~$ curl http://192.168.11.150:8080`

Результат:

`<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working.`

Задаание выполено. 
Для проверки создаю новый репозиторий github <https://github.com/ConstantaNF/Ansible> по инструкции из методички и отправляю туда нужные файлы.
Добавляем в каталог систему контроля версий git:

`adminkonstantin@2OSUbuntu:~/Ansible$ git init`

Результат:

`подсказка: Using 'master' as the name for the initial branch. This default branch name
подсказка: is subject to change. To configure the initial branch name to use in all
подсказка: of your new repositories, which will suppress this warning, call:
подсказка: 
подсказка: 	git config --global init.defaultBranch <name>
подсказка: 
подсказка: Names commonly chosen instead of 'master' are 'main', 'trunk' and
подсказка: 'development'. The just-created branch can be renamed via this command:
подсказка: 
подсказка: 	git branch -m <name>
Инициализирован пустой репозиторий Git в /home/adminkonstantin/Ansible/.git/`

Добавляем в неё нужные файлы:

`adminkonstantin@2OSUbuntu:~/Ansible$ git add nginx.yml Vagrantfile README.md`

Фиксируем изменения:

`adminkonstantin@2OSUbuntu:~/Ansible$ git commit -m "first commit"`

Результат:

`[master (корневой коммит) d84982e] first commit
 3 files changed, 292 insertions(+)
 create mode 100644 README.md
 create mode 100644 Vagrantfile
 create mode 100644 nginx.yml`

 Делаю эту ветку репозитория главной:

 `adminkonstantin@2OSUbuntu:~/Ansible$ git branch -M main`

 Добавляю информацию о созданном репозитории в GitHub:

 `git remote add origin https://github.com/ConstantaNF/Ansible.git`

 Отправляю файлы в удалённый репозиторий GitHub:

 `adminkonstantin@2OSUbuntu:~/Ansible$ git push -u origin main`

 Результат:

 `Username for 'https://github.com': ConstantaNF
Password for 'https://ConstantaNF@github.com': 
Перечисление объектов: 5, готово.
Подсчет объектов: 100% (5/5), готово.
При сжатии изменений используется до 4 потоков
Сжатие объектов: 100% (5/5), готово.
Запись объектов: 100% (5/5), 5.85 КиБ | 1.46 МиБ/с, готово.
Всего 5 (изменений 0), повторно использовано 0 (изменений 0), повторно использовано пакетов 0
To https://github.com/ConstantaNF/Ansible.git
[new branch]      main -> main
Ветка «main» отслеживает внешнюю ветку «main» из «origin».`

