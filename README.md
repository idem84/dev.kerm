# BDBACKUP.SH
##Shell Script для автоматического создания бэкапов баз данных MYSQL.

Сам скрипт и все ниже описанное подразумевает собой что Вы будете делать резервное копирование баз данных MYSQL из под **root** пользователя.

Для работы синхронизации с удаленным FTP, Вам понадобиться установка программы **lftp** на сервер: [http://lftp.yar.ru/get.html](http://lftp.yar.ru/get.html)
Синхронизация необходима чтобы созданные резервные файлы баз данных хранились не только локально, но еще и удаленно, в случае например если полетит/сгорит Ваш сервер в месте с бэкапами ваших баз данных.

###Шаг первый 1: Установите программу lftp:

Например если у Вас CentOS:

```html
yum install lftp
```

###Шаг второй 2: Создайте необходимые папки в нужных директориях:
Наши файлы будут храниться внутри папки **/tmp**.
Тут есть один нюанс, на некоторых OS или при определенных настройках, все папки и файлы внутри папки **/tmp** могут удалиться на сервере, например при перезагрузке сервера. Если у Вас так, то создайте все эти папки внутрии папки **/root** и измените пути в скрипте.

```html
mkdir /tmp/backups
mkdir /tmp/backups/mysql
```
###Шаг третий 3: Поместите файл bdbackup.sh в /root директорию
Сделайте на файл нужные права для выполнения скрипта:

```html
chmod +x /root/bdbackup.sh
```

###Шаг четвертый 4: В консоли вводим команду:

```html
crontab -e
```
Если вылезла ошибка что сервис не найден, Вам нужно установить cron на сервере, например краткая инструкция установки для **CentOS**:

```html
yum install crontabs -y
chkconfig crond on
touch /var/spool/cron/root
service crond start
crontab -e
```

Далее переходим в режим редактирования **i** и указываем в файле почту на которую будет присылаться информация о ходе выполнения бэкапа баз данных на сайте, время и переодичность запуска скрипта:

```html
MAILTO="alert@site.ru"
0 */4 * * * /root/bdbackup.sh
```

В данном случае указано запускать скрипт каждые 4 часа.

Выходим из редактирования **Esc**, вводим команду **:wq** и жмем **Enter**.

Все готово.

###Напишу еще немного по поводу работы с lftp:
За синхронизацию резервных файлов отвечает самая последняя строчка в скрипте

```html
lftp -e 'mirror --only-newer --reverse --no-empty-dirs --log /var/log/mirror.log --parallel=10 /tmp/backups/mysql /site/mysql; bye;' -u user,password ip address
```

* `--only-newer` - Скачивать только новые/обновленные файлы
* `--reverse` - Параметр определяет что нужно заливать файлы на удаленные сервер, а не скачивать с удаленного сервера.
* `--no-empty-dirs` - Параметр определяет что не нужно заливать пустые папки на удаленный сервер.
* `--parallel` - Параметр позволяет загружать параллельно до 10 файлов.
* `--log` - путь к файлу в котором будут храниться логи работы программы
* `user,password ip address` - логин, пароль и ip адрес удаленного FTP сервера. Пример: (user,password 213.183.204.3)
