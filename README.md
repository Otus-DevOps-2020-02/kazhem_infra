[![Build Status](https://travis-ci.com/Otus-DevOps-2020-02/kazhem_infra.svg?branch=master)](https://travis-ci.com/Otus-DevOps-2020-02/kazhem_infra)
# kazhem_infra
Kazhemskiy Mikhail OTUS-DevOps-2020-02 Infra repository

- [kazhem_infra](#kazhem_infra)
- [Домашние задания](#Домашние-задания)
  - [HomeWork 2: GitChatOps](#homework-2-gitchatops)
  - [HomeWork3: Знакомство с облачной инфраструктурой](#homework3-Знакомство-с-облачной-инфраструктурой)
  - [HomeWork4: Основные сервисы Google Cloud Platform (GCP)](#homework4-Основные-сервисы-google-cloud-platform-gcp)
  - [HomeWork5: Модели управления инфраструктурой Packer](#homework5-Модели-управления-инфраструктурой-packer)
    - [Задание со *](#Задание-со-)
  - [Homework6: Знакомство с Terraform](#homework6-Знакомство-с-terraform)
    - [Основное задание](#Основное-задание)
    - [Самостоятельные задания](#Самостоятельные-задания)
    - [Задание со *](#Задание-со--1)
    - [Задание с **](#Задание-с-)
  - [Homework7: Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform](#homework7-Принципы-организации-инфраструктурного-кода-и-работа-над-инфраструктурой-в-команде-на-примере-terraform)
    - [Самостоятельное задание](#Самостоятельное-задание)
    - [Задание со *](#Задание-со--2)
    - [Задание с **](#Задание-с--1)
  - [Homework8: Управление конфигурацией. Знакомство с Ansible](#homework8-Управление-конфигурацией-Знакомство-с-ansible)
    - [Задание со *](#Задание-со--3)
  - [Homework9: Продолжение знакомства с Ansible: templates, handlers, dynamic inventory, vault, tags](#homework9-Продолжение-знакомства-с-ansible-templates-handlers-dynamic-inventory-vault-tags)
    - [Задание со *](#Задание-со--4)
    - [Провижининг в Packer](#Провижининг-в-packer)
  - [Homework10 Ansible роли, управление настройками нескольких окружений и best practices](#homework10-ansible-роли-управление-настройками-нескольких-окружений-и-best-practices)
    - [Задание со *](#Задание-со--5)
    - [Задание с **](#Задание-с--2)
  - [Homework11 Разработка и тестирование Ansible ролей и плейбуков](#homework11-Разработка-и-тестирование-ansible-ролей-и-плейбуков)
    - [Доработка ролей](#Доработка-ролей)
    - [Задание со ☆](#Задание-со--6)

# Домашние задания
## HomeWork 2: GitChatOps
* Создан шаблон PR
* Создана интеграция с TravisCI
```bash
 travis encrypt "devops-team-otus:<ваш_токен>#<имя_вашего_канала>" --add notifications.slack.rooms --com
```
* Создана интеграция с чатом для репозитория
* Создана интеграция с чатом для TravisCI
* Отработаны навыки работы с GIT
## HomeWork3: Знакомство с облачной инфраструктурой

~~~
bastion_IP = 35.195.154.67
someinternalhost_IP = 10.132.0.3
~~~

* Создана УЗ для GCP
* Создана пара ssh ключей `~/.ssh/appuser` и публичная часть была добавлена в метаданные в Compute Engine GCP
* В Compute Engine были созданы две виртуальные машины - **bostion**, с внешним IP адресом `35.195.154.67` (и внутренним `10.132.0.2`) и **someinternalhost** только с внутренним IP адресом `10.132.0.3` (**без внешнего**)
* Для подключения **someinternalhost** необходимо сначала подключиться по **ssh** к хосту **bostion** с включенным SSH Agent Forwarding (параметр -A) и затем с него выполнить подключение по **ssh** к хосту `10.132.0.3`:
    ```bash
    ssh -A -i ~/.ssh/appuser appuser@35.195.154.67
    appuser@bastion:~$ ssh appuser@10.132.0.3
    appuser@someinternalhost:~$ hostname
    someinternalhost
    ```
 * Для подключения одной командой, минуя bastion хост можно:
    * добавить параметр `-J`  команде ssh:
        ```
        -J [user@]host[:port]
             Connect to the target host by first making a ssh connection to
             the jump host and then establishing a TCP forwarding to the ulti‐
             mate destination from there.  Multiple jump hops may be specified
             separated by comma characters.  This is a shortcut to specify a
             ProxyJump configuration directive.
        ```
       ```bash
       ssh -i ~/.ssh/appuser -J appuser@35.195.154.67 appuser@10.132.0.3
       appuser@someinternalhost:~$ hostname
       someinternalhost
       ```
    * через proxycommand с перенаправлением ввода:
      ```bash
      ssh appuser@10.132.0.3 -o "proxycommand ssh -W %h:%p -i ~/.ssh/appuser appuser@35.195.154.67"
      ```
* Для того, чтобы подключаться к **someinternalhost** через команду `ssh someinternalhost` необходимо создать файл `~/.ssh/config` добавив в него следующее содержимое:
    * Для ProxyJump:
        ```
        Host bastion
                HostName 35.195.154.67
                User appuser
                Port 22
                IdentityFile ~/.ssh/appuser
                ForwardAgent yes

        Host someinternalhost
                HostName 10.132.0.3
                User appuser
                Port 22
                IdentityFile ~/.ssh/appuser
                ProxyJump bastion
        ```
    * Для ProxyCommand:
        ```
        Host bastion
                HostName 35.195.154.67
                User appuser
                Port 22
                IdentityFile ~/.ssh/appuser
                ForwardAgent yes

        Host someinternalhost
                HostName 10.132.0.3
                User appuser
                Port 22
                IdentityFile ~/.ssh/appuser
                ProxyCommand ssh -W %h:%p bastion
        ```
 * Установлен и настроен VPN-сервер [pritunl](https://pritunl.com/)
   * Добавлены организация и пользователь
   * Добавлен сервер
   * Добавлено правило в брэндмауэр для доступа к внутренней сети
   * Создано доменное имя c помощью [sslip.io](https://sslip.io/): **35.195.154.67.sslip.io**, которое резолвитсяв IP 35.195.154.67. Домен был подписан сертификатом [LetsEncrypt]("https://letsencrypt.org/") с помощью [Certbot](https://certbot.eff.org/)
   * В нстройках pritunl (веб) был указан сертификат LetsEncrypt
   * Веб-интерфейс доступен по адресу https://35.195.154.67.sslip.io/ с валидным сертификатом

## HomeWork4: Основные сервисы Google Cloud Platform (GCP)
~~~

testapp_IP = 35.189.238.97
testapp_port = 9292

~~~
* Сделана установка и настройка [gcloud](https://cloud.google.com/sdk/docs/)
* С помощью утилиты gcloud была создана тестовая ВМ c названием **reddit-app**
    ```
    gcloud compute instances create reddit-app\
      --boot-disk-size=10GB \
      --image-family ubuntu-1604-lts \
      --image-project=ubuntu-os-cloud \
      --machine-type=g1-small \
      --tags puma-server \
      --restart-on-failure
    ```
* На ВМ были установлены Ruby, MongoDB а также Ruby приложение из указнного репозитория
* В настройка GCP было добавлено правило фаервола для доступа к порту `9292` приложения
* Команды по настройке системы и деплоя приложения были завернуты в скрипты
    * [install_ruby.sh](install_ruby.sh) - установка Ruby
    * [install_mongodb.sh](install_mongodb.sh) - установка MongoDB
    * [deploy.sh](deploy.sh)  - скачивание и запуск приложения
* Для настройки одной командой вышепеечисленные команды были добавлены в файл [startup_script.sh](startup_script.sh)
* Данный файл был добавлен в параметры gloud так, что при создании ВМ запускается cкрипт, который зустаноавливает зависимости и запускает нужное приложении (параметр `--metadata-from-file startup-script=<path_to_local_file>`):
    ```
    gcloud compute instances create reddit-app\
      --boot-disk-size=10GB \
      --image-family ubuntu-1604-lts \
      --image-project=ubuntu-os-cloud \
      --machine-type=g1-small \
      --tags puma-server \
      --restart-on-failure\
      --metadata-from-file startup-script=./startup_script.sh
    ```
 * Удалено, а затем добавлено через утилиту gcloud правило фаервола **default-puma-server**
    ```
    gcloud compute firewall-rules create default-puma-server \
      --allow tcp:9292 \
      --target-tags=puma-server
    ```
## HomeWork5: Модели управления инфраструктурой Packer
* Произведена установка [Packer](https://www.packer.io/downloads.html) на локальную машину
* Установлен и авторизовн **Application Default Credentials** (ADC) для того, чтобы Packer мог управлять ресурсами GCP через API вызовы
  ```
  gcloud auth application-default login
  ```
* Создан Packer template [ubuntu16.json](packer/ubuntu16.json)
    * Настроены **builders**, отвечающий за создание ВИ для билда
    * Настроены **provisioners** с типо _shell_, устанавливающие MongoDB и Ruby с помощью скриптов [install_mongodb.sh](packer/scripts/install_mongodb.sh) и [install_ruby.sh](packer/scripts/install_ruby.sh)
* [ubuntu16.json](packer/ubuntu16.json) проверен на наличие ошибок в синтакисисе
   ```
   packer validate ./ubuntu16.json
   ```
* Собран образ из шаблона [ubuntu16.json](packer/ubuntu16.json)
   ```
   packer build ubuntu16.json
   ```
* Создана ВМ через GUI GCP из собранного раннее образа
* Вручную, через SSH, установлен и запущен puma server
* Добавлены пользовательские переменные, обязателные вынесены в файл variables.json
    ```
    "variables": {
        "project_id": null,
        "source_image_family": null,
        "machine_type": "f1-micro",
        "ssh_username": "appuser"
    },
    ```
* Запущена сборка образа с пользвательскими переменными
  ```
  packer build --var-file variables.json ubuntu16.json
  ```
* Добавлены новые поля в builders:
  ```
  "image_description": "Reddit base app image with mongo and redis installed",
   "disk_size": 10,
   "disk_type": "pd-standard",
   "network": "default",
   "tags": ["puma-server"],
  ```
 ### Задание со *
 * Создан [puma.service](packer/files/puma.service) файл для автоматического запуска приложения
 * Создан файл [startup_script.sh](packer/scripts/startup_script.sh) для установки и настройки сервиса для старта прложения при запуске ВМ
 * Создан конфиг [immutable.json](packer/immutable.json) для packer, создающий образ семейства reddit-full с bake образом приложения
 * Создан файл [create-reddit-vm.sh](config-scripts/create-reddit-vm.sh),запускающий команду gloud, создающую ВМ на основе образа reddit-full.

## Homework6: Знакомство с Terraform
### Основное задание
* Установлен [Terraform](https://www.terraform.io/downloads.html)
  * [Getting Started Guide](https://www.terraform.io/docs/providers/google/guides/getting_started.html)
* Cоздан файл [main.tf](terrafiorm/main.tf), где были указазаны основные параметры подключения: версия terraform, provider (google), ID проекта и регион.
* Инициализоровна terraform:
  ```
  terraform init
  ```
* В файл [main.tf](terrafiorm/main.tf) добавлен ресурс [google_compute_instance](https://www.terraform.io/docs/providers/google/r/compute_instance.html) а так же boot_disk - образ, с которого создавать VM
* Основные команды:
  * `terraform plan` - запланировать изменения (посмотреть, что будет сделано при применении)
  * `terraform apply --auto-approve` - применить изменения с автоподтверждением
  * `terraform show` - показать текущее состаяние TF. К примерну показать внешний IP: `terraform show | grep nat_ip`
  * `terraform destroy` - удалить все созданные ресурсы
  * `terraform fmt` - отформатировать файлы *.tf до "правильного" вида (синтаксис)
* Добавлен SSH-ключ для подключения в metadata **инстанса**
* Добавлен файл [outputs.tf](terraform/outputs.tf) для создания выхоных переменных
* Добавлен ресурс *google_compute_firewall* для добавления правил фаервола
* Инстансу добавлен сетевые *tags* для применения правил фаервола
* Добавлены [Provisioners](https://www.terraform.io/docs/provisioners/index.html), которые вызываются только в момент **создания/удаления** ресурса
* В provisioners также добавлен файл *Systemd Unit service* [puma.service](terraform/files/puma.service) для автоматического запуска приожения reddit-app и скрипт деплоя [deploy.sh](terraform/files/deploy.sh)
* Добавлен файл с описанием  input переменных [variables.tf](terraform/variables.tf) и заменены в main.tf (var.{ИМЯ ПЕРЕМЕННОЙ})
  * Значения переменных вынесены в файл *terraform.tfvars*, который не индексирусется git-ом

### Самостоятельные задания
* Добавлены переменные зоны ресурса и приватного ключа для подключения провиженоров
* Отформатированы файлы настроек для паривльного вида `terraform fmt`
* Добавлен пример файла input переменных [terraform.tfvars.example](./terraform/terraform.tfvars.example)

### Задание со *
* Добавлен [ресурс](https://www.terraform.io/docs/providers/google/r/compute_project_metadata_item.html) `google_compute_project_metadata_item` для того, чтобы добавить ключи пользователя в метаданные проекта (***важно** - данные в секции EOF должны обязательно начинаться с самого начала строки - лишние пробелы GCP не отрезает и работает некорректно*):
  ```
  resource "google_compute_project_metadata_item" "ssh_keys" {
    key = "ssh-keys"
    value = <<EOF
  appuser1:${file(var.public_key_path)}
  appuser2:${file(var.appuser2_public_key_path)}
  EOF
  }
  ```
* Если добавить ssh ключ через веб интерфейс и это не будет соответсвовать настройкам терраформа, то при `terraform apply` такой ключ будет удален.

### Задание с **
При разработке использовались следующие ресурсы из описания [документации](https://cloud.google.com/load-balancing/docs/https/):


* Создан файл `lb.tf`, в котором описаны следующие сущности:
  * [google_compute_instance_group](https://www.terraform.io/docs/providers/google/r/compute_instance_group.html) со списком инстансов ВМ с запущенным приложением (1 экземпляр)
  * [google_compute_health_check](https://www.terraform.io/docs/providers/google/r/compute_health_check.html) для проверки доступности приложения на экземпляре ВМ
  * [google_compute_backend_service](https://www.terraform.io/docs/providers/google/r/compute_backend_service.html) со ссылкой на группы экземпляров ВМ (в данном случае на 1 группу), а так же со ссылкой на google_compute_health_check
  * [google_compute_url_map](https://www.terraform.io/docs/providers/google/r/compute_url_map.html) с описанием запросу к какому url на какой backend_service отправлять (в нашем случае все запросы ко всем url отправляются на 1 сервис)
  * [google_compute_target_http_proxy](https://www.terraform.io/docs/providers/google/r/compute_target_http_proxy.html) для проксирования http/https соединений к url_map
  * [google_compute_global_forwarding_rule](https://www.terraform.io/docs/providers/google/d/datasource_compute_forwarding_rule.html) для перенаправления ip4/ip6 трафика (для каждого типа трафика должно быть своё правило) на target_http_proxy (в нашем случае только ip4)
* *Backend service долго стартует*
* Был добавлен второй инстанс reddit-app (app2) в instance_group
  * При отключении puma сервера видно, что по health-check доступен только один из двух инстансов. При этом приложение через LB остается доступным.
  * При добавление нового инстанса копированием создается избыточное количество кода
* Для добавления N одинкаовых инстаносов добавлен параметр `count` в `google_compute_instance`. Сама переменая задается в `variables.tf`

## Homework7: Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform

* Заимпортирована текущие настройки terraform (ssh firewall rules)
  ```
  terraform import google_compute_firewall.firewall_ssh default-allow-ssh
  ```
* Добавлен ресурс [google_compute_address](https://www.terraform.io/docs/providers/google/r/compute_address.html) для того, чтобы иметь возможность обращаться к адресу инстанса из других ресурсов
  ```
  resource "google_compute_address" "app_ip" {
  name = "reddit-app-ip"
  }
  ```
* Для того чтобы использовать адрес внутри ресурса инстанса необходимо сослаться на него следующим образом (внутри инстанса ВМ). При этом ресурс ВМ становится зависимым от  ресурса `google_compute_address` и при создании инфорстрактуры создается после него.
  ```
  network_interface {
    network = "default"
    access_config {
    nat_ip = google_compute_address.app_ip.address
    }
  }
  ```
* В директории packer созданы шаблоны [db.json](packer/db.json) и [app.json](packer/app.json) а затем по ним созданы образы семейста `reddit-db-base` и `reddit-app-base` соотвественном. На первом установлена БД MongoDB, на втором - Ruby.
* Файл [main.tf](terraform/main.tf) разбит на несколько файлов, так, что в нем остался только описание версии тераформа и описание провайдера.
  * Созан файл [db.tf](terraform/db.tf) описывающий ВМ для БД
  * Созан файл [app.tf](terraform/app.tf) описывающий  ВМ для Reddit app
  * Создан файл [vpc.tf](terraform/vpc.tf) описывающие правила фаервола
* Созадны модули `db` и `app`. Описанные выше файлы `db.tf` и `app.tf` перенесены в эти модули.
  * После указание модулей в основном файле [main.tf](terraform/main.tf) необходимо подргрузить модули:
  ```
  terraform get
  ```
### Самостоятельное задание
* Добавлен модуль `vpc` (*переде применением не забыть заимпортить существующее правило фаервола*) по аналогии с модулями `db` и `app`
* Добавлена переменная `source_ranges` для установки IP адресов с которых будет возможен доступ по SSH и которая позволяет задавать ее при вызове модуля:
  ```
  #vpc/veriables.json
  variable source_ranges {
    description = "Allowed IP addresses"
    default = ["0.0.0.0/0"]
  }
  ```
  ```
  #main.tf
  module "vpc" {
    source          = "./modules/vpc"
    source_ranges = ["0.0.0.0/0"]
  }
  ```
  * Протестировано измениние этой переменный на свой IP (/32), а также на IP, отличный от него. Во втором случае доступ по SSH к машинам закрывется.

* В дериктории `terraform` добавлены папки `stage` и `prod`, каждая с набором файлов .tf для запуска инфаструктуры. Испульзуется одни и те же модули, но с разными параметрами (переиспользование). *При запуске терраформа **из новых папок** необходимо заного инициализировать терраформ*: `terraform init`
* Рассмотрены модули для google в [regisry HashiCorp](https://registry.terraform.io/browse/modules?provider=google)
* Добавлен модуль [storage-bucket](https://registry.terraform.io/modules/SweetOps/storage-bucket/google/0.3.1) и проверено, что создан бакет

### Задание со *
* В деррикториях stage и prod добвлен файл `backend.tf`, который описывает подключение удаленного стораджа [gcs](https://www.terraform.io/docs/backends/types/gcs.html) для хранения стейта.
  * При применении конфигурации терраформ больше не хранит локально файл `terraform.tfstate`
  * Одновременный запуск терраформа из двух дирректорий выдал ошибку, т.к. создаются одни и те же ресурсы.

### Задание с **
* Настроены provisioners для работы из модулей:
  * Необходимо было через переменные окружения передать `DATABASE_URL`. Было сделано через provisioner file с помощью атрибута `content`, который может записывать строку в файл:
  * Далее Unit service подкгружает переменные окружения из файла
  * Помимо прочего необходимо было сделать так, чтобы MongoDB поднималась не локалхосте, а, как минимум на 0.0.0.0. Поэтому при сборке образа с packer добалено изменение конфига mongod

## Homework8: Управление конфигурацией. Знакомство с Ansible
* Установлен [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* Создан [inventory.yml](ansible/inventory.yml) файл ([docs](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html))
* Протестирована работа модулей `shell`, `command`, `service`, `systemd` и сделан вывод о том, что использование специальных модулей для выполнения определенных команд (`systemd`, `service`) дают больше возможностей для дальнейшего использования вывода этих команд.
* Создан playbook [clone.yml](ansible/clone.yml), который использует модуль `git` для клонирования репозитория. Если при запуске плейбука директория после клонирования гита изменилась - ансибл пишет `changed`.
  ```
  ansible-playbook clone.yml

  PLAY [Clone] ***************************************************************************************

  TASK [Gathering Facts] *****************************************************************************
  ok: [appserver]

  TASK [Clone repo] **********************************************************************************
  changed: [appserver]

  PLAY RECAP *****************************************************************************************
  appserver                  : ok=2    changed=1    unreachable=0    failed=0
  ```
### Задание со *
* Создан файл inventory.json в [формате](https://medium.com/@Nklya/динамическое-инвентори-в-ansible-9ee880d540d6) динмического инвентори. Создан из существуего файла inventory.yml (или ini).
  ```
  ansible-inventory --list > inventory.json
  ```
  *  Данный способ не совсем корректен, т.к. динамичский инвентори генерируется обычно из каких либо внешних источников. К примерну его можно сгенерировать с помощью плагина [gcp_compute](https://docs.ansible.com/ansible/latest/scenario_guides/guide_gce.html#gce-dynamic-inventory), который собирет информацию о машинах в GCP (не используется в силу ограничения задания)
    ```
    # ansible-inventory --list  -i inventory.gcp.yml
    {
    "_meta": {
        "hostvars": {
            "35.195.204.42": {
                "canIpForward": false,
                "cpuPlatform": "Intel Haswell",
                "creationTimestamp": "2020-04-13T00:28:15.683-07:00",
                "deletionProtection": false,
                "disks": [
                    {
                        "autoDelete": true,
                        "boot": true,
                        "deviceName": "persistent-disk-0
      ...
    ```
* Создан python скрипт `json_inventory.py`, который вовзращает содержимое json-инвентори по параметру `--list` и данные хоста с параметром `--host`.
* Для того, чтобы использовать данный скрипт в качестве источника inventory по умолчаню, его можно прописать в `ansible.cfg` в качество значения ключа `inventory`:
  ```
  [defaults]
  inventory = ./json_inventory.py
  ...
  ```
* Вышеуказанный файл-плагин `inventory.gcp.yml` также можно указать в качестве инвентори, и тогда ansible будет динамически знать о вашей инфраструктуре GCP.
* Команда `ansible all -m ping` выполнена успешно

## Homework9: Продолжение знакомства с Ansible: templates, handlers, dynamic inventory, vault, tags
* Создан playbook [reddit_app.yml](ansible/reddit_app.yml)
  * Настроены сценарии для хоста MongoDB
    * Создан шаблон [mongod.conf.j2](ansible/templates/mongod.conf.j2), который с помощью модуля `templates` в плейбуке копирует, подставляя нужные значения переменных, шаблон настройки mongod на ВМ
    * Добавлены `tags` для того, чтобы была возможность запускать playbook только по каким-то определенным тагам.
    * Определены переменные в плейбке в секции `vars` - они будут переданы в шаблон
    * Добавлены `handlers`, которые вызываются из `tasks` по параметру `notify` - в данном случае `task`, при статусе `changed` вызывает `handler`, который рестартует сервис монги (с помощью модуля `service`)
    * Предварительный запуск `ansible-playbook` с параметром `--check` проходится по списку тасков, но не применяет их. Аналог `terraform plan`
      ```
      ansible-playbook reddit_app.yml --check --limit db
      ```
    * Применение конфигурации
      ```
      ansible-playbook reddit_app.yml --limit db

      PLAY [Configure hosts & deploy application] *******************************************************************

      TASK [Gathering Facts] ****************************************************************************************
      ok: [dbserver]

      TASK [Change mongo config file] *******************************************************************************
      changed: [dbserver]

      RUNNING HANDLER [restart mongod] ******************************************************************************
      changed: [dbserver]

      PLAY RECAP ****************************************************************************************************
      dbserver                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
      ```
  * Настроены сценарии для приложения
    * С помощью модуля `copy` скопирован `unit.service` файл для сервера puma
    * Добавлен таск, который через модуль `systemd` выставляет сирвису puma значение enabled
    * Добавлен handler для перезапуска через `systemd` сервиса `puma`
    * Добавлен шаблон для приложения, которые отвечает за определенных environment переменных
  * Настроен деплой с помощью модулей `git` и `bundler`
* Один play сценарий разделн на три (бд, приложение, деплой) в одном файле.
* Далее разделили на три разных файла (`app.yml`, `db.yml`, `deploy.yml`), которые объеденины по средством `import_playbook` в `site.yml`

###  Задание со *
* В качестве dynamic inventory использован плагин [gcp_compute](https://docs.ansible.com/ansible/latest/scenario_guides/guide_gce.html#gce-dynamic-inventory), который был сконфигурирован в прошлом ДЗ
  * [Пример](http://matthieure.me/2018/12/31/ansible_inventory_plugin.html) использования dynamic inventory вместе с параметром `keyed_groups` для группировки инстансов
  * В файл [inventory.gcp.yml](ansible/inventory.gcp.yml) добавлены разделы:
    * `hostnames` для отображения хостов по его имени в gcp
      ```
      hostnames:
      - name
      ```
    * `compose` для задания переменных, которые будут доступны в плейбуках (`hostvars`) - данные переменны и так доступны в
      ```
      compose:
      # add hostvar variables
      ansible_host: networkInterfaces[0].accessConfigs[0].natIP # external ip
      ansible_internal: networkInterfaces[0].networkIP # internal ip
      ```
      Чтобы посмотреть доступные переменные можно вывести их следущим образом из таска:
      ```
      - name: Debug
        debug:
          var: hostvars
        tags:
          - never
          - debug
      ```
    * `keyed_groups` для распределения хостов по группам в зависмости от заданного значения в GCP (в данном случае labels)
      ```
      keyed_groups:
        # Create groups from GCE labels
        - prefix: ""
          separator: ""
          key: labels['group']
      ```
  * В настройки инстанса `terraform` app для параметра группировки добавлено
    ```
    labels       = {
      group = "app"
    }
    ```
  * В настройки инстанса `terraform` db для параметра группировки  добавлено
    ```
    labels       = {
      group = "db"
    }
    ```
  * В итоге, укав в `ansible.cfg` путь до динмического инвентори по умолчанию:
    ```
    #ansible-inventory --graph
    @all:
      |--@app:
      |  |--reddit-app
      |--@db:
      |  |--reddit-db
      |--@ungrouped:
    ```
  * В файле шаблона `db_config.j2` изменено значение переменной на:
    ```
    DATABASE_URL={{ hostvars[groups['db'][0]]['ansible_internal']}}
    ```
### Провижининг в Packer

* Создан плейбук [packer_app.yml](ansible/packer_app.yml):
  * Устанавливает Ruby и Bundler с помощью модуля [apt](https://docs.ansible.com/ansible/latest/modules/apt_module.html#apt-module)
    ```
    tasks:
    - name: Install Ruby&Bundler
      apt:
        pkg:
          - ruby-full
          - ruby-bundler
          - build-essential
        update_cache: yes
    ```
* Создан плейбук [packer_db.yml](ansible/packer_db.yml):
  * Добавляет публичный ключ GPG с помощью модуля [apt_key](https://docs.ansible.com/ansible/latest/modules/apt_key_module.html#apt-key-module)
    ```
    - name: Import MongoDB public GPG Key
      apt_key:
          keyserver: hkp://keyserver.ubuntu.com:80
          id: 42F3E95A2C4F08279C4960ADD68FA50FEA312927
    ```
  * Добавлен репозиторий mongodb в список источников с помощью модуля [apt_repository](https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html)
    ```
    - name: Add MongoDB repository into sources list
      apt_repository:
        repo: deb http://repo.mongodb.org/apt/ubuntu {{ansible_distribution_release}}/mongodb-org/3.2 multiverse
        state: present
    ```
  * Установлен пакет `mongodb-org` c помощью [apt](https://docs.ansible.com/ansible/latest/modules/apt_module.html#apt-module)
    ```
    - name: Install MongoDB package
      apt:
        name: mongodb-org
        update_cache: yes
    ```
  * В [systemd](https://docs.ansible.com/ansible/latest/modules/systemd_module.html#systemd-module) разрешен сервис mongod
    ```
    - name: enable mongod
      systemd:
        name: mongod
        enabled: yes
    ```
* Эти плейбуки добавлены в качестве провиженоров в packer
  ```
  # packer/app.json
  ...
  "provisioners": [
      {
      "type": "ansible",
      "playbook_file": "ansible/packer_app.yml"
      }
  ]
  ...
  ```
  ```
  # packer/db.json
  ...
  "provisioners": [
    {
      "type": "ansible",
      "playbook_file": "ansible/packer_db.yml"
    }
  ]
  ...
  ```
* Созданы образы с помощью packer
  ```
  ...
  ==> googlecompute: Provisioning with Ansible...
  ==> googlecompute: Executing Ansible: ansible-playbook --extra-vars packer_build_name=googlecompute packer_builder_type=googlecompute -o IdentitiesOnly=yes -i /tmp/packer-provisioner-ansible187949315 /home/kazhem/develop/otus/kazhem_infra/ansible/packer_db.yml -e ansible_ssh_private_key_file=/tmp/ansible-key049835316
      googlecompute:
      googlecompute: PLAY [Install MongoDB] *********************************************************
  ...
  ```
* Созданы ВМ с помощью terraform
* Запущен плейбук site.yml - приложение работает


## Homework10 Ansible роли, управление настройками нескольких окружений и best practices

* Для хранения ролей создана дирректория `ansible/roles`
* Знакомство с [ansible-galaxy](https://galaxy.ansible.com/intro) - место хранения *community roles*
* С помощью `ansible-galaxy ` создана структура наших ролей `app` и `db`:
  ```
  $ ansible-galaxy init app
  $ ansible-galaxy init db
  ```
  ```
  $ ansible-galaxy init db
  - db was created successfully
  $ tree db
  db
  ├── defaults            # <-- Директория для переменных по умолчанию
  │   └── main.yml
  ├── files
  ├── handlers
  │   └── main.yml
  ├── meta                # <-- Информация о роли, создателе и зависимостях
  │   └── main.yml
  ├── README.md
  ├── tasks               # <-- Директория для тасков
  │   └── main.yml
  ├── tests
  │   ├── inventory
  │   └── test.yml
  └── vars                # <-- Директория для переменных, которые не должны
      └── main.yml
  6 directories, 8 files
  ```
* Роли вызываются из плейбуков из директории `roles`
  ```
  roles:
    - db
  ```

* В директории `ansible/enviroments` созданы окружения `stage` и `prod`
* Также туда перенес файл `inventory` (в качестве значения по умолчанию указан тот, что в окружении stage)
* Директория `group_vars`, созданная в директории плейбука или инвентори файла позволяет создавать файлы (имена, которых должны соответствовать названиям групп в инвентори файле) для определения переменных для группы хостов. (созданы)
* В ролях создан таск, для того чтобы отображать текущее окружение (через переменные group_vars группы `all`)
  ```
  - name: Show info about the env this host belongs to
    debug:
      msg: "This host is in {{ env }} environment!!!"
  ```
* Все плейбуки из корня директории `ansible` перенесены в дирректорию `ansible/playbooks`, а другие старые файлы перенесены в дирректорию `ansible/old`. В итоге получилась следущая структура дирректории `ansible`:
  <details>
  <summary>Подробнее</summary>
  ```
  ansible
  ├── ansible.cfg
  ├── environments
  │   ├── prod
  │   │   ├── credentials.yml
  │   │   ├── group_vars
  │   │   │   ├── all
  │   │   │   ├── app
  │   │   │   └── db
  │   │   ├── inventory
  │   │   ├── inventory.gcp.yml
  │   │   └── requirements.yml
  │   └── stage
  │       ├── credentials.yml
  │       ├── group_vars
  │       │   ├── all
  │       │   ├── app
  │       │   └── db
  │       ├── inventory
  │       ├── inventory.gcp.yml
  │       └── requirements.yml
  ├── old
  │   ├── files
  │   │   └── puma.service
  │   ├── inventory.json
  │   ├── inventory.yml
  │   ├── json_inventory.py
  │   └── templates
  │       ├── db_config.j2
  │       └── mongod.conf.j2
  ├── playbooks
  │   ├── app.yml
  │   ├── clone.yml
  │   ├── db.yml
  │   ├── deploy.yml
  │   ├── packer_app.yml
  │   ├── packer_db.yml
  │   ├── reddit_app_multiple_plays.yml
  │   ├── reddit_app_one_play.yml
  │   ├── site.yml
  │   └── users.yml
  ├── requirements.txt
  └── roles
      ├── app
      │   ├── defaults
      │   │   └── main.yml
      │   ├── files
      │   │   └── puma.service
      │   ├── handlers
      │   │   └── main.yml
      │   ├── meta
      │   │   └── main.yml
      │   ├── README.md
      │   ├── tasks
      │   │   └── main.yml
      │   ├── templates
      │   │   └── db_config.j2
      │   ├── tests
      │   │   ├── inventory
      │   │   └── test.yml
      │   └── vars
      │       └── main.yml
      ├── db
      │   ├── defaults
      │   │   └── main.yml
      │   ├── files
      │   ├── handlers
      │   │   └── main.yml
      │   ├── meta
      │   │   └── main.yml
      │   ├── README.md
      │   ├── tasks
      │   │   └── main.yml
      │   ├── templates
      │   │   └── mongod.conf.j2
      │   ├── tests
      │   │   ├── inventory
      │   │   └── test.yml
      │   └── vars
      │       └── main.yml
      └── jdauphant.nginx
          ├── ansible.cfg
          ├── defaults
          │   └── main.yml
          ├── handlers
          │   └── main.yml
          ├── meta
          │   └── main.yml
          ├── README.md
          ├── tasks
          │   ├── amplify.yml
          │   ├── cloudflare_configuration.yml
          │   ├── configuration.yml
          │   ├── ensure-dirs.yml
          │   ├── installation.packages.yml
          │   ├── main.yml
          │   ├── nginx-official-repo.yml
          │   ├── remove-defaults.yml
          │   ├── remove-extras.yml
          │   ├── remove-unwanted.yml
          │   └── selinux.yml
          ├── templates
          │   ├── auth_basic.j2
          │   ├── config_cloudflare.conf.j2
          │   ├── config.conf.j2
          │   ├── config_stream.conf.j2
          │   ├── module.conf.j2
          │   ├── nginx.conf.j2
          │   ├── nginx.repo.j2
          │   └── site.conf.j2
          ├── test
          │   ├── custom_bar.conf.j2
          │   ├── example-vars.yml
          │   └── test.yml
          ├── Vagrantfile
          └── vars
              ├── Debian.yml
              ├── empty.yml
              ├── FreeBSD.yml
              ├── main.yml
              ├── RedHat.yml
              └── Solaris.yml

  36 directories, 85 files
  ```
  </details>
* В [ansible.cfg](ansible/ansible.cfg) доабвлены селудюшие параметы:
  ```
  [defaults]
  ...
  # # Явно укажем расположение ролей (можно задать несколько путей через ; )
  roles_path = ./roles
  [diff]
  # Включим обязательный вывод diff при наличии изменений и вывод 5 строк контекста
  always = True
  context = 5
  ```
* Протестирована работа плейбуков после ретструктаризации
* Для установки `community-roles` из `ansible-galaxy` в папках окружений создают файл `requirements.yml`. Пример содержимого с ролью [jdauphant.nginx](https://github.com/jdauphant/ansible-role-nginx):
  ```
  - src: jdauphant.nginx
    version: v2.21.1
  ```
* Пример установки:
  ```
  ansible-galaxy install -r environments/stage/requirements.yml
  ```
* Комьюнити-роли не стоит коммитить в свой репозиторий, для этого в `.gitignore` добавлена запись:  `jdauphant.nginx`
* Переменные роли в group_vars/app:
  ```
  nginx_sites:
    default:
      - listen 80
      - server_name "reddit"
      - location / {
          proxy_pass http://127.0.0.1:порт_приложения;
        }
  ```
* В терраформ добавлено открытие 80 порта, роль nginx добавлена в плейбук `app.yml` (приложение после завпуска плейбука доступно на 80 порту)
* Добавлен плейбук users.yml, который создает пользователей в систме. Его перепенные хранятся в `environments/prod|stage/credentials.yml`
* Создан `vault.key` вне дирриктории проекта и добавлен в `ansible.cfg`:
  ```
  [defaults]
  ...
  vault_password_file = ~/.ansible/vault.key
  ```
* Файл `credentials.yml` зашифрован с помощью [ansible-vault](https://docs.ansible.com/ansible/devel/user_guide/vault.html)
  ```
  $ ansible-vault encrypt environments/prod/credentials.yml
  $ ansible-vault encrypt environments/stage/credentials.yml
  ```
* Файл можно редактировать с помощью команды `ansible-vault edit <file>` и расшифровать `ansible-vault decrypt <file>`


### Задание со *
* Для наладки работы приложения с учетом измениний в прощлом дз необходимо было только скопировать `inventory.gcp.yml` в дерриктории `enviroments/stage` и `enviroments/prod` и изменить путь инвентори по умолчанию.


### Задание с **
* С помощью [документации](https://medium.com/@Nklya/%D0%BB%D0%BE%D0%BA%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B2-travisci-2b5ef9adb16e) натсроена утилита `trytravis`. С ее помощью можно проводить тесты и отладку трависа в другом репозитории (kazhem/otus-trytravis).
* Что делает [travis](.travis.yml):
  * Проверка ДЗ (было)
  * `packer validate` для всех шаблонов
  * `terraform validat`e и `tflint` для окружений stage и prod
  * `ansible-lint` для плейбуков Ansible

* В README.md добавлен бейдж с статусом билда


## Homework11 Разработка и тестирование Ansible ролей и плейбуков

* Установлен [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* Установлен [Vagrant](https://www.vagrantup.com/downloads.html)
* Добавлен [Vagrantfile](ansible/Vagrantfile) с описанием terraform-инфраструктуры в vagrant
* Выполнен `vagrant up` для скачивания и запуска образов в соответсвии с файлом
* Краткий список команд
  * Проверка что бокс скачан `vagrant box list`
  * Посмотреть список запущенных ВМ `vagrant status`
  * Подключиться к ВП по ssh `vagrant ssh VMNAME`
  * Запустить провиженер `vagrant provision [VMNAME]`

### Доработка ролей
* [Провиженинг](https://www.vagrantup.com/docs/provisioning/) в Vagrant
* Добавлен провиженер в [Vagrantfile](ansible/Vagrantfile) ВМ `db`
  ```ruby
  db.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/site.yml"
    ansible.groups = {
    "db" => ["dbserver"],
    "db:vars" => {"mongo_bind_ip" => "0.0.0.0"}
    }
  end
  ```
* Запущен провиженер `vagrant provision dbserver`
* Возникла ошибка:
  ```
  fatal: [dbserver]: FAILED! => {"changed": false, "msg": "Could not find the requested service mongod: host"}
  ```
  Причина -- роль [db](ansible/roles/db) писалась с учётом базового packer-образа с уже установленной MongoDB
* В роль [db](ansible/roles/db) добавлены задачи по установке MongoDB
* Провиженинг завершён успешно
* Добавлен плейбук [base.yml](ansible/playbooks/base.yml) для установки python если он не установлен.

* Задачи роли [db](ansible/roles/db) из [db/tasks/main.yml](ansible/roles/db/tasks/main.yml) разнесены по файлам
  * Задачи по установке MongoDB вынесены в [install_mongodb.yml](ansible/roles/db/tasks/install_mongodb.yml)
  * Задачи по настройке MongoDB вынесены в [config_mongodb.yml](ansible/roles/db/tasks/config_mongodb.yml)
* Для проверки, повторно выполнен провиженинг. Прошёл успешно

* Задачи роли [app](ansible/roles/app) из [main.yml](ansible/roles/app/tasks/main.yml) разнесены по файлам
  * Задачи по установке MongoDB вынесены в [ruby.yml](ansible/roles/app/tasks/ruby.yml)
  * Задачи по настройке MongoDB вынесены в [puma.yml](ansible/roles/app/tasks/puma.yml)
* Добавлен провиженер в [Vagrantfile](ansible/Vagrantfile) ВМ `app`
  ```ruby
  db.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/site.yml"
    ansible.groups = {
    "db" => ["appserver"],
    "app:vars" => { "db_host" => "10.10.10.10"}
    }
  end
  ```
* Запущен провиженер `vagrant provision appserver`. Возникла ошибка:
  ```
  TASK [app : Add config for DB connection] **************************************
  fatal: [appserver]: FAILED! => {"changed": false, "checksum": "dfbe4b5cf3ec32d91d20045e2ee7f7b26c60ef34", "msg": "Destination directory /home/appuser does not exist"}
  ```
* Для исправления ошибки, параметризована роль [app](ansible/roles/app)
  * В переменные по умолчанию [app/defaults/main.yml](ansible/roles/app/defaults/main.yml) добавлен `deploy_user: appuser`
  * `puma.service` из `files` перемещён в `templates`
  * В шаблоне [puma.service.j2](ansible/roles/app/templates/puma.service.j2) все упоминания пользователя `appuser` заменены на переменную `{{ deploy_user }}`
  * В [puma.yml](ansible/roles/app/tasks/puma.yml) все упоминания пользователя `appuser` заменены на переменную `{{ deploy_user }}`
* Также `appuser` заменён на `{{ deploy_user }}` в [deploy.yml](ansible/playbooks/deploy.yml)
* В [Vagrantfile](ansible/Vagrantfile) добавлено переопределение переменных для провиженера
  ```ruby
  config.vm.define "appserver" do |app|
    ...

    app.vm.provision "ansible" do |ansible|
      ...
      ansible.extra_vars = {
        "deploy_user" => "vagrant"
      }
    end
  end
  ```
* Запущен провиженер `vagrant provision appserver`. Прошёл успешно

* Проверена работа приложения [http://10.10.10.20:9292/](http://10.10.10.20:9292/). Успешно.
* Конфигурация пересоздана, перепроверена и удалена
  ```shell
  vagrant destroy -f
  vagrant up
  #open in browser http://10.10.10.20:9292/
  vagrant destroy -f
  ```
### Задание со ☆
* Настройки nginx перенесены из `enviroments/stage/group_vars/app` в [Vagrantfile](ansible/Vagrantfile)
  ```ruby
    ansible.extra_vars = {
      "deploy_user" => "vagrant",
      "nginx_sites" => {
        "default" => [
          "listen 80",
          "server_name \"reddit\"",
          "location / {
            proxy_pass http://127.0.0.1:9292;
          }"
        ]
      }
    }
    ```
* Результат проверен. Сайт открывается по адресу [http://10.10.10.20](http://10.10.10.20)
