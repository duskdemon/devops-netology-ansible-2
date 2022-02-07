#devops-netology
### 08-ansible-04-roles
# Домашнее задание к занятию "8.4 Работа с Roles"

## Подготовка к выполнению
1. Создайте два пустых публичных репозитория в любом своём проекте: kibana-role и filebeat-role.
2. Добавьте публичную часть своего ключа к своему профилю в github.

## Основная часть

Наша основная цель - разбить наш playbook на отдельные roles. Задача: сделать roles для elastic, kibana, filebeat и написать playbook для использования этих ролей. Ожидаемый результат: существуют два ваших репозитория с roles и один репозиторий с playbook.

1. Создать в старой версии playbook файл `requirements.yml` и заполнить его следующим содержимым:
   ```yaml
   ---
     - src: git@github.com:netology-code/mnt-homeworks-ansible.git
       scm: git
       version: "2.0.0"
       name: elastic 
   ```
2. При помощи `ansible-galaxy` скачать себе эту роль.
3. Создать новый каталог с ролью при помощи `ansible-galaxy role init kibana-role`.
4. На основе tasks из старого playbook заполните новую role. Разнесите переменные между `vars` и `default`. 
5. Перенести нужные шаблоны конфигов в `templates`.
6. Создать новый каталог с ролью при помощи `ansible-galaxy role init filebeat-role`.
7. На основе tasks из старого playbook заполните новую role. Разнесите переменные между `vars` и `default`. 
8. Перенести нужные шаблоны конфигов в `templates`.
9. Описать в `README.md` обе роли и их параметры.
10. Выложите все roles в репозитории. Проставьте тэги, используя семантическую нумерацию.
11. Добавьте roles в `requirements.yml` в playbook.
12. Переработайте playbook на использование roles.
13. Выложите playbook в репозиторий.
14. В ответ приведите ссылки на оба репозитория с roles и одну ссылку на репозиторий с playbook.
---

**Выполнение:**

были созданы репозитории с ролями для кибана и файлбит соответственно:  
* https://github.com/duskdemon/kibana-role
* https://github.com/duskdemon/filebeat-role  
После чего они были скачаны при помощи ansible-galaxy на машину с Ansible (WSL) через requirements.yml:
```
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ ansible-galaxy install -r requirements.yml -p roles
- extracting elastic to /home/dusk/devops-netology-ansible-2/roles/elastic
- elastic (2.0.0) was installed successfully
- extracting kibana to /home/dusk/devops-netology-ansible-2/roles/kibana
- kibana (1.0.1) was installed successfully
- extracting filebeat to /home/dusk/devops-netology-ansible-2/roles/filebeat
- filebeat (1.0.1) was installed successfully
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ 
```
после некоторой отладки, когда роли заработали и плейбук выдал ОК, версии ролей стали 1.0.3 1.0.4 для kibana и filebeat соответственно.  Ниже результат выполнения плейбука.  
```
dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$ ansible-playbook -i inventory/hosts.yml site.yml --diff

PLAY [Assert elastic] ***************************************************************************

TASK [Gathering Facts] **************************************************************************
ok: [el-inst84]

TASK [elastic : Fail if unsupported system detected] ********************************************
skipping: [el-inst84]

TASK [elastic : include_tasks] ******************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/elastic/tasks/download_yum.yml for el-inst84

TASK [elastic : Download Elasticsearch's rpm] ***************************************************
ok: [el-inst84 -> localhost]

TASK [elastic : Copy Elasticsearch to managed node] *********************************************
ok: [el-inst84]

TASK [elastic : include_tasks] ******************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/elastic/tasks/install_yum.yml for el-inst84

TASK [elastic : Install Elasticsearch] **********************************************************
ok: [el-inst84]

TASK [elastic : Configure Elasticsearch] ********************************************************
ok: [el-inst84]

PLAY [Assert kibana] ****************************************************************************

TASK [Gathering Facts] **************************************************************************
ok: [ki-inst84]

TASK [kibana : Fail if unsupported system detected] *********************************************
skipping: [ki-inst84]

TASK [kibana : include_tasks] *******************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/kibana/tasks/download_yum.yml for ki-inst84

TASK [kibana : Download kibana's rpm] ***********************************************************
ok: [ki-inst84 -> localhost]

TASK [kibana : Copy kibana to managed node] *****************************************************
ok: [ki-inst84]

TASK [kibana : include_tasks] *******************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/kibana/tasks/install_yum.yml for ki-inst84

TASK [kibana : Install kibana] ******************************************************************
ok: [ki-inst84]

TASK [kibana : Configure Kibana] ****************************************************************
ok: [ki-inst84]

PLAY [Assert filebeat] **************************************************************************

TASK [Gathering Facts] **************************************************************************
ok: [fi-inst84]

TASK [filebeat : Fail if unsupported system detected] *******************************************
skipping: [fi-inst84]

TASK [filebeat : include_tasks] *****************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/filebeat/tasks/download_yum.yml for fi-inst84

TASK [filebeat : Download filebeat's rpm] *******************************************************
ok: [fi-inst84 -> localhost]

TASK [filebeat : Copy filebeat to managed node] *************************************************
ok: [fi-inst84]

TASK [filebeat : include_tasks] *****************************************************************
included: /home/dusk/devops-netology-ansible-2/roles/filebeat/tasks/install_yum.yml for fi-inst84

TASK [filebeat : Install filebeat] **************************************************************
ok: [fi-inst84]

TASK [filebeat : Configure filebeat] ************************************************************
ok: [fi-inst84]

PLAY RECAP **************************************************************************************
el-inst84                  : ok=7    changed=0    unreachable=0    failed=0   
fi-inst84                  : ok=7    changed=0    unreachable=0    failed=0   
ki-inst84                  : ok=7    changed=0    unreachable=0    failed=0   

dusk@DESKTOP-DQUFL9J:~/devops-netology-ansible-2$
```
