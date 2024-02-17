# Настраиваем центральный сервер для сбора логов

### Описание задания
1. В Vagrant разворачиваем 2 виртуальные машины web и log  
2. на web настраиваем nginx  
3. на log настраиваем центральный лог сервер на системе rsyslog
4. настраиваем аудит, следящий за изменением конфигов nginx 

Все критичные логи с web должны собираться и локально и удаленно.
Все логи с nginx должны уходить на удаленный сервер (локально только критичные).
Логи аудита должны также уходить на удаленную систему.

### Этапы выполнения

1. Создаём виртуальные машины
Создаём каталог, в котором будут храниться настройки виртуальной машины. В каталоге создаём файл с именем Vagrantfile, добавляем в него следующее содержимое:
<image src="/screens/vagrantfile.jpg" width="400" alt="vagrantfile">

Для правильной работы c логами, нужно, чтобы на всех хостах было настроено одинаковое время. 
Укажем часовой пояс (Московское время): ```cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime```
и перезупустим службу NTP Chrony: ``` systemctl restart chronyd ```
Далее проверим, что время и дата указаны правильно: ``` date ```
(Настроить NTP нужно на обоих серверах!)  

2. Установка nginx на виртуальной машине web
- для установки nginx сначала нужно установить epel-release: ``` yum install epel-release ``` 
- установим nginx: ``` yum install -y nginx ```  
- запускаем nginx: ``` systemctl start nginx  ```

Проверим, что nginx работает корректно:
``` systemctl status nginx ```  
<image src="/screens/status_nginx.jpg" width="400" alt="status_nginx" >

3. Настройка центрального сервера логов

Для приема логов внес следующие изменения в фал /etc/rsyslog.conf:
- раскомментировал строки  
```$ModLoad imudp```  
```$UDPServerRun 514```  
```$ModLoad imtcp```  
```$InputTCPServerRun 514```  
  
- добавил в конец файла строки  
```$template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"```  
```*.* ?RemoteLogs```  
```& ~'```  

- с помощью ansble заменил файл на сервере сбора логов (log-server).

Выполнил настройку отправли логов web-server. Для чего изменл местодля хранения логов с локального хранилища на сервер логов в файле /etc/nginx/nginx.conf  
- ```error_log syslog:server=192.168.50.15:514,tag=nginx_error;```  
- ```access_log  syslog:server=192.168.50.15:514,tag=nginx_access,severity=info combined;```  

4. Настройка аудита, контролирующего изменения конфигурации nginx

Для настройки аудита изменения конфигурации nginx внес изменения в файлы конфигурации на web-server:  
- в файле /etc/audit/rules.d/audit.rules, добавил строки  
```-w /etc/nginx/nginx.conf -p wa -k nginx_conf```  
```-w /etc/nginx/default.d -p wa -k nginx_conf```  
Таким образом был включен механизм записи локальной записи логов аудита  
- после чего настроил пересылку логов на сервер сбора логов, изменив строки в файле /etc/audit/auditd.conf  
```name_format = HOSTNAME```  
- выполнил установку audispd-plugins;  
- в файле /etc/audisp/plugins.d/au-remote.conf поменяем параметр active на yes;  
- /etc/audisp/audisp-remote.conf указа адрес сервера сбора логов;  
```remote_server = 192.168.50.15```  
- на сервере сбора логов открыл порт 60 для чего в файле /etc/audit/auditd.conf раскоменировал строку 
```tcp_listen_port = 60```  
  
Для автомотизации сформеровал файл playbook для ansible  
  
### Ansible
Для конфигурирования ролей серверов с помощью Ansible необходимо сгенерировать ssh-ключи используя команду ``` ssh-keygen ```
при этом имя ключей по умолчанию будет id_rsa, если имя будет задано другим, необходимо указать новое имя в Vagranfile в строке ``` config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/me.pub" ```
запуск автоматической конфигкрации осуществляется с помощью команды ``` ansible-playbook ./ansible/playbook.yml -i ./ansible/inventory```






