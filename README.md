# Homework-3 Знакомство с облачной инфраструктурой. Google Cloud Platform

В GCP создаются 2 ВМ - bastion c внешним и внутренним ip, someinternalhost только с внутренним ip
Подключение к bastion, используя в параметрах подключения ключ -A, чтобы явно включить SSH Agent
Forwarding
```ssh -i ~/.ssh/id_rsa -A aleksandraryutkin@35.233.7.146```
Подключение к someinternalhost после подключения к bastion
```ssh 10.132.0.3```

Подключение к someinternalhost одной командой осуществляется с помощью ключа -J директивы ProxyJump
```ssh -J aleksandraryutkin@35.233.7.146 aleksandraryutkin@10.132.0.3```
Данные для подключения
bastion_IP = 35.233.7.146
someinternalhost_IP = 10.132.0.3
Создана схема с VPN-сервером Pritunl, сгенерирована конфигурация VPN-клиента, что позволяет получить доступ в частную сеть облака
Для конфигурационного файла cloud-bastion на локальной машине закомментирована строчка dev-type tun, так как MacOS не поддерживает нужные кексты

# Homework-4 Деплой тестового приложения

## Перенос файлов `setupvpn.sh` и `cloud-bastion.ovpn` в директорию /VPN

mkdir VPN
git mv setupvpn.sh VPN
git mv cloud-bastion.ovpn VPN

## Создается новый инстанс

gcloud compute instances create reddit-app \
--boot-disk-size=10GB \
--image-family ubuntu-1604-lts \
--image-project=ubuntu-os-cloud \
--machine-type=g1-small \
--tags puma-server \
--restart-on-failure

## Данные для подключения

testapp_IP = 35.241.245.45
testapp_port = 9292

## Создание скриптов для установки Ruby, MongoDB и деплой приложения

install_ruby.sh
install_mongodb.sh
deploy.sh
