Дописываем авторизацию через гитхаб

*Скрытие функционала смены пароля

Скрытие функционала смены пароля, как в уроке

### Дописываем авторизацию через гитхаб

1. **Создаём приложение на GitHub:**
    - Перейти по ссылке: [https://github.com/settings/applications/new](https://github.com/settings/applications/new)
    - Заполнить поля:
        - Application name: название проекта
        - Homepage URL: [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
        - Callback URL: [http://127.0.0.1:8000/social-auth/complete/github/](http://127.0.0.1:8000/social-auth/complete/github/)
    - Нажать "Register application"
    - Если получаем ошибку, что мы не заполнили Webhook URL - отжимаем эту галку, у меня она почему-то была включена по умолчанию.
        
        ![[Untitled 18.png|Untitled 18.png]]
        
    - Получить Client ID и Client secret.
2. **Настройка в** `**settings.py**`**:**
    - Добавить параметры с полученными ключами:
        
        ```Python
        SOCIAL_AUTH_GITHUB_KEY = 'xxxxx'
        SOCIAL_AUTH_GITHUB_SECRET = 'xxxxxxxxxxx'
        ```
        
3. **Добавление кнопки авторизации через GitHub:**
    - В шаблоне `users/login.html` после формы добавить:
        
        ```Plain
        <hr>
        <p><a href="{% url 'social:begin' 'github' %}"><img src="/media/social-auth/github.png"></a></p>
        ```
        
        Если вы, как и я, решили хранить картинку с лого в папке static, не забываем про тег load, чтобы подгрузить статику в шаблон:
        
        ```Python
        {% load static %}
        ...
        <hr>
        <p><a href="{% url 'social:begin' 'github' %}"><img src="{% static 'users/images/github_logo.png' %}" width="50" height="50"></a></p>
        ```
        
4. **Возможные ошибки:**
    - Если получаем ошибку подобного типа:
        
        ![[Untitled 1 9.png|Untitled 1 9.png]]
        
        Проверяем, что не потеряли слэш в пути маршрута
        
        ![[Untitled 2 7.png|Untitled 2 7.png]]
        
    - Если авторизация прошла успешно, но мы получаем следующую ошибку:
        
        ![[Untitled 3 4.png|Untitled 3 4.png]]
        
        Дописываем адрес куда редиректить после авторизации в `settings.py`:
        
        ```Python
        LOGIN_REDIRECT_URL = '/'
        ```
        

### *Скрытие функционала смены пароля

Самый простой и быстрый способ скрыть кнопку смены пароля для пользователей, авторизованных через гитхаб (или другие способы социальной аутентификации) - добавить следующую проверку в шаблон:

```Python
{% if user.has_usable_password %}
    <p ><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
    {% endif %}
```

Что значит `if user.has_usable_password`?

Метод `has_usable_password()` является встроенным методом абстрактной модели `AbstractBaseUser`. Он предназначен для проверки того, может ли пользователь аутентифицироваться с помощью пароля.

Этот метод возвращает `True`, если в модели пользователя установлен действительный пароль, который может быть использован для аутентификации, и `False` в противном случае.

Например, когда пользователь авторизуется через социальные провайдеры, такие как GitHub или Google, Django создает запись пользователя в базе данных, но не генерирует для него реальный пароль. Вместо этого в поле `password` модели пользователя сохраняется специальный идентификатор, используемый для связи локального профиля с учетной записью на социальной платформе. В этом случае метод `has_usable_password()` вернет `False`, так как пароль не является действительным для аутентификации.

### Скрытие функционала смены пароля, как в уроке

1. Создаём разрешение "Social Auth" с кодовым именем "social_auth", если не сделали это в предыдущем уроке про разрешения)
2. Создаём группу "social" в админ-панели Django для пользователей, авторизованных через социальные сети
3. Добавляем пользователей в группу "social" при авторизации через социальные сети, используя механизм "pipeline" пакета Python-Social-Auth:
    - В файле `settings.py` определяем свой модуль pipeline и добавляем его в коллекцию SOCIAL_AUTH_PIPELINE:
        
        ```Python
        SOCIAL_AUTH_PIPELINE = (
            'social_core.pipeline.social_auth.social_details',
            'social_core.pipeline.social_auth.social_uid',
            'social_core.pipeline.social_auth.auth_allowed',
            'social_core.pipeline.social_auth.social_user',
            'social_core.pipeline.user.get_username',
            # 'social_core.pipeline.social_auth.associate_by_email',
            'social_core.pipeline.user.create_user',
            'users.pipeline.new_users_handler',# Пользовательский модуль
            'social_core.pipeline.social_auth.associate_user',
            'social_core.pipeline.social_auth.load_extra_data',
            'social_core.pipeline.user.user_details',
        )
        ```
        
        `SOCIAL_AUTH_PIPELINE` - это константа, определяющая последовательность функций (модулей), которые будут выполняться при авторизации пользователя через социальные сети с помощью пакета Python Social Auth.
        
        - `'social_core.pipeline.social_auth.social_details'` - получает базовую информацию о пользователе (имя, фамилию, электронную почту и т.д.) от социального провайдера.
        - `'social_core.pipeline.social_auth.social_uid'` - генерирует уникальный идентификатор (uid) для пользователя на основе информации, полученной от социального провайдера.
        - `'social_core.pipeline.social_auth.auth_allowed'` - проверяет, разрешена ли аутентификация через данный социальный провайдер.
        - `'social_core.pipeline.social_auth.social_user'` - пытается найти существующего пользователя в базе данных на основе полученной информации от социального провайдера.
        - `'social_core.pipeline.user.get_username'` - генерирует уникальное имя пользователя (username) в случае, если пользователь еще не существует в базе данных.
        - `'social_core.pipeline.social_auth.associate_by_email'` - этот модуль пытается связать (ассоциировать) пользователя, авторизующегося через социальную сеть, с существующим пользователем в базе данных по электронной почте.
        - `'social_core.pipeline.user.create_user'` - создает нового пользователя в базе данных, если он не был найден на предыдущем шаге.
        - `'social_core.pipeline.social_auth.associate_user'` - связывает локального пользователя с его учетной записью в социальной сети.
        - `'social_core.pipeline.social_auth.load_extra_data'` - загружает дополнительную информацию о пользователе от социального провайдера (если она есть).
        - `'social_core.pipeline.user.user_details'` - обновляет детали пользователя в базе данных на основе информации, полученной от социального провайдера.
    - Создаём файл `users/pipeline.py` с функцией `new_users_handler()`:
        
        ```Python
        from django.contrib.auth.models import Group
        
        def new_users_handler(backend, user, response, *args, **kwargs):
            group = Group.objects.filter(name='social')
            if len(group):
                user.groups.add(group[0])
        ```
        
4. В шаблоне `users/profile.html` отображаем ссылку "Сменить пароль" только для пользователей, не имеющих разрешение "social_auth":
    
    ```HTML
    {% if not perms.users.social_auth %}
    <hr>
    <p><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
    {% endif %}
    ```
    

При использовании данного подхода будет проблема с отображением функционала смены пароля для суперпользователей, чтобы этого избежать, добавляем проверку на суперпользователя:

```HTML
{% if user.is_superuser or not perms.users.social_auth %}
<hr>
<p><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
{% endif %}
```

Или используем первый способ)

Также не забываем изменить обязательность поля email в форме профиля, так как для пользователей, авторизованных через GitHub мы не сохраняем в БД данную информацию.

```HTML
class ProfileUserForm(forms.ModelForm):
    ...
    email = forms.CharField(disabled=True, required=False, label='E-mail', widget=forms.TextInput(attrs={'class': 'form-input'}))
    ...
```

<div class="page-break" style="page-break-before: always;"></div>
