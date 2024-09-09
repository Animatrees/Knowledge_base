Установка СУБД PostgreSQL

Установка PGAdmin4

Создание базы данных и пользователя-администратора

Установка пакета psycopg3

Подключение PostgreSQL к проекту Django

Выполнение миграций для создания необходимых таблиц

Загрузка данных из фикстур

### **Установка СУБД PostgreSQL**

1. **Обновление списка пакетов и установка PostgreSQL:**
    
    ```Shell
    sudo apt update
    sudo apt install postgresql postgresql-contrib
    ```
    
    [Документация по установке PostgreSQL](https://www.postgresql.org/download/linux/ubuntu/)
    
2. **Запуск и проверка статуса сервиса PostgreSQL:**
    
    ```Shell
    sudo systemctl start postgresql
    sudo systemctl enable postgresql
    sudo systemctl status postgresql
    ```
    
3. **Добавление пароля суперпользователю:**
    
    По умолчанию после установки PostgreSQL создается суперпользователь postgres без пароля, что позволяет войти в базу данных без аутентификации.
    
    Чтобы добавить пароль:
    
    - Переключиться на суперпользователя postgres:
        
        ```Shell
        sudo -u postgres psql
        ```
        
    - Внутри psql выполнить команду для смены пароля:
        
        ```SQL
        ALTER USER postgres PASSWORD 'новый_пароль';
        ```
        
    - Выйти из psql:
        
        ```SQL
        \q
        ```
        

### Установка PGAdmin4

1. Добавить репозиторий PgAdmin:
    
    ```Plain
    sudo apt-get install ca-certificates wget
    wget --quiet -O - https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add -
    sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list'
    ```
    
2. Обновить список пакетов:
    
    ```Plain
    sudo apt-get update
    ```
    
3. Установить PgAdmin4:
    
    ```Plain
    sudo apt-get install pgadmin4
    ```
    

### **Создание базы данных и пользователя-администратора**

1. Войдите в PostgreSQL под учетной записью суперпользователя postgres:
    
    ```Shell
    sudo -u postgres psql
    ```
    
2. Создайте нового суперпользователя для вашей новой базы данных:
    
    ```SQL
    CREATE ROLE имя_суперпользователя WITH LOGIN SUPERUSER PASSWORD 'пароль';
    ```
    
    Замените `имя_суперпользователя` и `пароль` нужными значениями.
    
3. Создайте новую базу данных:
    
    ```SQL
    CREATE DATABASE имя_базы_данных OWNER имя_суперпользователя;
    ```
    
    Замените `имя_базы_данных` и `имя_суперпользователя` соответствующими значениями.
    
4. Выйдите из psql:
    
    ```SQL
    \q
    ```
    

### **Установка пакета** `**psycopg3**`

Ставим именно эту версию, так как

> Support for `psycopg2` is likely to be deprecated and removed at some point in the future.

В терминале pycharm:

```Shell
pip install 'psycopg[binary]'
```

Если вы используете `zsh`, чтобы избежать ошибки, экранируйте квадратные скобки обратными слэшами:

```Shell
pip install psycopg\[binary\]
```

[Документация](https://www.psycopg.org/psycopg3/docs/).

### **Подключение PostgreSQL к проекту Django**

Найдите константу `DATABASES` в `settings.py` и измените ее следующим образом:

```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'имя_базы_данных',
        'USER': 'имя_суперпользователя',
        'PASSWORD': 'пароль',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

[Документация по настройке баз данных в Django](https://docs.djangoproject.com/en/4.0/ref/settings/#databases)

### **Выполнение миграций для создания необходимых таблиц**

Создаём все необходимые таблицы в базе данных.

```Shell
python manage.py migrate
```

### **Загрузка данных из фикстур**

Загружаем fixture:

```Shell
python manage.py loaddata db.json
```

<div class="page-break" style="page-break-before: always;"></div>
