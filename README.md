## Start LEMP server with Ansible

Данный playbook для Ansible устанавливает на удаленный сервер Nginx, PHP и SSL сертификат Let's Encrypt.

**Требования для запуска:**
1. Локальная или удаленная машина на Ubuntu, с которой будет производиться настройка удаленного сервера.
2. Установленный на машине, с которой идет настройка, Ansible и несколько пакетов из Ansible Galaxy для Nginx, PHP, Cerbot.
3. Удаленный сервер на Ubuntu, который будет настраиваться.
4. Сгенерированные SSH ключи для пользователя root и www.
5. Доступ к DNS панели и собственный домен, для настройки SSL сертификата.

## Инструкция

### 1. Удаленный сервер
Арендовать любой сервер на Ubuntu (достаточно 10 Гб диск, 1Гб оперативной памяти, и 1-ядерный процессор 2Ггц).

### 2. Домен
Купить домен (если необходим), для настройки SSL сертификата. Также необходимо иметь доступ к изменению DNS записей 
(обычно есть в веб-панели хостера), чтобы прописать ip своего сервера.

### 3. Локальная машина
Можно установить Ansible на локальную машину с Ubuntu или другим дистрибутивом, который поддерживает Ansible. 
Также можно все действия проводить в Ubuntu из WSL2 для Windows 10.

### 4. SSH-ключи

1. Открываем терминал, переходим в папку с SSH ключами:

`cd ~/.ssh/`

2. Смотрим, что есть в папке, чтобы не перезаписать уже существующие ключи:

`ls -la`

3. Генерируем ключ для пользователя **root** с удаленного сервера:

`ssh-keygen`

4. Далее нас спрашивают, какое имя файла ключа указать, по умолчанию ставится всегда id_rsa, 
напишем свое собственное имя, так как ключей может понадобиться больше чем 1. Вы можете указать любое другое имя, 
которого нет в папке с ключами:

`www_root_key`

5. После нескольких вопросов (их можно пропустить, либо можете задать свои параметры, если это нужно), будет сгенерировано два ключа:
   `www_root_key` и `www_root_key.pub`, приватный и публичный соответственно.

   
6. Далее нужно добавить соответствие ключ-пользователь-хост, чтобы SSH понимал, какой ключ использовать для подключения к нужному серверу. 
Для этого смотрим, есть ли в папке файл `config`, если есть, открываем его с помощью редактора **nano** (вся информация по нему есть в [сети](https://www.nano-editor.org/dist/latest/nano.html)):

`nano config`

7. Если файл уже был, вы можете увидеть там записи подобного плана 
(все параметры и их назначение можно найти в [сети](https://www.opennet.ru/man.shtml?topic=ssh_config&category=5), здесь один из вариантов настройки, если у вас стандартный порт подключения по SSH **22** и тд.):

`Host YOUR.OWN.IP.ADDRESS` - указываете ip адрес вашего сервера, куда будете подключаться<br>
`HostName YOUR.OWN.IP.ADDRESS` - тоже самое<br>
`IdentityFile ~/.ssh/www_root_key` - файл приватного ключа для пользователя **root**<br>
`IdentitiesOnly yes` - использование файлов идентификации только из конфига<br>
`User root` - под каким пользователем осуществлять подключение<br>

8. Если файла не было, создаем его и уже потом открываем через **nano**:

`touch config`

9. Создаем такую же конфигурацию для вашего собственного сервера и SSH ключа, просто добавить запись в файл config ниже существующих через строку.


10. Сохраняем файл **config** (`ctrl+O -> Enter -> ctrl+X`).


11. Точно также как для **root** создаем файлы ключей для пользователя **www** и добавляем их в **config**, указывая `User www`.


12. Копируем публичный ключ на сервер для пользователя **root** (для пользователя **www** ключ будет копироваться с помощью Ansible):

`ssh-copy-id -i ~/.ssh/www_root_key root@YOUR.OWN.IP.ADDRESS`

13. Появится запрос пароля для **root**, введите его.

14. Теперь и вы и Ansible сможет подключаться к серверу по SSH ключу. Чтобы проверить работоспособность ключа, выполните команду:

`ssh root@YOUR.OWN.IP.ADDRESS`

Если запроса пароля не было и вы вошли на удаленный сервер под пользователем **root**, то все работает верно.

15. Выйдите с удаленного сервера:

`exit`

### 5. Установка Ansible

Теперь необходимо установить Ansible на локальную машину 
(**важно** для Ubuntu версии старше, чем 20.04, возможно потребуется предварительно добавить репозиторий с Ansible):

`sudo apt install ansible`


### 6. Установка из этого репозитория

1. Клонируйте репозиторий с данным **playbook** в локальную папку **ansible**:

`git clone git@github.com:quoterbox/ansible-start-lemp.git ansible`

2. Перейдите в папку **ansible**, отсюда вы чуть позже будете запускать playbook для настройки сервера:

`cd ansible/`

3. Установить нужные вам пакеты из Ansible Galaxy, например, необходимые для этого playbook:

`ansible-galaxy install geerlingguy.nginx geerlingguy.php geerlingguy.certbot geerlingguy.mysql geerlingguy.php-mysql`

4. Проверьте работоспособность **ansible**:

`ansible --version`

### 6. Настройка playbook

1. Желательно открыть весь код из репозитория в любой IDE (например, PyCharm, PHPStorm и др.).

2. Переименовать файл `hosts.ini.example` в `hosts.ini` и вписать туда ip адрес удаленного сервера.

3. `ansible.cfg` - указать путь к файлу с хостами hosts.ini

4. `playbook.yml` - раскомментировать/закомментировать ненужные таски и роли

5. `group_vars/main.yml` - указать **домен**, пользователя **www** и путь к его ssh ключам (предварительно их нужно добавить как и для root пользователя), пользователя **root** и путь к его ключам

6. `vars/main.yml` - указать нужную версию php, если она отличается от той, что там указана, и прочие настройки для php, nginx и mysql.

7. Проверить удается ли пользователю **ansible** указанному в **group_vars/main** подключиться с ключом к серверу используя `hosts.ini` файл:

`ansible all -m ping`

Ответом должно быть `pong` с дополнительной meta информацией.


### 7. Запуск playbook

Теперь у вас должны быть настроены ключи доступа к серверу, установлен Ansible и несколько необходимых пакетов из Ansible Galaxy.

Запускаем playbook:

`ansible-playbook playbook.yml`

Должны пройти все шаги установок без ошибок, ошибки подсветятся красным.

### 8. Папка сайта

Данным playbook из репозитория будет настроена работа с папкой:

`/var/www/ВАШ_ДОМЕН/src/public`, зависит от того, что указано в **root** директории для **nginx** в файле `/vars/main.yml` в переменных для playbook.

Саму папку домена вы можете создать сами внутри папки `/var/www/`. Для проверки работоспособности сервера, вы можете залить файл index.php с содержимым:

`<?php phpinfo();`

Теперь папка сайта должна быть доступна по адресу `http(s)://ВАШ_ДОМЕН`

### 9. Возможные ошибки

**Пример ошибки:** Если при попытке открыть сайт http(s)://ВАШ_ДОМЕН/index.php появляется ошибка 502 от NGINX:
1) Проверьте, есть ли файлы в директории **root**, которая указана в файле `vars/main.yml` в секции **nginx_vhosts**
   1) Пример: в **nginx_vhosts** указан параметр `root: "/var/www/{{ domain }}/src/public"`, значит на сервере в папке `/var/www/YOUR_DOMAIN_NAME.com/src/public/` должен находиться файл `index.php`
2) Проверьте запущены ли процессы **php-fpm**:
   1) Пример: запустите команду `sudo ps aux | grep 'php'`, она выдаст список пулов доступных для работы **php-fpm**, если ни одного процесса и пула нет, значит php-fpm не настроен или не запущен, вам следует проверить настройки пула в файле `vars/main.yml` в секции **php_fpm_pools**
3) Проверьте пользователя и группу, от которого запускаются **nginx** и **php-fpm**: по умолчанию веб пользователь - **www**, а группа - **www-data**. Эта команда `sudo ps aux | grep 'php'` покажет от какого пользователя и группы запущен процесс **php-fpm**
4) Проверьте права на файлы сайта в вашей **root** директории, например, `/var/www/YOUR_DOMAIN_NAME.com/src/public/`. Права должны быть такие же как у веб пользователя из плейбука, в данном примере это веб пользователь - **www**, а группа - **www-data**.
5) Проверьте, запущен ли у вас NGINX: `systemctl status nginx`  (команда для Ubuntu 20.04 и подобные). Статус должен быть **active**
6) Если у вас возникли проблемы с NGINX и php-fpm, но все вроде настроено правильно, первым делом перезагрузите сервер.

### 10. Основные конфигурационные файлы

Эти файлы настраиваются автоматически. Если у вас возникла ошибка, скорее всего вы можете ее исправить в настройках плейбука, а не самих конфигурационных файлах на сервере.

**NGINX:**
1. `/etc/nginx/nginx.conf`
2. `/etc/nginx/sites-enabled/YOUR_DOMAIN.com.80.conf`
3. `/etc/nginx/sites-enabled/YOUR_DOMAIN.com.conf`

**PHP-FPM:**
1. `/etc/php/YOUR_PHP_VERSION/fpm/php.ini`
2. `/etc/php/YOUR_PHP_VERSION/fpm/php-fpm.conf`
3. `/etc/php/8.1/fpm/pool.d/www.conf`  (Если имя вашего пула, который вы указали в `vars/main.yml` **www**)

### LICENSE
[MIT License](./LICENSE.md)