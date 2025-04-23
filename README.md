# Развертывание YDB и выполнение CRUD-операций на Windows с Python

Эта инструкция поможет настроить Yandex Database (YDB), развернуть серверлесс-базу и выполнить CRUD-операции с использованием Python.

## Требования

- Установлены: PowerShell, Visual Studio Code, Git, Python 3.7+.
- Аккаунт в Yandex Cloud.

## Пошаговое выполнение

1. **Настройка рабочего окружения**

   - В Visual Studio Code создайте папку, например, `yandex-db`.
   - Откройте терминал PowerShell в VS Code и создайте виртуальное окружение:

     ```powershell
     python -m venv venv
     .\venv\Scripts\activate
     ```

2. **Установка и настройка Yandex Cloud CLI**

   - Установите CLI по инструкции: https://yandex.cloud/ru/docs/cli/operations/install-cli.
   - В браузере на https://console.yandex.cloud создайте облако, например, `dbcloude`.
   - В разделе облака `dbcloude` → «Права доступа» назначьте своему аккаунту роль `Admin`.\
     *Примечание*: Права `Admin` наследуются на все каталоги внутри облака.
   - В разделе «Обзор» создайте каталог, например, `dbcatalog`.
   - В PowerShell выполните:

     ```powershell
     yc init
     ```
     - Создайте новый профиль, например, `user`.
     - Перейдите по ссылке для получения токена, скопируйте и вставьте его в PowerShell.
     - Выберите облако `dbcloude` и каталог `dbcatalog` (или создайте новый).
     - Укажите зону по умолчанию, например, `ru-central1-a`.

3. **Создание serverless-базы YDB**

   - В PowerShell создайте базу данных:

     ```powershell
     yc ydb database create my-ydb --serverless
     ```

     *Примечание*: Замените `my-ydb` на желаемое имя базы.
   - Дождитесь статуса `RUNNING` в консоли Yandex Cloud (раздел «Managed Service for YDB» в каталоге).
   - Сохраните параметры из вывода:
     - `endpoint`: `grpcs://ydb.serverless.yandexcloud.net:2135` (по умолчанию)
     - `database`: `/ru-central1/<folder_id>/<database_id>`

4. **Установка YDB Python SDK**

   - В терминале PowerShell (в папке `yandex-db`) выполните:

     ```powershell
     git clone https://github.com/ydb-platform/ydb-python-sdk.git
     cd ydb-python-sdk/examples/basic_example_v1
     pip install ydb iso8601
     ```

5. **Получение IAM-токена**

   - Выполните:

     ```powershell
     yc iam create-token
     ```

     Пример вывода: `t1.9euelZqV...`
   - Сохраните токен в переменную:

     ```powershell
     $env:YDB_ACCESS_TOKEN_CREDENTIALS = "<ваш_токен>"
     ```

6. **Запуск тестового приложения с CRUD-операциями**

   - Выполните:

     ```powershell
     python __main__.py -e grpcs://ydb.serverless.yandexcloud.net:2135 -d /ru-central1/<folder_id>/<database_id>
     ```
   - Ожидаемый вывод:

     ```
     > describe table: series
     column, name: series_id, Uint64
     column, name: title, Utf8
     column, name: series_info, Utf8
     column, name: release_date, Uint64
     
     > select_simple_transaction:
     series, id: 1, title: IT Crowd, release date: b'2006-02-03'
     
     > bulk upsert: episodes
     
     > select_prepared_transaction:
     episode title: To Build a Better Beta, air date: b'2016-06-05'
     
     > select_prepared_transaction:
     episode title: Bachman's Earnings Over-Ride, air date: b'2016-06-12'
     
     > select_parametrized_transaction:
     episode title: Daily Active Users, air date: b'2016-06-19'
     
     > select_parametrized_transaction:
     episode title: The Uptick, air date: b'2016-06-26'
     
     > explicit TCL call
     
     > select_parametrized_transaction:
     episode title: TBD, air date: b'2025-04-23'
     ```

7. **Проверка результатов**

   - В консоли Yandex Cloud перейдите в базу `my-ydb` → «Навигация».
   - Просмотрите таблицы `series` и `episodes` и их содержимое.

## Примечания

- Статус базы данных переходит из `PROVISIONING` в `RUNNING` через несколько минут.
- Для повторного запуска активируйте виртуальное окружение:

  ```powershell
  .\venv\Scripts\activate
  ```
- Храните IAM-токен в безопасном месте, он нужен для аутентификации.

Эта инструкция позволяет быстро развернуть YDB и выполнить CRUD-операции в Yandex Cloud с использованием Python.
