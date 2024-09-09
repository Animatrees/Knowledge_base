### Установка Django

1. Открываем созданный на предыдущем шаге проект в PyCharm.
2. В PyCharm открываем терминал и убеждаемся, что виртуальное окружение было создано и мы в него вошли.
    
    ![[Untitled 9.png|Untitled 9.png]]
    
3. Если не вошли, то проходим следующую процедуру: **File → Settings → Project: django → Python Interpreter → Add Interpreter → Add local … → Existing → Ok → Ok**
4. Устанавливаем Django через терминал PyCharm командой `pip install django==4.2.1`
5. После установки можно проверить список доступных команд `django-admin`
    
    Примерный список команд:
    
    check  
    compilemessages  
    createcachetable  
    dbshell  
    diffsettings  
    dumpdata  
    flush  
    inspectdb  
    loaddata  
    makemessages  
    makemigrations  
    migrate  
    optimizemigration  
    runserver  
    sendtestemail  
    shell  
    showmigrations  
    sqlflush  
    sqlmigrate  
    sqlsequencereset  
    squashmigrations  
    startapp  
    startproject  
    test  
    testserver  
    

### Создание проекта и запуск тестового веб-сервера

1. Чтобы создать структуру каталогов для нового проекта в Django, используем команду `django-admin startproject project-name`, где `project-name` имя вашего проекта. Принято давать имена, которые будут соответствовать будущему доменному имени сайта.
    
    Для нового проекта будет создана следующая файловая структура:
    
    ![[Untitled 1 2.png|Untitled 1 2.png]]
    
    **Описание файлов:**
    
    - `**_init__.py**`**:** Этот файл используется для обозначения пакета Python. Он должен присутствовать в каждой папке, которая содержит модули Python.
    - `**asgi.py**`**:** Этот файл определяет интерфейс ASGI (Asynchronous Server Gateway Interface) для вашего приложения. ASGI - это асинхронный протокол, который позволяет веб-серверам взаимодействовать с приложениями Python.
    - `**settings.py**`**:** Этот файл содержит все настройки вашего проекта Django. Он включает в себя такие параметры, как имя базы данных, секретный ключ, URL-адрес сервера и т. д.
    - `**urls.py**`**:** Этот файл определяет URL-маршруты вашего приложения. Он сопоставляет URL-адреса с представлениями, которые генерируют ответы HTTP.
    - `**wsgi.py**`**:** Этот файл определяет интерфейс WSGI (Web Server Gateway Interface) для вашего приложения.
    - `**manage.py**`**:** Этот файл сценария Python используется для управления проектом Django. Он позволяет выполнять такие задачи, как создание миграций базы данных, запуск сервера и выполнение тестов.
2. Переходим в созданный проект командой `cd project-name`.
3. Чтобы запустить тестовый веб-сервер используем команду `python manage.py runserver`
    
    ![[Untitled 2 2.png|Untitled 2 2.png]]
    
    По умолчанию используется порт 8000, но мы можем указать любой порт, например, `python manage.py runserver 4000` или `python manage.py runserver localhost 4000`
    
    При первом запуске сервера в проекте появится еще один файл db.sqlite3 – файл Базы данных SQLite3.
    
4. Для остановки сервера `ctrl + c`.

<div class="page-break" style="page-break-before: always;"></div>
