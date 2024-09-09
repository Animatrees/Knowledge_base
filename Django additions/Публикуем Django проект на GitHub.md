Подготовка

Публикация

### Подготовка

Прежде чем опубликовать любой проект на GitHub нужно его правильно подготовить…

А для этого вам нужно проверить некоторые вещи:

1. **Секретные данные:**
    
    - Убедитесь, что файл `settings.py` не содержит секретных ключей, паролей или других конфиденциальных данных.
    - Используйте переменные окружения или отдельный файл конфигурации для хранения секретной информации.
    
    Несколько мини-уроков по теме:
    
    > [!info] Python и переменные окружения | Где и как хранить секреты в коде | .env, .gitignore  
    > ⚡️ Уроки Python для начинающих.  
    > [https://www.youtube.com/watch?v=Bi1Jp6yRCWw](https://www.youtube.com/watch?v=Bi1Jp6yRCWw)  
    
    > [!info] Работаем с переменными окружения в python (два способа)  
    > Здравствуйте мои маленькие любители программирования.  
    > [https://www.youtube.com/watch?v=mbtbbVc7vTQ](https://www.youtube.com/watch?v=mbtbbVc7vTQ)  
    
    Если вы читаете этот текст после курса по Django, ваш .env файл может содержать, например, следующие данные:
    
    ```Python
    SECRET_KEY=
    DEBUG=True
    ALLOWED_HOSTS=
    INTERNAL_IPS=
    DATABASE_URL=
    EMAIL_HOST=
    EMAIL_HOST_USER=
    EMAIL_HOST_PASSWORD=
    SOCIAL_AUTH_GITHUB_KEY=
    SOCIAL_AUTH_GITHUB_SECRET=
    REDIS_URL=
    ```
    
    Не забудьте добавить этот файл в .gitignore! (следующий пункт)
    
2. **.gitignore:**
    - Создайте файл `.gitignore` для исключения ненужных файлов. Подробнее про создание и исключение файлов из git-а можно почитать здесь:
        
        [  
        Файл .gitignore. Для чего и как используется](https://www.notion.so/gitignore-57b81130038d40228d8f634d25b43f63?pvs=21)
        
3. **База данных:**
    - Не включайте файлы базы данных (например, db.sqlite3) в репозиторий.
    - Добавьте их в .gitignore.
4. **Виртуальное окружение:**
    - Не добавляйте папку виртуального окружения в репозиторий.
    - Добавьте ее в .gitignore.
5. **Миграции**:
    - Решите, включать ли файлы миграций в репозиторий:
        - Плюсы:
            - Обеспечивает согласованность структуры базы данных между разработчиками.
            - Упрощает развертывание, так как миграции уже готовы к применению.
            - Сохраняет историю изменений схемы базы данных.
        - Минусы:
            - Может привести к конфликтам при слиянии веток, если несколько разработчиков создают миграции одновременно.
6. **Статические файлы:**
    - Проверьте, правильно ли настроена обработка статических файлов.
        
        - Убедитесь, что `STATIC_URL` определен (обычно '/static/').
        - Определите `STATIC_ROOT` (путь, куда Django будет собирать статические файлы) и исключите данную директорию из отслеживания git-ом.
        - При необходимости, настройте `STATICFILES_DIRS`.
        
        Пример:
        
        ```Python
        STATIC_URL = '/static/'
        STATIC_ROOT = BASE_DIR / 'staticfiles'
        STATICFILES_DIRS = [BASE_DIR / 'static']
        ```
        
    - Пример того, как может выглядеть файл `.gitignore` после предыдущих шагов.
        
        ```Visual
        # Django
        *.log
        *.pot
        __pycache__/
        media
        
        # PyCharm
        .idea/
        
        # Postgres
        *.sql
        
        # Redis
        dump.rdb
        
        # Environment variables
        .env
        
        # Kubuntu
        .directory
        
        # Virtual environment
        djvenv/
        
        # Compiled Python files
        *.pyc
        
        # Static files collected
        /staticfiles/
        
        # Media files
        /media/
        ```
        
7. **Файл requirements.txt:**
    - Создайте файл со списком всех зависимостей проекта.
    - Файл `requirements.txt` критически важен при публикации проекта на GitHub, так как он содержит список всех Python-зависимостей проекта. Это обеспечивает воспроизводимость среды разработки, упрощает процесс развертывания и совместной работы над проектом. С его помощью новые разработчики могут быстро настроить окружение, а инструменты CI/CD легко установить нужные пакеты. Файл служит своеобразной документацией используемых библиотек, помогает контролировать версии зависимостей и обеспечивает совместимость между разными системами. В целом, requirements.txt значительно облегчает управление проектом и его зависимостями.
    - Команда для создания файла:
        
        ```Shell
        pip freeze > requirements.txt
        ```
        
8. **README.md:**
    
    - Добавьте файл с описанием проекта, инструкциями по установке и запуску.
    - Структура файла:  
        Обычно  
        [README.md](http://readme.md/) содержит следующие секции:  
        a) Название проекта  
        b) Краткое описание  
        c) Установка  
        d) Использование  
        e) Настройка  
        f) Вклад в проект  
        g) Лицензия  
        
    
    - Пример того, как может выглядеть [README.md](http://README.md)
        
        ```Markdown
        # Project Title
        
        A brief description of the project and its purpose.
        
        ## Table of Contents
        
        - [Technologies](#technologies)
        - [Installation](#installation)
        - [Configuration](#configuration)
        - [Running the Project](#running-the-project)
        - [Testing](#testing)
        - [Development](#development)
        - [License](#license)
        - [Contacts](#contacts)
        
        ## Technologies
        
        - Python 3.x
        - Django 3.x/4.x
        - PostgreSQL/MySQL/SQLite (choose the appropriate one)
        - Docker (optional)
        - Other technologies and libraries used
        
        ## Installation
        
        1. Clone the repository
            ```sh
            git clone https://github.com/your-username/project-name.git
            cd project-name
            ```
        
        2. Create and activate a virtual environment
            ```sh
            python -m venv venv
            source venv/bin/activate  # use `venv\Scripts\activate` for Windows
            ```
        
        3. Install dependencies
            ```sh
            pip install -r requirements.txt
            ```
        
        4. Create a `.env` file and configure environment variables
            ```env
            DEBUG=True
            SECRET_KEY='your-secret-key'
            DATABASE_URL='your-database-url'
            ```
        
        ## Configuration
        
        1. Apply database migrations
            ```sh
            python manage.py migrate
            ```
        
        2. Create a superuser
            ```sh
            python manage.py createsuperuser
            ```
        
        3. Collect static files
            ```sh
            python manage.py collectstatic
            ```
        
        ## Running the Project
        
        1. Start the development server
            ```sh
            python manage.py runserver
            ```
        
        2. Open your browser and go to [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
        
        ## Testing
        
        1. Run tests
            ```sh
            python manage.py test
            ```
        
        ## Development
        
        ### Project Structure
        
        A brief description of the project structure and main directories.
        
        - `project_name/` - root directory of the project
        - `project_name/settings/` - Django settings
        - `project_name/urls.py` - routing
        - `app_name/` - your application
        - etc.
        
        ### Common Commands
        
        - Create an application
            ```sh
            python manage.py startapp app_name
            ```
        
        - Apply migrations
            ```sh
            python manage.py migrate
            ```
        
        - Create migrations
            ```sh
            python manage.py makemigrations
            ```
        
        ## License
        
        Specify the license, for example, MIT.
        
        ## Contacts
        
        Provide contact information for feedback.
        
        - Email: your.email@example.com
        - Telegram: [@your_username](https://t.me/your_username)
        - Other contact methods
        ```
        
9. **Лицензия:**
    - Добавьте файл LICENSE с выбранной вами лицензией. Почитать подробнее про разные типы лицензий можно здесь:
        
        > [!info] Licensing a repository - GitHub Docs  
        > Public repositories on GitHub are often used to share open source software.  
        > [https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository)  
        
        Для простого пет проекта можно использовать MIT лицензию. Лицензия MIT позволяет свободное использование, копирование, модификацию и распространение программного обеспечения при условии, что везде будет указан оригинальный автор и текст лицензии, а сам код предоставляется без каких-либо гарантий.
        
        - Пример.
            
            ```Markdown
            # MIT-LICENSE.txt
            
            MIT License
            
            Copyright (c) 2024 First name Last name and others
            
            Permission is hereby granted, free of charge, to any person obtaining
            a copy of this software and associated documentation files (the
            "Software"), to deal in the Software without restriction, including
            without limitation the rights to use, copy, modify, merge, publish,
            distribute, sublicense, and/or sell copies of the Software, and to
            permit persons to whom the Software is furnished to do so, subject to
            the following conditions:
            
            The above copyright notice and this permission notice shall be
            included in all copies or substantial portions of the Software.
            
            THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
            EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
            MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
            NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
            LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
            OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
            WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
            ```
            
            Не забудьте в начале лицензии использовать ваше имя)
            
10. **Проверка кода:**
    - Удалите неиспользуемый код и отладочные принты.
    - Убедитесь, что код соответствует стандартам PEP 8.

## Публикация

Итак, после того, как все подготовительные работы выполнены, переходим к публикации!)

1. Заходим в аккаунт на GitHub, тыкаем в создание нового репо
    
    ![[Untitled 22.png|Untitled 22.png]]
    
2. Называем проект и выбираем в каком виде вы хотите разместить его на репозитории - приватном или публичном.
    
    ![[Untitled 1 12.png|Untitled 1 12.png]]
    
    Нажимаем создать.
    
3. Далее выполняем ряд команд в терминале (находясь в папке проекта):
    
    ```Shell
    # инициализация git в проекте
    git init
    # переименовываем основную ветку в main
    # не обязательно, но рекомендуется github-ом
    git branch -M main
    # добавление всех файлов в git (кроме тех, что в .gitignore)
    git add .
    # создаём первый коммит
    git commit -m "Initial commit"
    # связываем наш локальный проект с проектом на GitHub
    git remote add origin git@github.com:yourusername/yourproject.git
    # заливаем файлы на GitHub
    git push -u origin main
    ```
    

Вот, собственно и все шаги)

Подробнее про Git и работу с ним писала здесь:

[![](https://www.notion.so/icons/git_green.svg)Git 101](https://www.notion.so/Git-101-d36cf8ea9bd440649148cd71c0e4f406?pvs=21)