<span style="color:blue">
VAGRANT-СТЕНД ДЛЯ ОБНОВЛЕНИЯ ЯДРА И СОЗДАНИЯ ОБРАЗА СИСТЕМЫ
</span>

####             ЦЕЛЬ ДОМАШНЕГО ЗАДАНИЯ:
Научиться обновлять ядро в ОС Linux. Получение навыков работы с Vagrant, Packer и публикацией готовых образов в Vagrant Cloud. 

#### ОПИСАНИЕ ДОМАШНЕГО ЗАДАНИЯ
1) Обновить ядро ОС из репозитория ELRepo
2) Создать Vagrant box c помощью Packer
3) Загрузить Vagrant box в Vagrant Cloud

#### ДОПОЛНИТЕЛЬНЫЕ ЗАДАНИЯ:
1) Ядро собрано из исходников
2) В образе нормально работают VirtualBox Shared Folders

#### СОЗДАЕМ СТРУКТУРУ КАТАЛОГОВ:
```
packer/
    └── centos.json                     
├── http
│   └── ks.cfg                         
├── scripts
│   ├── stage-1-kernel-update.sh
│   ├── stage-2-clean.sh
│   └── stage-3-GuestAdditions.sh
```
### В ФАЙЛЕ CENTOS.JSON МЕНЯЕМ ССЫЛКУ НА ОБРАЗ, Т.К. ССЫЛКА ИЗ МЕТОДИЧИКИ НЕРАБОЧАЯ:
`
"iso_url": "http://centos-mirror.rbc.ru/pub/centos/8-stream/isos/x86_64/CentOS-Stream-8-20230429.0-x86_64-boot.iso"
`
### ТАКЖЕ МЕНЯЕМ КОНТРОЛЬНУЮ СУММУ:
`
"iso_checksum": "017e6f8924248c204fe649403e0fe6896302a6b3c6b5a69968889758d805df26"
`
### МЕНЯЕМ КОМАНДУ ВЫКЛЮЧЕНИЯ (УЖЕ НЕ ПОМНЮ ЗАЧЕМ)
`
"shutdown_command": "echo 'vagrant' | sudo -S shutdown"
`
### СТАВИМ БОЛЬШЕЕ ЗНАЧЕНИЕ Т.К. PACKER СОБИРАЕТ НЕ БЫСТРО
`
"ssh_timeout": "40m"
`
### ДОБАВЛЯЕМ КОНТРОЛЛЕР С GUEST ADDITIONS
```
"vboxmanage": [
       [
          "storageattach",
          "{{.Name}}",
          "--storagectl",
          "IDE Controller",
          "--port",
          "1",
          "--device",
          "0",
          "--type",
          "dvddrive",
          "--medium",
          "/usr/share/virtualbox/VBoxGuestAdditions.iso"

        ]
```
### НЕМНОГО МЕНЯЕМ (НЕ ПОМНЮ ЗАЧЕМ)        
`
"execute_command": "echo 'vagrant'| {{.Vars}} sudo -S -E bash '{{.Path}}'"
`
### МЕНЯЕМ СКРИПТЫ 
#### В СТРОЧКУ УСТАНОВКИ ЯДРА ДОБАВЛЯЕМ УСТАНОВКУ ЗАГОЛОВОЧНЫХ ФАЙЛОВ, УТИЛИТ И БИБЛИОТЕК(БЕЗ НИХ ОТКАЗЫВАЮТСЯ РАБОТАТЬ ГОСТЕВЫЕ ДОПОЛНЕНИЯ)
`yum --enablerepo elrepo-kernel install --allowerasing kernel-ml kernel-ml-devel kernel-ml-core kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs -y
`
#### ТАКЖЕ УСТАНАВЛИВАЕМ УТИЛИТЫ ДЛЯ УСТАНОВКИ МОДУЛЕЙ ЯДРА
`yum install gcc make perl tar bzip2 -y
`
#### УДАЛЯЕМ МОДУЛИ СТАРОГО ЯДРА
`
yum remove kernel-modules-4.18.0-492.el8.x86_64 kernel-core-4.18.0-492.el8.x86_64 kernel-4.18.0-492.el8.x86_64 -y
`
#### ДОБАВЛЯЕМ ЕЩЕ ОДИН СКРИПТ ДЛЯ УСТАНОВКИ ГОСТЕВЫХ ДОПОЛНЕНИЙ
`
!/bin/bash
`
##### Создание папки для гостевых дополнений
`
sudo mkdir /media/GA
`
##### Монтирование образа гостевых дополнений
`
sudo mount /home/vagrant/VBoxGuestAdditions.iso /media/GA
`
##### Запуск установки гостевых дополнений
`
sudo /media/GA/./VBoxLinuxAdditions.run
`
##### Перезагрузка ВМ
`
shutdown -r now
`
