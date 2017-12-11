# api-json-db — json DB с открытым кодом

«API json DB» — система управления базами данных с открытым исходным кодом которая использует JSON документы и схему базы данных. Написана на PHP. Подключается через Composer как обычный пакет PHP, после подключения сама настраивается за несколько секунд. Имеет свой RESTful API интерфейс, что позволяет использовать ее с любым другим языком программирования. Умеет ставить в очередь на запись при блокировке файлов другими процессами и кэшировать любые запросы.

«API json DB» — это менеджер json файлов с возможностью кеширования популярных запросов, проверкой валидности файлов и очередью на запись при блокировке таблиц (файлов db) на запись другими процессами.

Главная задача «API json DB» обеспечить быструю отдачу контента (кэширование) и бесперебойную работу сайта при сбоях или недоступности API от которой она получает данные.

## Использование 

— Хранилище для кеша если `"cached": "true"`

— Хранилище для логов или других данных с разбивкой на файлы с лимитом записей или размером файла.

— Основная база данных с таблицами до 1 млн. записей. `"api": "false"`

— Автоматически подключение когда API недоступен. Сайт продолжает работать и пишет все `POST`, `PUT`, `DELETE` запросы в request для последующей синхронизации с API. Когда API снова станет доступный база сначала синхронизирует все данные пока не обработает и не удалит все файлы из папки request и после этого переключится на API

## Возможности

— `api-json-db` это менеджер json файлов с возможностью кеширования популярных запросов и очередью на запись при блокировки файлов на запись другим процессом и проверкой валидности файлов.

— Разбивает таблицы на несколько файлов при достижении указанного в настройках лимита с учетом релевантности записей. Все часто запрашиваемые записи попадут в основной файл. Первый файл будет содержать в два раза меньше данных чем следующий, таким образом самые популярные данные будут находится в файле меньшего размера чем менее популярные, что обеспечит скорость работы.

— Очередь на запись через создание временных файлов в папке temp которые будут ожидать свою очередь на запись. Скрипт будет искать в папке temp файлы, если нашел брать все файлы и открывать их и решать что с ними делать.

— Кеширование - построено по простому принципу - создается файл с точным дубликатом результата по каждому запросу с указанием времени жизни. При запросе проверяется наличие кеша если нет обращаемся к базе или API

— Шифрование данных - возможно все таблицы нужно кодировать а возможно только выбранные

— При запросе можно выбрать тип данных которые отдаст база db (без кеширования) или cached (из кэша), а также будет ли с помощью пользователя обрабатывать очередь на запись в temp и какое количество.

## Старт
Подключить пакет с помощью Composer

```json
"require": {
	"pllano/api-json-db": "*"
}
```

Все у вас уже есть json база данных и вы можете уже создавать таблицы и работать с db

## RESTful API роутинг для cURL запросов

Если вы хотите использовать `RESTful API роутинг` выполните следующие действия:

— В файле `api.php` указать директорию где будет находится база желательно ниже корневой директории вашего сайта. (Пример: ваш сайт находится в папке /www/example.com/public/ разместите базу в `/www/_db_/` таким образом к ней будет доступ только у скриптов). 

— Перенесите файлы `api.php` и `.htaccess` в директорию к которой будет доступ через url (Пример: https://example.com/_12345_/api.php название директории должно быть максимально сложным)

— Запустите `https://example.com/_12345_/api.php` если все хорошо вы увидите

```json
{
    "status": "OK",
    "code": 200,
    "message": "db_json_api works!"
}
```
 
«API json DB» — Движок базы написан на PHP и имеет свой RESTfull API роутинг для CURL запросов, что позволяет использовать ее с любым другим языком программирования. Использует стандарт [APIS-2018](https://github.com/pllano/APIS-2018/) — это формат обмена данными сервер-сервер и клиент-сервер. `Стандарт APIS-2018 - не является общепринятым !` Стандарт является взглядом в будущее и рекомендацией для унификации построения легких движков интернет-магазинов нового поколения.

### Безопасность

При запуске база автоматически создаст файл с ключом доступа который находится в директории: `/www/_db_/core/key_db.txt`

Активация доступа по ключу устанавливается в файле `api.php`

```php
$config['settings']['db']['access_key'] = true;
```

Если установлен флаг `true` - тогда во всех запросах к db необходимо указывать параметр

```php
$key = $config['settings']['db']['key']; // Взять key из конфигурации

// https://example.com/_12345_/api.php?key='.$key.'
// https://example.com/_12345_/table_name?key='.$key.'
// https://example.com/_12345_/table_name/id?key='.$key.'

// curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET https://example.com/_12345_/table_name?key='.$key.'

// curl --request POST "https://example.com/_12345_/table_name" --data "key='.$key.'"

```

При запросе без ключа API будет отдавать ответ

```json
{
    "status": "403",
    "code": 403,
    "message": "Access is denied"
}
```

### Создание таблиц

Вы можете создавать таблицы автоматически с использованием файла `/www/_db_/core/db.json`

Система проверяет файл `db.json` и создает все таблицы которых не существует.

По умолчанию система автоматически создает таблицы по стандарту `APIS-2018`, таким образом у вас есть полностью работоспособная база данных.

Вы можете создавать свою конфигурацию DB для этого отредактируйте `db.json` и удалите лишние таблицы в `/www/_db_/`

### Прямое подключение к DB без RESTful API

Вы можете работать с базой данных напрямую без REST API интерфейса.

<a name="feedback"></a>
## Поддержка, обратная связь, новости

Общайтесь с нами через почту open.source@pllano.com

Если вы нашли баг в работе API json DB загляните в
[issues](https://github.com/pllano/api-json-db/issues), возможно, про него мы уже знаем и
чиним. Если нет, лучше всего сообщить о нём там. Там же вы можете оставлять свои
пожелания и предложения.

За новостями вы можете следить по
[коммитам](https://github.com/pllano/api-json-db/commits/master) в этом репозитории.
[RSS](https://github.com/pllano/api-json-db/commits/master.atom).

Лицензия API json DB
-------

The MIT License (MIT). Please see [LICENSE](LICENSE.md) for more information.

