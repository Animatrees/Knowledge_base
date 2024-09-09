Протокол OAuth 2.0

Библиотеки для работы с OAuth 2.0

Установка social-auth-app-django

### Протокол OAuth 2.0

**OAuth 2.0** — это протокол авторизации, который позволяет приложениям получать ограниченный доступ к ресурсам пользователей на веб-сайтах или сервисах без необходимости раскрытия пароля. Протокол широко используется для аутентификации и авторизации в веб-приложениях, мобильных приложениях и других системах.

![[Untitled 17.png|Untitled 17.png]]

**Процесс авторизации в OAuth 2.0 можно разбить на несколько этапов:**

1. **Запрос авторизации**: Клиент перенаправляет пользователя на сервер авторизации с запросом на доступ к ресурсам.
2. **Аутентификация пользователя**: Пользователь вводит свои учетные данные на сервере авторизации и соглашается предоставить доступ к своим ресурсам.
3. **Выдача кода авторизации**: Сервер авторизации перенаправляет пользователя обратно к клиенту с временным кодом авторизации.
4. **Обмен кода на токен доступа**: Клиент отправляет код авторизации на сервер авторизации и получает токен доступа.
5. **Доступ к ресурсам**: Клиент использует токен доступа для запросов к ресурсному серверу, чтобы получить доступ к ресурсам пользователя.

### Библиотеки для работы с OAuth 2.0

1. **django-allauth**: Популярная библиотека для аутентификации с поддержкой OAuth, OpenID и многих других механизмов. Поддерживает регистрацию и вход через социальные сети и email. [Документация](https://docs.allauth.org/en/latest/)
2. **Django-Social-Auth**: Устаревшая библиотека для аутентификации через социальные сети, теперь замененная на `social-auth-app-django`. Тем не менее, она может быть полезна для старых проектов. [Документация](https://github.com/omab/django-social-auth)
3. **Python Social Auth**: Библиотека для аутентификации через социальные сети, предоставляющая множество бекендов для различных провайдеров. Легко интегрируется и поддерживает расширяемую архитектуру. [Документация](https://python-social-auth.readthedocs.io/en/latest/)
4. **social-auth-app-django**: Модуль Python Social Auth заточенный под Django. [Документация](https://python-social-auth.readthedocs.io/en/latest/configuration/django.html)
5. **django-oauth-toolkit**: Мощная библиотека для реализации серверной части OAuth2 в Django приложении. Поддерживает все типы грантов OAuth2 и легко интегрируется с существующими приложениями. [Документация](https://django-oauth-toolkit.readthedocs.io/en/latest/)
6. **django-rest-framework-social-oauth2**: Расширение для Django Rest Framework, объединяющее поддержку OAuth2 и социальных аутентификаций. Поддерживает интеграцию с DRF и стандартный OAuth2. [Документация](https://github.com/RealmTeam/django-rest-framework-social-oauth2)

### Установка social-auth-app-django

1. Установите библиотеку `social-auth-app-django` с помощью pip:
    
    ```Shell
    pip install social-auth-app-django
    ```
    
2. Добавьте `social_django` в список установленных приложений INSTALLED_APPS в файле `settings.py`:
    
    ```Python
    INSTALLED_APPS = (
        ...
        'social_django',
        ...
    )
    ```
    
3. При использовании PostgreSQL добавьте следующую настройку в `settings.py` для хранения extra_data в формате JSONB:
    
    ```Python
    SOCIAL_AUTH_JSONFIELD_ENABLED = True
    ```
    
4. Синхронизируйте базу данных для создания необходимых моделей:
    
    ```Shell
    python manage.py migrate
    ```
    
5. Добавьте нужные бекенды аутентификации в настройку AUTHENTICATION_BACKENDS в `settings.py`:
    
    ```Python
    AUTHENTICATION_BACKENDS = [
        'social_core.backends.github.GithubOAuth2',
        'django.contrib.auth.backends.ModelBackend',
        'users.authentication.EmailAuthBackend',
    ]
    ```
    
6. Добавьте записи URL в файл `urls.py` вашего проекта:
    
    ```Python
    from django.urls import path, include
    
    urlpatterns = [
        ...
        path('social-auth', include('social_django.urls', namespace='social')),
        ...
    ]
    ```
    
7. Если вам нужен пользовательский namespace, добавьте следующую настройку в `settings.py`:
    
    ```Python
    SOCIAL_AUTH_URL_NAMESPACE = 'social'
    ```

<div class="page-break" style="page-break-before: always;"></div>
