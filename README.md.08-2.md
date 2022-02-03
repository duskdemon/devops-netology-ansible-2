#devops-netology
### 08-ansible-02-playbook
 Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
В VirtualBox поднял два хоста на CentOS7б настроил идентификацию по сертификату:
__ansible.cfg__
```
[defaults]    
private_key_file = /home/dusk/devops-netology-ansible-2/abk.pem
```
Содержимое __Inventory/prod.yml__:  
```
---
  elas:
    hosts:
      elas01:
        ansible_host: 192.168.88.16
        ansible_connection: ssh
        ansible_user: root
  kiba:
    hosts:
      kiba01:
        ansible_host: 192.168.88.15
        ansible_connection: ssh
        ansible_user: root
```

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.  
**Выполнение:**  
Добавил фаил переменных для Кибаны - /kiba/vars.yml

```
---
kibana_version: "7.16.1"
kibana_home: "/opt/kibana/{{ kibana_version }}"
```

Добавил файл в Temlpate kbn.sh.j2

```
# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
#!/usr/bin/env bash

export ES_HOME={{ kibana_home }}
export PATH=$PATH:$ES_HOME/bin
```

Добавил таск, блок в site.yml
```
- name: Install Kibana
  hosts: kiba
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template:
        src: templates/kib.sh.j2
        dest: /etc/profile.d/kib.sh
      tags: kibana
```
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.  
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.  
**Выполнение:**  
Проверка линт прошла без ошибок, т.к. до этого много раз правил файл, запускал с "--check" и без, и забыл про линтер. В итоге все ОК:  
```
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] *****************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [elas01]
ok: [kiba01]

TASK [Set facts for Java 11 vars] ***************************************************************************************************
ok: [elas01]
ok: [kiba01]

TASK [Upload .tar.gz file containing binaries from local storage] *******************************************************************
ok: [elas01]
ok: [kiba01]

TASK [Ensure installation dir exists] ***********************************************************************************************
ok: [elas01]
ok: [kiba01]

TASK [Extract java in the installation directory] ***********************************************************************************
skipping: [elas01]
skipping: [kiba01]

TASK [Export environment variables] *************************************************************************************************
ok: [elas01]
ok: [kiba01]

PLAY [Install Elasticsearch] ********************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [elas01]

TASK [Upload tar.gz Elasticsearch from remote URL] **********************************************************************************
ok: [elas01]

TASK [Create directrory for Elasticsearch] ******************************************************************************************
ok: [elas01]

TASK [Extract Elasticsearch in the installation directory] **************************************************************************
skipping: [elas01]

TASK [Set environment Elastic] ******************************************************************************************************
ok: [elas01]

PLAY [Install Kibana] ***************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [kiba01]

TASK [Upload tar.gz Kibana from remote URL] *****************************************************************************************
ok: [kiba01]

TASK [Create directrory for Kibana] *************************************************************************************************
ok: [kiba01]

TASK [Extract Kibana in the installation directory] *********************************************************************************
skipping: [kiba01]

TASK [Set environment Kibana] *******************************************************************************************************
ok: [kiba01]

PLAY RECAP **************************************************************************************************************************
elas01                     : ok=9    changed=0    unreachable=0    failed=0   
kiba01                     : ok=9    changed=0    unreachable=0    failed=0   

```


9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.  
```
#### Переменные group_vars
*java_oracle_jdk_package - имя пакета Java  
*java_jdk_version - версия Java  
*elastic_home - переменная домашнего каталога Elasticsearch  
*kibana_home - переменная домашнего каталога Kibana  
*elastic_version - версия Elasticsearch  
*kibana_version - версия Kibana  

#### Описание Play 

##### Install Java
 установлены тэги "java"
 * установка переменных (facts)
 * загрузка пакета
 * создние рабочего каталога
 * распаковка пакета
 * создание по шаблону переменных окружений (templates)
##### Install Elastic
 установлены тэги "elastic"
 * загрузка установочного пакета 
 * создание рабочего каталога
 * распаковка в рабочий каталог из пакета
 * создание по шаблону переменных окружений (templates)

##### install Kibana
 установлены тэги "kibana"
 * загрузка установочного пакета 
 * создание рабочего каталога
 * распаковка в рабочий каталог из пакета
 * создание по шаблону переменных окружений (templates)
```
### Доработка

1. Просьба приложить к описанию результат вывода ansible-lint и исправить его ошибки и предупреждения  

Для проверки работы линтера, внес лишний символ в site.yml, проверил, скорректировал ошибку:  

```
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ sudo nano site.yml
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ ansible-lint site.yml -v
Syntax Error while loading YAML.
  found character that cannot start any token

The error appears to have been in '/home/dusk/devops-netology-ansible-2/site.yml': line 2, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

---
- `name: Install Java
  ^ here

dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ sudo nano site.yml
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ ansible-lint site.yml -v
Examining site.yml of type playbook
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$
```
2. Обязательно удалите из под версионного контроля java. Во-первых это нарушение лиценционного соглашения. Во-вторых дистрибутивы не хранят под версионным контролем, он для исходного кода  
Удалил папку files из репозитория. На самом деле, самого дистрибутива там, конечно, не было. 
