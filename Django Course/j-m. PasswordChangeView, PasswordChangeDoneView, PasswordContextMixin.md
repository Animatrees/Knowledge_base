Класс PasswordChangeView

Класс PasswordChangeDoneView

PasswordContextMixin

Классы форм - SetPasswordForm и PasswordChangeForm

### Класс PasswordChangeView

**PasswordChangeView** - это класс представления, который позволяет авторизованным пользователям изменять свой пароль. Он инкапсулирует в себе логику для отображения формы смены пароля и ее обработки при отправке.

**Основные преимущества использования PasswordChangeView:**

1. **Готовая функциональность "из коробки"**  
    PasswordChangeView предоставляет готовую страницу смены пароля без необходимости писать код для обработки формы и сохранения нового пароля. Достаточно указать путь к нему в  
    `urls.py` и предоставить шаблон.
2. **Легкая настройка через атрибуты класса**  
    Поведение PasswordChangeView можно настроить с помощью атрибутов, таких как  
    `template_name` для указания пользовательского шаблона, `success_url` для задания URL редиректа после успешной смены пароля, `form_class` для использования своей формы и `extra_context` для передачи дополнительных данных в контекст шаблона.
3. **Обеспечение безопасности**  
    PasswordChangeView уже включает в себя необходимые меры безопасности, такие как декораторы  
    `login_required` (только авторизованные пользователи могут менять пароль), `sensitive_post_parameters` (пароли не сохраняются в истории браузера) и `csrf_protect` (защита от CSRF атак).
4. **Расширяемость через наследование**  
    При необходимости дополнительной функциональности, можно унаследоваться от PasswordChangeView и переопределить необходимые методы, такие как  
    `get_form_kwargs` для передачи дополнительных аргументов в форму или `form_valid` для выполнения дополнительных действий при успешном сохранении формы.

**Пример использования**

Чтобы задействовать PasswordChangeView в проекте достаточно добавить пути в `urls.py`:

```Python
from django.contrib.auth.views import PasswordChangeView, PasswordChangeDoneView

urlpatterns = [
    path('password_change/', PasswordChangeView.as_view(), name='password_change'),
    path('password_change/done/', PasswordChangeDoneView.as_view(), name='password_change_done'),
    ...
]
```

И добавить ссылку в шаблон профиля пользователя:

```HTML
<hr>
<p ><a href="{% url 'users:password_change' %}">Сменить пароль</a></p>
```

Но в таком случае для смены пароля будет использоваться страница из админки, а также страница для отображения успешной смены пароля.

**Переменные контекста шаблона**

- `form` - форма смены пароля (по умолчанию `PasswordChangeForm`)
- `title` - заголовок страницы (по умолчанию "Password change")
- Любые дополнительные переменные, указанные в `extra_context`

**Наследование**

`PasswordChangeView` наследуется от двух классов:

- `PasswordContextMixin` - миксин, добавляющий дополнительные данные в контекст шаблона
- [[i-e. Класс FormView (+ FormMixin, ProcessFormView, BaseFormView)]] - базовый класс для обработки форм в Django.

**Атрибуты класса и их значения по умолчанию:**

1. `form_class = PasswordChangeForm`  
    Указывает класс формы, которая будет использоваться для смены пароля. По умолчанию это  
    `PasswordChangeForm` из `django.contrib.auth.forms`. Эта форма включает поля для ввода текущего пароля, нового пароля и его подтверждения.
2. `success_url = reverse_lazy("password_change_done")`  
    URL, на который будет перенаправлен пользователь после успешной смены пароля. Здесь используется  
    `reverse_lazy()` вместо `reverse()`, потому что URL-шаблоны могут быть еще не загружены на момент определения атрибутов класса.
3. `template_name = "registration/password_change_form.html"`  
    Путь к шаблону страницы смены пароля. Этот шаблон будет использоваться для рендеринга формы смены пароля.  
    
4. `title = _("Password change")`  
    Заголовок страницы смены пароля. Используется функция перевода  
    `_()` для возможности локализации.

**Методы класса:**

1. `dispatch(self, *args, **kwargs)`  
    Метод  
    `dispatch` вызывается перед любым другим методом обработки запроса (`get`, `post` и т.д.). Здесь он декорирован тремя декораторами:  
      
    После выполнения декораторов вызывается метод  
    `dispatch` базового класса (`super().dispatch(*args, **kwargs)`).
    - `@method_decorator(login_required)` - требует, чтобы пользователь был залогинен для доступа к этой странице.
    - `@method_decorator(csrf_protect)` - добавляет защиту от CSRF-атак.
    - `@method_decorator(sensitive_post_parameters())` - указывает, что POST-параметры содержат чувствительные данные (пароль), чтобы они не сохранялись в истории браузера.
2. `get_form_kwargs(self)`  
    Этот метод возвращает словарь аргументов, которые будут переданы в конструктор класса формы при её создании.  
    Сначала получаем базовый словарь аргументов от родительского класса (  
    `super().get_form_kwargs()`). Затем добавляем в словарь текущего пользователя (`self.request.user`) под ключом `"user"`. Это нужно, так как `PasswordChangeForm` требует аргумент `user` для проверки текущего пароля и сохранения нового.
3. `form_valid(self, form)`  
    Этот метод вызывается, когда форма прошла валидацию при отправке (метод  
    `POST`).  
    Сначала вызываем  
    `form.save()`, чтобы сохранить новый пароль. Затем вызываем `update_session_auth_hash(self.request, form.user)`, чтобы обновить хеш аутентификации в сессии. Это нужно, чтобы все остальные сессии пользователя стали недействительными (т.е. требовали повторного логина с новым паролем). Текущая сессия продолжит работать.  
    Наконец, возвращаем результат вызова  
    `form_valid` базового класса (`super().form_valid(form)`), который, как правило, делает редирект на `success_url`.

### Класс PasswordChangeDoneView

**PasswordChangeDoneView** - это класс представления в Django, который отображает страницу с сообщением об успешном изменении пароля пользователя. Он используется в связке с `PasswordChangeView` и показывается после того, как пользователь успешно сменил свой пароль.

**Основные преимущества использования PasswordChangeDoneView:**

1. **Готовая страница подтверждения смены пароля**  
    PasswordChangeDoneView предоставляет готовую страницу с сообщением об успешной смене пароля без необходимости создавать отдельный шаблон и view-функцию для этой цели.  
    
2. **Настройка через атрибуты класса**  
    Поведение PasswordChangeDoneView можно настроить с помощью атрибутов, таких как  
    `template_name` для указания своего шаблона и `extra_context` для передачи дополнительных данных в контекст шаблона.
3. **Защита доступа**  
    Класс использует декоратор  
    `login_required`, чтобы ограничить доступ к странице только для авторизованных пользователей.

**Переменные контекста шаблона:**

- `title` - заголовок страницы (по умолчанию "Password change successful")
- Любые дополнительные переменные, указанные в `extra_context`

**Наследование:**

`PasswordChangeDoneView` наследуется от двух классов:

- `PasswordContextMixin` - миксин, добавляющий дополнительные данные в контекст шаблона
- [[i-b. Классы TemplateView, ContextMixin, TemplateResponseMixin]] - базовый класс для отображения шаблона в Django.

**Атрибуты класса и их значения по умолчанию:**

1. `template_name = "registration/password_change_done.html"`  
    Путь к шаблону страницы подтверждения смены пароля.  
    
2. `title = _("Password change successful")`  
    Заголовок страницы. Используется функция перевода  
    `_()` для возможности локализации.

**Методы класса:**

1. `dispatch(self, *args, **kwargs)`  
    Метод  
    `dispatch` вызывается перед методом `get` для обработки запроса. Он декорирован `@method_decorator(login_required)`, который требует, чтобы пользователь был залогинен для доступа к этой странице. После выполнения декоратора вызывается метод `dispatch` базового класса (`super().dispatch(*args, **kwargs)`).

### PasswordContextMixin

`PasswordContextMixin` - это миксин, который добавляет дополнительные данные в контекст шаблона для представлений, связанных с операциями над паролем пользователя. Он используется такими представлениями, как `PasswordChangeView`, `PasswordResetView`, `PasswordResetConfirmView` и другими.

Задача этого миксина - предоставить единый способ указания заголовка страницы (`title`), подзаголовка (`subtitle`) и любых дополнительных переменных (`extra_context`) для шаблонов страниц, связанных с паролями.

**Реализует всего один метод и определяет один атрибут:**

- `get_context_data(**kwargs)`  
    Переопределяет метод get_context_data(), добавляя в контекст шаблона следующие переменные:  
    
    `title` - заголовок страницы. Берется из атрибута title класса представления.  
      
    `subtitle` - подзаголовок страницы. Всегда равен None, так как ни одно из стандартных представлений для работы с паролями не задает атрибут subtitle.  
    Все переменные из словаря extra_context, если он не None.  
    
- Атрибут `extra_context` - по умолчанию None.

### Классы форм - `SetPasswordForm` и `PasswordChangeForm`

`SetPasswordForm` и `PasswordChangeForm` - это два класса форм Django, которые используются для смены пароля пользователя. Они отличаются тем, что `SetPasswordForm` позволяет установить новый пароль без ввода старого, а `PasswordChangeForm` требует ввода текущего пароля для подтверждения личности пользователя.

**SetPasswordForm:**

1. Форма имеет два поля: `new_password1` и `new_password2` для ввода нового пароля и его подтверждения.
2. Конструктор формы (`__init__`) принимает объект пользователя (`user`), для которого будет установлен новый пароль.
3. Метод `clean_new_password2` производит валидацию введенных паролей:
    - Проверяет, что оба поля заполнены и их значения совпадают. Если нет, выбрасывает `ValidationError` с сообщением об ошибке.
    - Вызывает `password_validation.validate_password` для проверки нового пароля на соответствие настройкам валидации паролей в Django.
4. Метод `save` устанавливает новый пароль для пользователя, вызывая `user.set_password(password)`, и сохраняет пользователя, если `commit=True`.

**PasswordChangeForm:**

1. Наследуется от `SetPasswordForm` и добавляет поле `old_password` для ввода текущего пароля.
2. Переопределяет сообщения об ошибках (`error_messages`), добавляя сообщение `password_incorrect` на случай, если введенный текущий пароль неверен.
3. Изменяет порядок полей формы с помощью атрибута `field_order`, чтобы поле `old_password` было первым.
4. Добавляет метод `clean_old_password` для валидации введенного текущего пароля:
    - Проверяет, что введенный пароль соответствует текущему паролю пользователя, вызывая `user.check_password(old_password)`. Если нет, выбрасывает `ValidationError` с сообщением об ошибке.

**Использование форм:**

```Python
# Пример использования SetPasswordForm
form = SetPasswordForm(user=request.user, data=request.POST)
if form.is_valid():
    form.save()

# Пример использования PasswordChangeForm
form = PasswordChangeForm(user=request.user, data=request.POST)
if form.is_valid():
    form.save()
```

Обе формы принимают объект пользователя (`user`) и данные из запроса (`request.POST`) для валидации и сохранения нового пароля.

Таким образом, `SetPasswordForm` можно использовать в сценариях, где не требуется подтверждение текущего пароля (например, при сбросе пароля по ссылке), а `PasswordChangeForm` - когда пользователь меняет свой пароль самостоятельно через личный кабинет и требуется дополнительная проверка безопасности.

<div class="page-break" style="page-break-before: always;"></div>
