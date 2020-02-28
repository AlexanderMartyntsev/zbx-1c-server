Предполагаем, что zabbix сервер и агент сервера уже установлены и сконфигурированы.

####Адрес на исходник https://github.com/nikimaxim/zbx-1c-server

## Установка

### Необходимо создать службу "Сервера администрирования кластера серверов" RAS.

Воспользуемся скриптом **scCreator.bat** для создания службы:
```
@echo off
set Version="8.3.16.1030"
set SrvUserName=.\USR1CV8
set SrvUserPwd="password"
set CtrlPort=1540
set AgentName=localhost
set RASPort=1545
set SrvcName="1C:Enterprise 8.3 RAS"
set BinPath="\"C:\Program Files\1cv8\%Version%\bin\ras.exe\" cluster --service --port=%RASPort% %AgentName%:%CtrlPort%"
set Desctiption="1C:Enterprise 8.3 RAS"
sc stop %SrvcName%
sc delete %SrvcName%
sc create %SrvcName% binPath= %BinPath% start= auto obj= %SrvUserName% password= %SrvUserPwd% displayname= %Desctiption%
```
### Запускаем созданную службу. Можно поставить запуск от системной учётной записи, чтобы не париться.

### Кладём скрипт **zbx_1c_server.ps1** в папку "C:\service"

### Модифицируем кофиг файл агента zabbix **zabbix_agentd.conf**

Добавляем параметры для сбора данных с сервера администрирования 1С:
```
UserParameter=1c.discovery[*],powershell -NoProfile -NoLogo -ExecutionPolicy Bypass -File "C:\service\zbx_1c_server.ps1" $1 $2
UserParameter=1c.session[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list --infobase="$2" | find /c "1CV8"
UserParameter=1c.session-all[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list | find /c "1CV8"
UserParameter=1c.session-all-hibernate[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list | find /c ": yes"
UserParameter=1c.bgj[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list --infobase="$2" | find /c "BackgroundJob"
UserParameter=1c.web-session[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list --infobase="$2" | find /c "WebClient"
UserParameter=1c.designer-session[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 session --cluster="$1" list --infobase="$2" | find /c "Designer"
UserParameter=1c.working-process[*],"C:\Program Files\1cv8\8.3.16.1030\bin\rac.exe" localhost:1545 process --cluster="$1" list | find /c "process"
```
Если сервер 1С находится не по адресу **localhost:1545**, то правим адрес, иначе, можно адрес не указывать.</br>
Правим так же путь к актуальной версии платформы.

5. Перезапускаем zabbix агент.

6. Для получения данных, необходимо к узлу сети в панель управления zabbix добавить шаблон **zbx_export_templates.xml**

<br/>

#### Examples images:
- Graph: 1C working process.
![Image alt](https://github.com/nikimaxim/zbx-1c-server/blob/master/img/1.png)
- Graph: 1C session for all DB
![Image alt](https://github.com/nikimaxim/zbx-1c-server/blob/master/img/2.png)
- Graph: 1C info one DB
![Image alt](https://github.com/nikimaxim/zbx-1c-server/blob/master/img/3.png)

<br/>

- Discovery rules

<br/>

![Image alt](https://github.com/nikimaxim/zbx-1c-server/blob/master/img/4.png)

<br/>

- Items prototypes

<br/>

![Image alt](https://github.com/nikimaxim/zbx-1c-server/blob/master/img/5.png)

<br/>

#### License
- GPL v3
