# Развертывание базы данных YDB и выполнение CRUD-операций на Windows с Python

Этот гид поможет вам шаг за шагом создать serverless-базу данных YDB в Yandex Cloud, настроить окружение и выполнить CRUD-операции через Python.

## Требования

- Установлены: PowerShell, Visual Studio Code, Git, Python 3.7+.
- Аккаунт в Yandex Cloud.

## Пошаговый процесс

1. **Создание рабочего окружения**

   - Запустите Visual Studio Code и создайте папку для проекта.

   - В терминале PowerShell внутри VS Code настройте виртуальное окружение:

     ```powershell
     python -m venv venv
     .\venv\Scripts\activate
     ```

2. **Настройка инструментов Yandex Cloud**

   - Установите CLI по инструкции: https://yandex.cloud/ru/docs/cli/operations/install-cli.

   - В консоли Yandex Cloud создайте новое облако.

   - В разделе прав доступа назначьте своему аккаунту по умолчанию (пользовательские аккаунты) роль `Admin` (права распространяются на все каталоги облака).

   - В разделе «Обзор» создайте каталог.

   - В PowerShell выполните:

     ```powershell
     yc init
     ```

     - Создайте новый профиль или выберите существующий.
     - Перейдите по ссылке из терминала, скопируйте токен и вставьте его в PowerShell.
     - Укажите созданные облако и каталог (или создайте новый через «Create a new folder»).
     - Выберите зону, например, `ru-central1-a`.

3. **Создание serverless-базы YDB**

   - В PowerShell создайте базу:

     ```powershell
     yc ydb database create <имя_базы> --serverless
     ```

     *Примечание*: Замените `<имя_базы>` на желаемое имя.

   - Дождитесь статуса `RUNNING` в консоли Yandex Cloud (раздел «Managed Service for YDB»).

   - Сохраните параметры из вывода:

     - `endpoint`: `grpcs://ydb.serverless.yandexcloud.net:2135`
     - `database`: `/ru-central1/<folder_id>/<database_id>`

4. **Установка Python SDK для YDB**

   - В терминале PowerShell (в папке проекта) выполните:

     ```powershell
     git clone https://github.com/ydb-platform/ydb-python-sdk.git
     cd ydb-python-sdk/examples/basic_example_v1
     pip install ydb iso8601
     ```

5. **Получение токена доступа**

   - Выполните:

     ```powershell
     yc iam create-token
     ```

     Токен будет выглядеть, например, как `t1.9euelZqV...`.

   - Сохраните токен:

     ```powershell
     $env:YDB_ACCESS_TOKEN_CREDENTIALS = "<ваш_токен>"
     ```

6. **Запуск тестового приложения**

   - Выполните команду для запуска CRUD-операций:

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

   - В консоли Yandex Cloud откройте базу → раздел «Навигация».
   - Просмотрите созданные таблицы и их содержимое.

## Полезные заметки

- Статус базы меняется с `PROVISIONING` на `RUNNING` через несколько минут.

- Для повторного запуска активируйте виртуальное окружение:

  ```powershell
  .\venv\Scripts\activate
  ```

- Храните токен доступа в безопасном месте, он необходим для взаимодействия с облаком.
