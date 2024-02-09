# Настраиваем центральный сервер для сбора логов

### Описание задания
1. В Vagrant разворачиваем 2 виртуальные машины web и log  
2. на web настраиваем nginx  
3. на log настраиваем центральный лог сервер на любой системе на выбор:  
- journald;
- rsyslog;
- elk.
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






