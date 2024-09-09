Как в лекции:

Обработка маршрута для contact

Установка и настройка капчи

Добавление поля капчи в форму

Изменение логики:

Логический апгрейд

Обновляем капчу

Отправляем письмо от пользователя на почту

## Как в лекции:

### Обработка маршрута для contact

1. В файле `women/views.py` создайте класс представления для формы обратной связи:
    
    ```Python
    from django.urls import reverse_lazy
    from django.views.generic.edit import FormView
    from django.contrib.auth.mixins import LoginRequiredMixin
    from .forms import ContactForm
    from .utils import DataMixin
    
    class ContactFormView(LoginRequiredMixin, DataMixin, FormView):
        form_class = ContactForm
        template_name = 'women/contact.html'
        success_url = reverse_lazy('home')
        title_page = "Обратная связь"
    
        def form_valid(self, form):
            print(form.cleaned_data)
            return super().form_valid(form)
    ```
    
2. В файле `women/forms.py` создайте класс формы:
    
    ```Python
    from django import forms
    
    class ContactForm(forms.Form):
        name = forms.CharField(label='Имя', max_length=255)
        email = forms.EmailField(label='Email')
        content = forms.CharField(widget=forms.Textarea(attrs={'cols': 60, 'rows': 10}))
    ```
    
3. Создайте шаблон `contact.html` в каталоге `templates/women/`:
    
    ```HTML
    {% extends 'base.html' %}
    
    {% block content %}
    <h1>{{ title }}</h1>
    
    <form method="post">
        {% csrf_token %}
    
        <div class="form-error">{{ form.non_field_errors }}</div>
    
        {% for f in form %}
        <p><label class="form-label" for="{{ f.id_for_label }}">{{ f.label }}: </label>{{ f }}</p>
        <div class="form-error">{{ f.errors }}</div>
        {% endfor %}
    
        <button type="submit">Отправить</button>
    </form>
    
    {% endblock %}
    ```
    
4. В файле `women/urls.py` добавьте маршрут для формы:
    
    ```Python
    from django.urls import path
    from . import views
    
    urlpatterns = [
        # другие маршруты
        path('contact/', views.ContactFormView.as_view(), name='contact'),
    ]
    ```
    

### Установка и настройка капчи

[Документация](https://django-simple-captcha.readthedocs.io/en/latest/index.html#contents).

1. Установите модуль `django-simple-captcha`:
    
    ```Shell
    pip install django-simple-captcha
    ```
    
2. Добавьте `captcha` в список установленных приложений в `settings.py`:
    
    ```Python
    INSTALLED_APPS = [
        # другие приложения
        'captcha',
    ]
    ```
    
3. Выполните миграцию для капчи:
    
    ```Shell
    python manage.py migrate
    ```
    
4. Добавьте маршруты капчи в корневой файл маршрутов проекта (например, `project/urls.py`):
    
    ```Python
    from django.urls import path, include
    
    urlpatterns = [
        # другие маршруты
        path('captcha/', include('captcha.urls')),
    ]
    ```
    

### Добавление поля капчи в форму

Импортируйте класс поля капчи и добавьте его в `ContactForm` в `women/forms.py`:

```Python
from captcha.fields import CaptchaField

class ContactForm(forms.Form):
    name = forms.CharField(label='Имя', max_length=255)
    email = forms.EmailField(label='Email')
    content = forms.CharField(widget=forms.Textarea(attrs={'cols': 60, 'rows': 10}))
    captcha = CaptchaField()
```

## Изменение логики:

### Логический апгрейд

Для меня было странным, что:

1. Мы даём доступ к форме обратной связи только авторизованным пользователям. А что если у пользователя как раз проблема с авторизацией/регистрацией?)
2. Мы показываем капчу авторизованным пользователям (зачем??))
3. Мы не подтягиваем в форму данные о пользователе, то есть в теории он может написать там всё что угодно, не совпадающее с его реальными данными.

Исправила всё вышеперечисленное в коде:

1. В представлении - убираем наследование от `LoginRequiredMixin` и добавляем метод `get_form_kwargs()`, чтобы передавать в форму данные о пользователе.
    
    ```Python
    class ContactFormView(DataMixin, FormView):
        form_class = ContactForm
        template_name = 'basefunc/contact.html'
        success_url = reverse_lazy('home')
        page_title = "Обратная связь"
    
        def get_form_kwargs(self):
            kwargs = super(ContactFormView, self).get_form_kwargs()
            kwargs['user'] = self.request.user
            return kwargs
    ```
    
2. В форме - в методе `__init__` проверяем аутентифицирован ли пользователь и если да - подтягиваем его данные и защищаем поля от редактирования - `readonly` (атрибут `disabled` нам не подойдёт, так как при отправке формы, предзаполненные данные в таких полях не включаются в данные формы и соответственно не отправляются). Также добавляем поле капчи, если пользователь не аутентифицирован.
    
    ```Python
    class ContactForm(forms.Form):
        name = forms.CharField(label='Имя', widget=forms.TextInput(attrs={'class': 'form-input'}))
        email = forms.CharField(required=False, label='E-mail',
                                widget=forms.TextInput(attrs={'class': 'form-input'}))
        content = forms.CharField(label='Комментарий', widget=forms.Textarea(attrs={'cols': 60, 'rows': 10}))
    
        def __init__(self, *args, **kwargs):
            user = kwargs.pop('user', None)
            super(ContactForm, self).__init__(*args, **kwargs)
            if user and user.is_authenticated:
                self.fields['name'].initial = user.username
                self.fields['name'].widget.attrs['readonly'] = True
                self.fields['email'].initial = user.email
                self.fields['email'].widget.attrs['readonly'] = True
            else:
                self.fields['captcha'] = CaptchaField()
    ```
    

### Обновляем капчу

Чтобы добавить возможность обновлять капчу по кнопке, правим шаблон. Добавляем в него условие для проверки, аутентифицирован пользователь или нет (помним, что для таких пользователей капчи в принципе не будет)). Также добавляем js скрипт для непосредственно обновления:

```Python
{% extends 'base.html' %}

{% block content %}
<h1>{{ title }}</h1>

<form method="post">
    {% csrf_token %}

    <div class="form-error">{{ form.non_field_errors }}</div>

    {% for f in form %}
    <p><label class="form-label" for="{{ f.id_for_label }}">{{ f.label }}: </label>{{ f }}</p>
    <div class="form-error">{{ f.errors }}</div>
    {% endfor %}
    {% if not user.is_authenticated %}
    <button class="js-captcha-refresh" type="button">Refresh captcha</button>
    {% endif %}
    <p><button type="submit">Отправить</button></p>
</form>
<script src="https://code.jquery.com/jquery-3.6.4.min.js"></script>
<script>
$('.js-captcha-refresh').click(function(){
    $form = $(this).parents('form');
    $.getJSON("/captcha/refresh/", function (result) {
        $('.captcha').attr('src', result['image_url']);
        $('\#id_captcha_0').val(result['key'])
    });
    return false;
});
</script>
{% endblock %}
```

### Отправляем письмо от пользователя на почту

И, наконец, финальный штрих - реализация непосредственно самой отправки письма) Используем класс EmailMessage в классе представлении:

```Python
from django.core.mail import EmailMessage


class ContactFormView(DataMixin, FormView):
    form_class = ContactForm
    template_name = 'basefunc/contact.html'
    success_url = reverse_lazy('home')
    page_title = "Обратная связь"

    def get_form_kwargs(self):
        kwargs = super(ContactFormView, self).get_form_kwargs()
        kwargs['user'] = self.request.user
        return kwargs

    def form_valid(self, form):
        # Получение данных формы
        name = form.cleaned_data['name']
        email = form.cleaned_data['email']
        content = form.cleaned_data['content']

        # Формирование данных для письма
        subject = f'Обратная связь от {name}'
        message = f'Сообщение от {name} ({email}):\n\n{content}'
        from_email = settings.EMAIL_HOST_USER
        recipient_list = [settings.EMAIL_HOST_USER]

        # Использование EmailMessage для установки reply_to
        email_message = EmailMessage(
            subject,
            message,
            from_email,
            recipient_list,
            reply_to=[email]  # Адрес электронной почты пользователя в качестве адреса для ответа
        )

        email_message.send()
        return super().form_valid(form)
```

<div class="page-break" style="page-break-before: always;"></div>
