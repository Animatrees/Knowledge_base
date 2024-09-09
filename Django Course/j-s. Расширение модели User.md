Способы расширения модели

Создание связанной модели подробно

Создание кастомной модели User подробно

AbstractBaseUser

PermissionsMixin

AbstractUser

Реализация кастомной модели User

## Способы расширения модели

1. **Создание связанной модели:**
    
    Создается отдельная модель (например, `Profile`), которая связывается с моделью `User` через отношение "один к одному" (`OneToOneField`). Дополнительные данные пользователя хранятся в этой связанной модели.
    
    Плюсы:
    
    - Простота реализации и поддержки
    - Не требует изменений в существующей модели User
    - Позволяет хранить дополнительные данные отдельно от основной модели User
    
    Минусы:
    
    - Необходимость выполнять дополнительные запросы для получения связанных данных
    - Усложнение кода при работе с пользователями и их профилями
2. **Создание кастомной модели User:**
    
    Создается новая модель `User`, которая наследуется от встроенного класса `AbstractUser`. Эта модель может включать дополнительные поля и методы, специфичные для приложения.
    
    Плюсы:
    
    - Возможность добавления любых дополнительных полей непосредственно в модель User
    - Отсутствие необходимости в дополнительных запросах для получения данных пользователя
    - Упрощение кода при работе с пользователями
    
    Минусы:
    
    - Необходимость изменения существующей модели User
    - Потенциальные конфликты с сторонними приложениями, которые ожидают стандартную модель User
3. ***Использование прокси-модели (Proxy Model):**
    
    Создается прокси-модель, которая наследуется от модели `User`, но не создает новую таблицу в базе данных. Прокси-модель позволяет добавлять дополнительные методы и свойства к модели User без изменения ее структуры.
    
    Плюсы:
    
    - Возможность добавления дополнительных методов и свойств к модели User без изменения ее структуры
    - Сохранение совместимости со стандартной моделью User
    
    Минусы:
    
    - Невозможность добавления новых полей в модель User
    - Ограниченность функциональности по сравнению с другими подходами0
4. ***Использование AbstractBaseUser и PermissionsMixin:**
    
    Создается полностью настраиваемая модель `User`, которая наследуется от `AbstractBaseUser` и `PermissionsMixin`. Этот подход дает полный контроль над полями и поведением модели User, но требует реализации дополнительных методов и классов менеджеров.
    
    Плюсы:
    
    - Полная гибкость в определении полей и поведения модели User
    - Возможность реализации собственной логики аутентификации и авторизации
    
    Минусы:
    
    - Сложность реализации и поддержки
    - Необходимость реализации дополнительных методов и классов менеджеров
    - Потенциальная несовместимость с некоторыми сторонними приложениями

## **Создание связанной модели подробно**

У этого способа действительно больше танцев с бубнами, чем у способа с переопределением модели)) по крайней мере мне так показалось на первый взгляд)

Во-первых, всегда нужно учитывать в коде есть ли у пользователя профиль. То есть нам нужно либо предусмотреть его автоматическое создание при регистрации на самых начальных этапах, либо проверять его наличие перед обращением к атрибутам, которые мы хотим из этого профиля получить.

**Итак, реализация по порядку.**

1. Создаём связанную модель Profile.
    
    ```Python
    \#users/models.py
    
    class Profile(models.Model):
        user = models.OneToOneField(User, on_delete=models.CASCADE, verbose_name='profile')
        photo = models.ImageField(upload_to='users/profile/', blank=True, null=True, verbose_name='Фотография')
        birth_date = models.DateField(blank=True, null=True, verbose_name='Дата рождения')
    
        def __str__(self):
            return self.user.username
    ```
    
2. Добавляем поля из связанной модели в форму профиля.
    
    ```Python
    \#users/forms.py
    
    class ProfileUserForm(forms.ModelForm):
        username = forms.CharField(disabled=True, label='Логин', widget=forms.TextInput(attrs={'class': 'form-input'}))
        email = forms.CharField(disabled=True, label='E-mail', widget=forms.TextInput(attrs={'class': 'form-input'}))
        photo = forms.ImageField(label='Фотография', required=False, widget=forms.FileInput(attrs={'class': 'form-input'}))
        this_year = datetime.date.today().year
        birth_date = forms.DateField(label='Дата рождения', required=False,
                                     widget=forms.SelectDateWidget(years=tuple(range(this_year - 100, this_year - 5)),
                                                                   attrs={'class': 'form-input'}))
    
        class Meta:
            model = get_user_model()
            fields = ['username', 'email', 'first_name', 'last_name']
            labels = {
                'first_name': 'Имя',
                'last_name': 'Фамилия',
            }
            widgets = {
                'first_name': forms.TextInput(attrs={'class': 'form-input'}),
                'last_name': forms.TextInput(attrs={'class': 'form-input'}),
            }
    
        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)
            try:
                profile = self.instance.profile
            except Profile.DoesNotExist:
                pass
            else:
                self.fields['photo'].initial = profile.photo
                self.fields['birth_date'].initial = profile.birth_date
    
        def save(self, commit=True):
            user = super().save(commit=False)
            try:
                profile = user.profile
            except Profile.DoesNotExist:
                profile = Profile(user=user)
    
            profile.photo = self.cleaned_data.get('photo', profile.photo)
            profile.birth_date = self.cleaned_data.get('birth_date', profile.birth_date)
    
            if commit:
                user.save()
                profile.save()
            return user
    ```
    
    - Цель метода `__init__` в данном случае - инициализировать поля `photo` и `birth_date` формы значениями из связанного профиля пользователя, если он существует.
    - Метод `save` позволяет сохранить данные формы в модели пользователя и связанном профиле, независимо от того, существовал ли профиль ранее или нет. Если профиль не существует, он будет создан автоматически при сохранении формы.
3. Обновляем шаблон для отображения профиля.
    
    ```HTML
    {% extends 'base.html' %}
    {% load static %}
    
    {% block content %}
    <h1>Профиль</h1>
    
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {% if user.profile.photo %}
        <p ><img src="{{ user.profile.photo.url }}" width="100" height="100">
        {% else %}
        <p ><img src="{% static 'users/images/ava.png' %}" width="100" height="100">
        {% endif %}
        <div class="form-error">{{ form.non_field_errors }}</div>
        {% for f in form %}
        <p><label class="form-label" for="{{ f.id_for_label }}">{{f.label}}: </label>{{ f }}</p>
        <div class="form-error">{{ f.errors }}</div>
        {% endfor %}
    
        <p><button type="submit">Сохранить</button></p>
        <p ><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
    </form>
    {% endblock %}
    ```
    
4. Добавляем в админку отображение полей из профиля.
    
    ```Python
    \#users/admin.py
    
    from django.contrib import admin
    from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
    from django.contrib.auth.models import User
    from django.utils.safestring import mark_safe
    
    from .models import Profile
    
    
    class ProfileInline(admin.StackedInline):
        model = Profile
        can_delete = False
        verbose_name_plural = 'profile'
    
    
    class UserAdmin(BaseUserAdmin):
        list_display = ('username', 'show_photo', 'show_birth_date', 'first_name', 'last_name', 'email', 'is_staff')
        inlines = (ProfileInline,)
    
        @admin.display(description='Фотография')
        def show_photo(self, user: User):
            if user.profile.photo:
                return mark_safe(f'<img src="{user.profile.photo.url}" alt="Фото для {user.username}" width=70')
    
        @admin.display(description='Дата рождения')
        def show_birth_date(self, user: User):
            return user.profile.birth_date
    
    
    admin.site.unregister(User)
    admin.site.register(User, UserAdmin)
    ```
    
    С помощью `ProfileInline` мы добавляем поля photo и birth_date внутрь формы редактрирования записи.
    
    А используя вычисляемые поля `show_photo` и `show_birth_date`, добавляем информацию на основную страницу с пользователями.
    
    При этом мы получаем дублирующие запросы к базе данных из-за обращение к связанной модели) здесь не оптимизировала, но можно сделать самостоятельно по описанному ранее [[g-g. Отображение связанных полей в форме редактирования]].
    
5. *Для того, чтобы гарантировать создание профиля в момент регистрации пользователя можно либо добавить дополнительную логику при сохранении данных из формы, либо

## **Создание кастомной модели User подробно**

Чтобы понимать что именно мы наследуем от модели `AbstractUser`, предлагаю для начала покопаться во внутренностях этого класса) А начнём с того, что сам он наследует часть своей функциональности от классов `AbstractBaseUser` и `PermissionsMixin`.

### `AbstractBaseUser`

Основное назначение класса `AbstractBaseUser` в Django - служить базовым классом для создания кастомных моделей пользователей. Он предоставляет основную функциональность и поля, необходимые для работы с пользователями в системе аутентификации Django.

**Ключевые аспекты назначения** **`AbstractBaseUser`****:**

1. Абстрактность: `AbstractBaseUser` является абстрактным классом, что означает, что он не может быть использован напрямую как модель пользователя. Вместо этого он должен быть расширен и дополнен необходимыми полями и методами для создания конкретной модели пользователя.
2. Хранение пароля: `AbstractBaseUser` предоставляет поле `password` для хранения пароля пользователя. Он также содержит методы для установки пароля (`set_password()`), проверки пароля (`check_password()`) и установки непригодного пароля (`set_unusable_password()`).
3. Аутентификация: Класс предоставляет методы и свойства, связанные с аутентификацией пользователя, такие как `is_authenticated` и `get_session_auth_hash()`. Эти методы позволяют проверять, был ли пользователь аутентифицирован, и управлять сессиями пользователя.
4. Идентификация пользователя: `AbstractBaseUser` использует поле, указанное в атрибуте `USERNAME_FIELD`, в качестве уникального идентификатора пользователя. Это поле должно быть переопределено в подклассах для указания конкретного поля, которое будет использоваться в качестве идентификатора пользователя (например, `username` или `email`).

- Поля, методы, атрибуты подробно.
    - **Поля модели:**
        - `password`: Поле для хранения пароля пользователя. Имеет тип `CharField` с максимальной длиной 128 символов.
        - `last_login`: Поле для хранения даты и времени последнего входа пользователя. Имеет тип `DateTimeField` и может быть пустым (`blank=True, null=True`).
    - **Атрибуты класса:**
        - `is_active`: Булево значение, указывающее, активен ли пользователь. По умолчанию равно `True`.
        - `REQUIRED_FIELDS`: Список полей, которые необходимо заполнить при создании пользователя через командную строку (`createsuperuser`). По умолчанию пустой список.
        - `USERNAME_FIELD`: Имя поля, которое будет использоваться в качестве уникального идентификатора пользователя. Должно быть переопределено в подклассах.
        - `EMAIL_FIELD`: Имя поля, которое будет использоваться для хранения адреса электронной почты пользователя. Должно быть переопределено в подклассах, если используется электронная почта.
    - **Методы:**
        - `__str__()`: Возвращает строковое представление пользователя, используя метод `get_username()`.
        - `save(*args, **kwargs)`: Сохраняет пользователя в базе данных. Если был установлен новый пароль через `set_password()`, вызывает `password_validation.password_changed()` для выполнения валидации пароля.
        - `get_username()`: Возвращает значение поля, указанного в `USERNAME_FIELD`.
        - `clean()`: Нормализует имя пользователя, используя метод `normalize_username()`.
        - `natural_key()`: Возвращает кортеж, содержащий имя пользователя, которое может быть использовано в качестве естественного ключа.
        - `set_password(raw_password)`: Устанавливает пароль пользователя, выполняя хеширование пароля с помощью `make_password()`.
        - `check_password(raw_password)`: Проверяет, соответствует ли переданный пароль `raw_password` сохраненному паролю пользователя. Возвращает булево значение.
        - `set_unusable_password()`: Устанавливает пароль пользователя в непригодное значение, чтобы его нельзя было использовать для аутентификации.
        - `has_usable_password()`: Возвращает `False`, если для пользователя был установлен непригодный пароль с помощью `set_unusable_password()`.
        - `get_session_auth_hash()`: Возвращает HMAC пароля пользователя, используемый для инвалидации сессии при изменении пароля.
    - **Методы класса:**
        - `get_email_field_name()`: Возвращает имя поля электронной почты, указанное в атрибуте `EMAIL_FIELD`. По умолчанию возвращает `'email'`.
        - `normalize_username(username)`: Применяет нормализацию Unicode NFKC к имени пользователя, чтобы визуально идентичные символы с разными кодовыми точками Unicode считались одинаковыми.
    - **Свойства:**
        
        - `is_anonymous`: Всегда возвращает `False`. Используется для сравнения объектов `User` с анонимными пользователями.
        - `is_authenticated`: Всегда возвращает `True`. Используется для проверки, был ли пользователь аутентифицирован.
        
          
        

### `PermissionsMixin`

Миксин `PermissionsMixin` предоставляет функциональность для управления разрешениями и группами пользователей. Он добавляет необходимые поля и методы для поддержки моделей `Group` и `Permission` при использовании `ModelBackend` для аутентификации.

1. **Поля модели:**
    - `is_superuser`: Булево поле, указывающее, является ли пользователь суперпользователем. Если значение `True`, пользователь имеет все разрешения без явного их назначения.
    - `groups`: Поле `ManyToManyField`, представляющее группы, к которым принадлежит пользователь. Пользователь получает все разрешения, назначенные каждой из его групп.
    - `user_permissions`: Поле `ManyToManyField`, представляющее специфические разрешения, назначенные непосредственно пользователю.
2. **Методы:**
    - `get_user_permissions(obj=None)`: Возвращает список строк разрешений, которые пользователь имеет напрямую. Если передан объект `obj`, возвращает только разрешения, связанные с этим объектом.
    - `get_group_permissions(obj=None)`: Возвращает список строк разрешений, которые пользователь имеет через свои группы. Если передан объект `obj`, возвращает только разрешения, связанные с этим объектом.
    - `get_all_permissions(obj=None)`: Возвращает список всех разрешений, которые пользователь имеет, как напрямую, так и через группы. Если передан объект `obj`, возвращает только разрешения, связанные с этим объектом.
    - `has_perm(perm, obj=None)`: Возвращает `True`, если пользователь имеет указанное разрешение `perm`. Если пользователь активен (`is_active=True`) и является суперпользователем (`is_superuser=True`), этот метод всегда возвращает `True`. Если передан объект `obj`, проверяет разрешение для этого конкретного объекта.
    - `has_perms(perm_list, obj=None)`: Возвращает `True`, если пользователь имеет все указанные разрешения из списка `perm_list`. Если передан объект `obj`, проверяет разрешения для этого конкретного объекта.
    - `has_module_perms(app_label)`: Возвращает `True`, если пользователь имеет любые разрешения в указанном приложении (по метке приложения `app_label`). Если пользователь активен (`is_active=True`) и является суперпользователем (`is_superuser=True`), этот метод всегда возвращает `True`.

> [!important]  
> Миксин PermissionsMixin тесно связан с использованием ModelBackend для аутентификации. Если вы не наследуетесь от PermissionsMixin в своей модели пользователя, необходимо убедиться, что вы не вызываете методы разрешений из ModelBackend, поскольку ModelBackend предполагает наличие определенных полей в модели пользователя. Если эти поля отсутствуют, при проверке разрешений могут возникнуть ошибки базы данных.  

### `AbstractUser`

Итак, какую же функциональность реализует сам класс `AbstractUser` помимо той, что он получает от родительских классов?

Класс `AbstractUser` предоставляет базовую структуру и функциональность для создания пользовательских моделей пользователей в Django. Он включает в себя поля для хранения информации о пользователе, такие как имя пользователя, имя, фамилия, адрес электронной почты и др.

- **Поля модели:**
    - `username`: Обязательное поле типа `CharField`, представляющее имя пользователя. Имеет максимальную длину 150 символов и должно быть уникальным. Допустимы только буквы, цифры и символы `@/./+/-/_`. Используется валидатор `UnicodeUsernameValidator` для проверки корректности имени пользователя.
    - `first_name`: Поле типа `CharField`, представляющее имя пользователя. Максимальная длина - 150 символов. Может быть пустым.
    - `last_name`: Поле типа `CharField`, представляющее фамилию пользователя. Максимальная длина - 150 символов. Может быть пустым.
    - `email`: Поле типа `EmailField`, представляющее адрес электронной почты пользователя. Может быть пустым.
    - `is_staff`: Булево поле, указывающее, может ли пользователь войти в административный сайт. По умолчанию - `False`.
    - `is_active`: Булево поле, указывающее, должен ли пользователь считаться активным. По умолчанию - `True`. Вместо удаления учетных записей пользователей рекомендуется использовать это поле. Знаю лично пример, когда “разработчик” снёс из базы пользователя, на которого было завязано много сущностей в системе и сломал работу сайта)
    - `date_joined`: Поле типа `DateTimeField`, представляющее дату и время регистрации пользователя. По умолчанию - текущая дата и время.
- **Менеджер модели:**
    - `objects`: Экземпляр класса `UserManager`, который является менеджером модели пользователя. Предоставляет методы для создания и управления пользователями.
- **Константы класса:**
    - `EMAIL_FIELD`: Строка, указывающая имя поля электронной почты. По умолчанию - `"email"`.
    - `USERNAME_FIELD`: Строка, указывающая имя поля имени пользователя. По умолчанию - `"username"`.
    - `REQUIRED_FIELDS`: Список полей, которые должны быть заполнены при создании пользователя через командную строку (`createsuperuser`). По умолчанию - `["email"]`.
- **Метаданные модели:**
    - `verbose_name`: Удобочитаемое имя модели в единственном числе. По умолчанию - `"user"`.
    - `verbose_name_plural`: Удобочитаемое имя модели во множественном числе. По умолчанию - `"users"`.
    - `abstract`: Булево значение, указывающее, что класс является абстрактным и не должен создавать таблицу в базе данных. Должно быть установлено в `True`.
- **Методы:**
    - `clean()`: Переопределяет метод `clean()` родительского класса. Нормализует адрес электронной почты, вызывая `BaseUserManager.normalize_email()`. При переопределении этого метода необходимо вызвать `super().clean()` для сохранения нормализации.
    - `get_full_name()`: Возвращает полное имя пользователя, объединяя `first_name` и `last_name` с пробелом между ними.
    - `get_short_name()`: Возвращает короткое имя пользователя, которое является значением поля `first_name`.
    - `email_user(subject, message, from_email=None, **kwargs)`: Отправляет электронное письмо пользователю. Принимает тему письма, сообщение, адрес отправителя (необязательно) и дополнительные аргументы для функции `send_mail()`.

Вот столько всего уже написано за нас и для нас) Остаётся только отнаследоваться от класса `AbstractUser` и добавить поля, которых нам не хватает.

### Реализация кастомной модели `User`

Для сохранения базы данных и даже той же таблицы с пользователями, что уже была создана базовым классом, я использовала следующий алгоритм:

1. Создаём класс модели `User` со следующим содержимым:
    
    ```Python
    from django.contrib.auth.models import AbstractUser
    from django.db import models
    
    
    # Create your models here.
    class User(AbstractUser):
    
        class Meta:
            db_table = 'auth_user'
    ```
    
2. Добавляем наш класс в настройки `settings.py`:
    
    ```Python
    AUTH_USER_MODEL = 'users.User'
    ```
    
3. Комментируем в файлах `settings.py` и `urls.py` строчки с admin:
    
    ```Python
    INSTALLED_APPS = [
        # 'django.contrib.admin',
        ...
    ]
    
    urlpatterns = [
        # path('admin/', admin.site.urls),
        ...
    ]
    ```
    
4. Создаём миграцию:
    
    ```Python
    python manage.py makemigrations
    ```
    
5. Удаляем миграцию в приложении women (или как оно называется в вашем случае), у меня это номер 0024 - `0024_alter_tags_options_post_author_alter_post_photo.py`
6. Фейково применяем миграцию из users:
    
    ```Python
    python manage.py migrate --fake users 0001_initial
    ```
    
7. Возвращаем то, что закомментировали на 3 шаге.
8. Добавляем в модель User нужные нам поля:
    
    ```Python
    class User(AbstractUser):
        photo = models.ImageField(upload_to="users/%Y/%m/%d/", blank=True, null=True, verbose_name="Фотография")
        date_birth = models.DateTimeField(blank=True, null=True, verbose_name="Дата рождения")
    
        class Meta:
            db_table = 'auth_user'
    ```
    
9. И наконец опять создаём миграции и применяем их.
    
    ```Python
    python manage.py makemigrations
    python manage.py migrate
    ```
    

Профит!)

![[Untitled 16.png|Untitled 16.png]]

После этого можно добавить отображение кастомных полей в админке:

```Python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.utils.safestring import mark_safe

from users.models import User


class CustomUserAdmin(UserAdmin):
    model = User
    list_display = ['username', 'show_photo', 'photo', 'email', 'first_name', 'last_name', 'date_birth', 'is_staff',
                    'is_active']
    # добавляем поля на страницу редактирования
    fieldsets = UserAdmin.fieldsets + (
        (None, {'fields': ('photo', 'date_birth')}),
    )
    # добавляем поля в форму создания нового пользователя
    add_fieldsets = UserAdmin.add_fieldsets + (
        (None, {'fields': ('photo', 'date_birth')}),
    )

    @admin.display(description='Фото')
    def show_photo(self, user: User):
        if user.photo:
            return mark_safe(f'<img src="{user.photo.url}" alt="Фото для {user.username}" width=70')


admin.site.register(User, CustomUserAdmin)
```

И на самом сайте в профиль пользователя:

```Python
class ProfileUserForm(forms.ModelForm):
    username = forms.CharField(disabled=True, label='Логин', widget=forms.TextInput(attrs={'class': 'form-input'}))
    email = forms.CharField(disabled=True, label='E-mail', widget=forms.TextInput(attrs={'class': 'form-input'}))
    this_year = datetime.date.today().year
    date_birth = forms.DateField(label='Дата рождения', widget=forms.SelectDateWidget(years=tuple(range(this_year - 100, this_year - 5))))

    class Meta:
        model = get_user_model()
        fields = ['photo', 'username', 'email', 'date_birth', 'first_name', 'last_name']
        labels = {
            'first_name': 'Имя',
            'last_name': 'Фамилия',
        }
        widgets = {
            'first_name': forms.TextInput(attrs={'class': 'form-input'}),
            'last_name': forms.TextInput(attrs={'class': 'form-input'}),
        }
```

```HTML
{% extends 'base.html' %}

{% block content %}
<h1>Профиль</h1>

<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {% if user.photo %}
    <p ><img src="{{ user.photo.url }}" width="100" height="100">
    {% else %}
    <p ><img src="/media/users/default.png" width="100" height="100">
    {% endif %}
    <div class="form-error">{{ form.non_field_errors }}</div>
    {% for f in form %}
    <p><label class="form-label" for="{{ f.id_for_label }}">{{f.label}}: </label>{{ f }}</p>
    <div class="form-error">{{ f.errors }}</div>
    {% endfor %}

    <p><button type="submit">Сохранить</button></p>
    <p ><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
</form>
{% endblock %}
```

*3 и 4 способы я не тестировала, поэтому описания не будет)

<div class="page-break" style="page-break-before: always;"></div>
