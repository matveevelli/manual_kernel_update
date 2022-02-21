# Установка ПО
Среда выполнения:
Домашний сервер с Ubuntu 18.04.2 LTS
## Установка Virtualbox
Версия VirtualBox - 5.2.42_Ubuntur137960
```
sudo apt-get update
sudo apt-get install virtualbox
```
## Установка Vagrant
Версия Vagrant - 2.2.19
```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vagran
```
## Установка Packer
```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer
```
# Kernel update
Создан форк репозитория, клонирован на сервер
```
git clone git@github.com:matveevelli/manual_kernel_update.git
```
перелогинился под su потому что были пробелемы с правами
```
sudo su
```
запустил машину из каталога manual_kernel_update:
```
sudo vagrant up
```
Подключился по ssh. Проверил версию ядра
```
root@matveevs:/home/matveev/manual_kernel_update# vagrant ssh
[vagrant@kernel-update ~]$ uname -r
3.10.0-1127.el7.x86_64
```
Обновил ядро и перезагрузился
```
sudo yum install -y http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo grub2-set-default 0
sudo reboot
```
зашел на виртуалку
```
vagrant ssh
uname -r
```
Новая версия ядра:
5.16.10-1.el7.elrepo.x86_64

## Packer
Теперь необходимо создать свой образ системы, с уже установленым ядром 5й версии. Для это воспользуемся ранее установленной утилитой packer
в каталоге ./packer
### Изменения в файле centos.js:
"headless": "true",
**Отключает запуск ui vb, иначе ошибка NS_ERROR_FAILURE (0x80004005)**

### Создание образа Packer
В каталоге packer выполнил команду исправления т.к была ошибка легаси
```
packer fix centos.json > centos_new.json
```
сбилдил
```
packer build centos.json
```
Builds finished. Создался box в текущем каталоге

## Vagrant cloud
Зарегистрировался в Vagrant cloud, создал box. Загрузил бокс через консоль авторизовавшись по инструкции
```
# vagrant cloud auth login
Vagrant Cloud username or email: <user_email>
Password (will be hidden): 
Token description (Defaults to "Vagrant login from DS-WS"):
You are now logged in.
```

## Проверка
Так как уже был запущен образ в vagrant, выполнил:
```
vagrant destroy
```
Изменил в Vagrantfile `box_name` на `matveevelli/centos-7-5`. Выполнил ``vagrant up`` для проверки. Поднялась машина с нужной версией ядра `5.16.10-1.el7.elrepo.x86_64`
