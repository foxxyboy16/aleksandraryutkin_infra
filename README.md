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
Формирование секции “builders" для создания виртуальной машины для билда и секции "provisioners", позволяющей устанавливать нужное ПО

## Создание и использование image

Запуск build образа  
Создание инстанса, который использует созданный образ  
Подключение по ssh к VM  
Деплой и запуск приложения

## Использование переменных и дополнительных атрибутов

Вынос определенных параметров в переменные  
Добавление дополнительных атрибутов при билде образа

# Homework-6 Практика IaC с использованием Terraform

## Установка terraform

Для курсов необходимо установить конкреткную версию terraform 0.11.11.  
Гайд по установке для MacOs  
https://blog.gruntwork.io/installing-multiple-versions-of-terraform-with-homebrew-899f6d124ff9  

## Описание конфигурационного файла main.tf

Указывается используемую версию terraform  
Описываеися секция Provider, которая позволяет Terraform управлять ресурсами GCP через API вызовы  
После описания запускается команда для загрузки провайдеров  

```terraform init```  

Добавляется ресурс для создания инстанса VM в GCP  
Планируются изменения и просматривается информация по ним  

```terraform plan```

Изменения применяются  

```terraform apply```

Добавляются ssh данные для подключения к VM в секции metadata  
Интересующая информацию - внешний адрес VM - выносится в выходную переменную (output variable)  
Добавляется правило файервола в виде ресурса, открывая порт для приложения  
Добавляется тег инстанса, который был указан в правиле файервола  
Внутри ресурса VM добавлеются Provisioners, которые запускают приложение  
Выносим часть параметров во входные переменные  

## Проверка работы приложения

Удаляем ранее установленные инстансы

```terraform destroy```

Планируем, применяем, проверям работу приложения (external_ip:port)

# Homework-7 Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform.  

## Правила файервола  

Найти интересующее правило файервола в секции default  

```gcloud compute firewall-rules list```  

Добавить ресурс для правила в конфиг main.tf
Импортировать информацию о созданном без помощи Terraform ресурсе в state файл

```terraform import google_compute_firewall.firewall_ssh default-allow-ssh```

Применить изменения

## Взаимосвязи ресурсов  

Задать IP для инстанса с приложением в виде внешнего ресурса в конфигурационном файле main.tf  
Сослаться на атрибуты ресурса, который этот IP создает, внутри конфигурации ресурса VM. В итоге создалась неявная зависимость одного ресурса от другого (также существуют явные зависимости, где используется параметр depends on)

## Структуризация ресурсов  

Вынести БД в отдельный инстанс VM  
Создать 2 VM (приложение и БД), разделив конфиг main.tf на два конфига app.tf и db.tf. При этом необходимо вынести правило файервола для ssh доступа, которое будет применимо для всех инстансов нашей сети в отдельный конфиг vpc.tf  
Применить новую конфигурацию  

## Модули  

Создать модули app, db и vpc каждый в отдельной директории со стандартным набором конфигурационных файлов  
Удалить ранее созданные конфигурационные файлы app.tf и db.tf  
Добавить модули в конфиг main.tf  
Для использования модулей запустить  

```terraform get```

Применить новую конфигурацию  
Параметризировать модуль vpc в конфиге main.tf для возможности указывать с каких ip адресов можно подключиться к VM  
Переиспользовать модули путем создания окружений stage и prod, указав для каждого из них ip, с которых можно получить доступ к VM (для stage доступ открыт для всех ip, для prod - только свой)  
Применить конфигурацию для обоих окружений  

## Работа с реестром модулей (на примере создания бакета в сервисе Storage)

Создать в директории terraform конфигурациооный файл storage-bucket.tf  
Применить конфигурацию  
Убедиться, что бакеты создались и доступны  
