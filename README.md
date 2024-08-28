# Домашнее задание к занятию 3 «Использование Ansible»

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.

![изображение](https://github.com/stepynin-georgy/hw_ansible_3/blob/main/img/Screenshot_148.png)

2. Репозиторий LightHouse находится [по ссылке](https://github.com/VKCOM/lighthouse).

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.
2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.
4. Подготовьте свой inventory-файл `prod.yml`.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

Осталась одна ошибка, как ее исправить пока нет идей. Но плейбук проходит полностью.

![изображение](https://github.com/stepynin-georgy/hw_ansible_3/blob/main/img/Screenshot_151.png)

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

Плейбук с флагом `--check` не проходит дальше установки пакетов, т.к. они не загружены. 

![изображение](https://github.com/stepynin-georgy/hw_ansible_3/blob/main/img/Screenshot_150.png)

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

```
root@netology:/opt/ansible_hw3# ansible-playbook -i playbook/inventory/prod.yml playbook/site.yml --diff
[WARNING]: Found both group and host with same name: vector
[WARNING]: Found both group and host with same name: clickhouse
[WARNING]: Found both group and host with same name: lighthouse

PLAY [Install Clickhouse] **********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host clickhouse is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [clickhouse]

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************
failed: [clickhouse] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.deb", "elapsed": 0, "gid": 1000, "group": "user", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "user", "response": "HTTP Error 404: Not Found", "size": 246378832, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/deb/pool/main/c/clickhouse-common-static/clickhouse-common-static_22.3.3.44_all.deb"}
ok: [clickhouse] => (item=clickhouse-client)
ok: [clickhouse] => (item=clickhouse-server)

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************
ok: [clickhouse]

TASK [Install clickhouse packages] *************************************************************************************************************************************************************************************
ok: [clickhouse] => (item=clickhouse-common-static)
ok: [clickhouse] => (item=clickhouse-client)
ok: [clickhouse] => (item=clickhouse-server)

TASK [Deploy config clickhouse] ****************************************************************************************************************************************************************************************
--- before: /etc/clickhouse-server/config.xml
+++ after: /root/.ansible/tmp/ansible-local-17448tzktnvyf/tmp8fb8w2l2/clickhouse.config.j2
@@ -161,8 +161,7 @@
          This is required when interserver_https_port is accessible from untrusted networks,
          and also recommended to avoid SSRF attacks from possibly compromised services in your network.
       -->
-    <!--<interserver_http_credentials>
-        <user>interserver</user>
+    <!--<interserver_http_credentials>        <user>interserver</user>
         <password></password>
     </interserver_http_credentials>-->
 
@@ -183,10 +182,9 @@
     <!-- <listen_host>0.0.0.0</listen_host> -->
 
     <!-- Default values - try listen localhost on IPv4 and IPv6. -->
-    <!--
+
     <listen_host>::1</listen_host>
-    <listen_host>127.0.0.1</listen_host>
-    -->
+    <listen_host>0.0.0.0</listen_host>
 
     <!-- Don't exit if IPv6 or IPv4 networks are unavailable while trying to listen. -->
     <!-- <listen_try>0</listen_try> -->
@@ -367,7 +365,7 @@
 
     <!-- Path to temporary data for processing hard queries. -->
     <tmp_path>/var/lib/clickhouse/tmp/</tmp_path>
-    
+
     <!-- Disable AuthType plaintext_password and no_password for ACL. -->
     <!-- <allow_plaintext_password>0</allow_plaintext_password> -->
     <!-- <allow_no_password>0</allow_no_password> -->`
@@ -1294,4 +1292,4 @@
         </tables>
     </rocksdb>
     -->
-</clickhouse>
+</clickhouse>
\ No newline at end of file

changed: [clickhouse]

TASK [Deploy users config clickhouse] **********************************************************************************************************************************************************************************
--- before: /etc/clickhouse-server/users.xml
+++ after: /root/.ansible/tmp/ansible-local-17448tzktnvyf/tmpmbw8lm8g/clickhouse.users.j2
@@ -100,6 +100,10 @@
             <!-- User can create other users and grant rights to them. -->
             <!-- <access_management>1</access_management> -->
         </default>
+           <netology>
+            <password>netology</password>
+            <access_management>1</access_management>
+        </netology>
     </users>
 
     <!-- Quotas. -->
@@ -120,4 +124,4 @@
             </interval>
         </default>
     </quotas>
-</clickhouse>
+</clickhouse>
\ No newline at end of file

changed: [clickhouse]

TASK [Flush handlers] **************************************************************************************************************************************************************************************************

TASK [Wait for clickhouse-server to become available] ******************************************************************************************************************************************************************
Pausing for 15 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse]

TASK [Create database] *************************************************************************************************************************************************************************************************
ok: [clickhouse]

TASK [Create table for logs] *******************************************************************************************************************************************************************************************
changed: [clickhouse]

PLAY [Install Vector] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host vector is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [vector]

TASK [Create vector work directory] ************************************************************************************************************************************************************************************
ok: [vector]

TASK [Get Vector distrib] **********************************************************************************************************************************************************************************************
ok: [vector]

TASK [Unzip Vector archive] ********************************************************************************************************************************************************************************************
ok: [vector]

TASK [Install Vector binary] *******************************************************************************************************************************************************************************************
ok: [vector]

TASK [Check Vector installation] ***************************************************************************************************************************************************************************************
changed: [vector]

TASK [Create Vector config vector.toml] ********************************************************************************************************************************************************************************
ok: [vector]

TASK [Create vector.service daemon] ************************************************************************************************************************************************************************************
changed: [vector]

TASK [Modify vector.service file] **************************************************************************************************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -8,7 +8,7 @@
 User=vector
 Group=vector
 ExecStartPre=/usr/bin/vector validate
-ExecStart=/usr/bin/vector
+ExecStart=/usr/bin/vector --config /etc/vector/vector.toml
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID
 Restart=no

changed: [vector]

TASK [Create user vector] **********************************************************************************************************************************************************************************************
ok: [vector]

TASK [Create Vector data_dir] ******************************************************************************************************************************************************************************************
ok: [vector]

RUNNING HANDLER [Start Vector service] *********************************************************************************************************************************************************************************
changed: [vector]

PLAY [Install Lighthouse] **********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host lighthouse is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [lighthouse]

TASK [Install nginx] ***************************************************************************************************************************************************************************************************
ok: [lighthouse]

TASK [Create Nginx config] *********************************************************************************************************************************************************************************************
ok: [lighthouse]

TASK [Install git] *****************************************************************************************************************************************************************************************************
ok: [lighthouse]

TASK [Copy lighthouse rfom git] ****************************************************************************************************************************************************************************************
ok: [lighthouse]

TASK [Create lighthouse config] ****************************************************************************************************************************************************************************************
ok: [lighthouse]

PLAY RECAP *************************************************************************************************************************************************************************************************************
clickhouse                 : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
lighthouse                 : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector                     : ok=12   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Для проверки Lighthouse перешел по адресу `http://89.169.174.116/#http://89.169.172.66:8123/?user=netology` и авторизовался:

![изображение](https://github.com/stepynin-georgy/hw_ansible_3/blob/main/img/Screenshot_152.png)

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---





