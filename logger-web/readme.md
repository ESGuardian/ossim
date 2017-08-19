## Что это?
Это способ добавить к AlienVault OSSIM внешний логгер бесплатно.

## Как это сделано?
Используется внешняя БД MongoDB. Все события, которые собирает ossim-agent записываются в базу ossim в коллекции `logger.<data>`, например `logger.20160607`. Для этого внесены незначительные изменения в файлы `Agent.py, Event.py и Output.py` в каталоге `/usr/share/alienvault/ossim-agent/`. В файле `/etc/ossim/agent/config.cfg` добавилась секция `[output-esguard]` в которой можно активировать запись в MongoDB и указать параметры соединения с ней.

## Как использовать?
Прежде всего, необходимо установить и настроить MongoDB на кокой-либо машине, желательно с дисками не менее 1ТБ. Ориентировочный размер базы можно оценить по формуле `1,4К х N x Days`, где N - количество записей в сутки, Days - количество дней хранения логов. 

Следует создать в MongoDB пользователя с административными правами а также создать базу данных `ossim` и в ней пользователей agent и reader с правани чтения/записи и только чтения, соответственно.

Теперь можно разрешить bind MongoDB к внешним интерфейсам сервера, а не только к localhost и не забыть включить обязательную аутентификацию.

Как выполнить все вышеперечисленные действия, смотрите в документации на MongoDB. Это не сложно.

После настройки сервера MongoDB необходимо скопировать из репозитария файлы `Agent.py, Event.py и Output.py` из каталога `/usr/share/alienvault/ossim-agent/` в одноименные каталоги на сервере AlienVault OSSIM. В файл `/etc/ossim/agent/config.cfg` вписать секцию `[output-esguard]` (образец в соответствующем файле в репозитарии) и установить `enable=True`. Перезапустить агент командой `/etc/init.d/ossim-agent restart`. Запись пошла. 

Для просмотра логов можно использовать любой GUI tool для MongoDB или воспользоваться моим браузером логов, который представляет собой файл PHP, работающий на каком-нибудь веб-сервере. Из опыта скажу, что этот вариант удобнее.

## Установка браузера логов
Нужен любой веб-сервер с поддержкой PHP. Здесь описана установка на Debian 8 (jessie) с его стандартным appache2. 

	apt-get install make php5-dev php-pear  
	pecl install mongo

Теперь вставляем в  файл `/etc/php5/apache2/php.ini`, расширение `mongo.so` можно в самом начале файла:

	[PHP]
	extension=mongo.so
	...
Всё. Подготовка завершена. Копируем файл /ossim/logger-web/reader.php в document-root веб-сервера `/var/www/html/`. Редактируем в файле секцию:

	// open connection to MongoDB server
    $conn = new Mongo('mongodb://172.16.0.17', array(
        'username' => 'reader',
        'password' => 'pass',
        'db'       => 'ossim'
    ));

Записываем сюда правильные данные соединения.
## Использование браузера логов

Заходим на веб страницу `http://yourserver/reader.php`. Если всё нормально вы увидите страницу с надписью:

	Записей в журнале: 1,919,681 
	Готов к работе 

Над которой будет примитивная форма запроса. Разумеется число может быть другим. Это количество документов в коллекции `logger.<data>`.  
В форме можно задать поиск по нескольким параметрам: плагин, сигнатура, имя пользователя, IP источника, IP назначения. А также текстовый или regex поиск по сырому логу (payload).  


Я выбрал такую форму запросов исходя из практического опыта. Позже я добавлю форму произвольного запроса в формате JSON для продвинутых пользователей и возможность сохранять запросы в "закладках".

## История версий

* v.0.0.2 С учетом опыта использования улучшена форма для запросов. 
* v.0.0.1 Начало.
