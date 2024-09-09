Класс BaseUserCreationForm

Валидация паролей

Класс UserCreationForm

Апргейд регистрации

### Класс BaseUserCreationForm

`BaseUserCreationForm` является базовым классом формы в Django, который служит фундаментом для создания форм регистрации пользователей. Его основное назначение - предоставить разработчикам удобную отправную точку для построения собственных форм создания пользователей. Этот класс инкапсулирует базовую структуру и функциональность, необходимые для ввода данных нового пользователя, таких как имя пользователя, пароль и подтверждение пароля.

**BaseUserCreationForm реализует следующую функциональность:**

- Связывает форму с моделью пользователя (`User`) с помощью наследования от `forms.ModelForm`.
- Определяет поля формы, такие как `username`, `password1` и `password2`, с соответствующими виджетами и метками.
- Выполняет проверку соответствия пароля и его подтверждения в методе `clean_password2()`.
- Валидирует пароль с помощью встроенных валидаторов Django в методе `_post_clean()`. Эта валидация проверяет длину пароля, его распространенность и состоит ли он только из цифр.
- Сохраняет нового пользователя в базе данных с хешированным паролем (использует метод `set_password()`) в методе `save()`.

### Валидация паролей

Вместе с классом BaseUserCreationForm мы получаем довольно жесткую валидацию паролей и возможно в каких-то случаях нам не нужны все эти рамки.

![[Untitled 14.png|Untitled 14.png]]

Но мы можем настроить логику валидации с помощью файла `settings.py` и константы - `AUTH_PASSWORD_VALIDATORS`. По умолчанию она выглядит следующим образом:

```Python
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

1. `UserAttributeSimilarityValidator` - проверяет, насколько пароль отличается от некоторых атрибутов пользователя, таких как имя, фамилия, email и т.д. Это сделано для того, чтобы пароль не был слишком очевидным и легко угадываемым.
2. `MinimumLengthValidator` - проверяет минимальную длину пароля.
3. `CommonPasswordValidator` - проверяет пароль на вхождение в список из 20000 самых распространенных паролей. Это помогает избежать слабых и легко подбираемых паролей.
4. `NumericPasswordValidator` - проверяет, что пароль состоит не только из цифр.

Валидаторы применяются в том порядке, в котором они перечислены в `AUTH_PASSWORD_VALIDATORS`. Все сообщения об ошибках и подсказки также выводятся в этом порядке.

При необходимости можно кастомизировать параметры валидаторов. Например, для `MinimumLengthValidator` можно задать свою минимальную длину:

```Python
{
        "NAME": "django.contrib.auth.password_validation.MinimumLengthValidator",
        "OPTIONS": {
            "min_length": 9,
        },
```

Для `UserAttributeSimilarityValidator` - список атрибутов пользователя для сравнения и максимальную степень сходства пароля с ними.

`CommonPasswordValidator` позволяет использовать свой файл со списком частых паролей.

### Класс UserCreationForm

Это реализация формы создания пользователя, основанная на классе BaseUserCreationForm.

Помимо того, что уже умеет делать его родительский класс, добавляется только проверка имени пользователя на уникальность, с помощью определения метода `clean_username`. Чтобы избежать путаницы с похожими именами, форма не допускает имен, отличающихся только регистром.

### Апргейд регистрации

Используя класс UserCreationForm, мы можем улучшить наш код класса формы для регистрации нового пользователя:

```Python
class RegisterUserForm(UserCreationForm):
    password1 = forms.CharField(
        label='*Пароль',
        widget=forms.PasswordInput(attrs={"autocomplete": "new-password",
                                          'class': 'form-input'}),
    )
    password2 = forms.CharField(
        label='*Повторите пароль',
        widget=forms.PasswordInput(attrs={"autocomplete": "new-password",
                                          'class': 'form-input'}),
    )

    class Meta:
        model = get_user_model()
        fields = ('username', 'first_name', 'last_name', 'email', 'password1', 'password2')
        labels = {
            'username': '*Логин',
            'email': 'E-mail',
            'first_name': 'Имя',
            'last_name': 'Фамилия'
        }
        widgets = {
            'username': forms.TextInput(attrs={'class': 'form-input'}),
            'email': forms.EmailInput(attrs={'class': 'form-input'}),
            'first_name': forms.TextInput(attrs={'class': 'form-input'}),
            'last_name': forms.TextInput(attrs={'class': 'form-input'}),
        }
```

Теперь нам не нужно в форме прописывать метод для проверки совпадения паролей.

Также мы можем улучшить шаблон для отображения формы, так как в виджетах формы мы указали классы для них)

```HTML
<form method="post">
    {% csrf_token %}
    <div class="form-error">{{ form.non_field_errors }}</div>
    {% for f in form %}
    <p ><label class="form-label" for="{{ f.id_for_label }}">{{f.label}}: </label>{{ f }}</p>
    <div class="form-error">{{ f.errors }}</div>
    {% endfor %}
    <p>Поля со * обязательны для заполнения!</p>
    <p><button type="submit" class="form-button">Зарегистрироваться</button></p>
</form>
```

И изменить функцию-представления на класс, так как логика хэширования пароля и сохранения пользователя в базу данных уже реализована в базовом классе формы BaseUserCreationForm!

Для этого мы можем использовать наследование от CreateView. [[i-f. Класс CreateView (+ ModelFormMixin, BaseCreateView)]] - это generic класс представления, который предоставляет функциональность для создания новых объектов, включая пользователей.

```Python
class UserRegistrationView(DataMixin, CreateView):
    form_class = RegisterUserForm
    template_name = 'users/register.html'
    page_title = 'Регистрация'
    success_url = reverse_lazy('user:login')
```

Использую данный класс мы получаем практически ту же функциональность, что у нас была ранее, кроме перенаправления на шаблон `register_done.html`, чтобы её добавить, нам придётся либо дополнительно определять этот маршрут и создавать для него класс представление, либо переопределять метод - `form_valid` в классе.

Не забываем изменить представление в маршруте:

```Python
urlpatterns = [
    path('login/', views.LoginUser.as_view(), name='login'),
    path('logout/', LogoutView.as_view(next_page='users:login'), name='logout'),
    path('register/', views.UserRegistrationView.as_view(), name='register'),
]
```

<div class="page-break" style="page-break-before: always;"></div>
