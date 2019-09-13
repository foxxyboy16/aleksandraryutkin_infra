# Homework-3 Знакомство с облачной инфраструктурой. Google Cloud Platform

## Создание VM и подключение к ним

В GCP создаются две VМ - bastion c внешним и внутренним ip, someinternalhost только с внутренним ip  
Подключение к bastion, используя в параметрах подключения ключ -A, чтобы явно включить SSH Agent
Forwarding  
```ssh -i ~/.ssh/id_rsa -A aleksandraryutkin@35.233.7.146```  
Подключение к someinternalhost после подключения к bastion  
```ssh 10.132.0.3```  

## Подключение в одну команду

Подключение к someinternalhost одной командой осуществляется с помощью ключа -J директивы ProxyJump  
```ssh -J aleksandraryutkin@35.233.7.146 aleksandraryutkin@10.132.0.3```  

## Данные для подключения

bastion_IP = 35.233.7.146  
someinternalhost_IP = 10.132.0.3  

## VPN

Создана схема с VPN-сервером Pritunl, сгенерирована конфигурация VPN-клиента, что позволяет получить доступ в частную сеть облака

## Возникшие сложности

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

## Возникшие сложности

Столкнулся с проблемой, что не подходил приватный ключ для MongoDB в скрипте, заменил на подходящий

# Homework-5 Сборка образов VM при помощи Packer

## Подготовка

Установка packer.  
Установка ADC (Application Default Credentials) для аутентификации и управлению ресурсами GCP своего акаунта.  
Формирование секции “builders" для создания виртуальной машины для билда и секции "provisioners", позволяющнй устанавливать нужное ПО

## Создание и использование image

Запуск build образа  
Создание инстанса, который использует созданный образ  
Подключение по ssh к VM  
Деплой и запуск приложения

## Использование переменных и дополнительных атрибутов

Вынос определенных параметров в переменные
Добавление дополнительных атрибутов при билде образа
