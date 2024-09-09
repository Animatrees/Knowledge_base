1. Создаём форму авторизации. Для этого в приложении `users` создаем файл `forms.py` и объявляем в нем класс `LoginUserForm`, наследуемый от `forms.Form`. В классе определяем поля `username` и `password` для ввода логина и пароля соответственно:
    
    ```Python
    from django import forms
    
    class LoginUserForm(forms.Form):
        username = forms.CharField(label='Логин', widget=forms.TextInput(attrs={'class': 'form-input'}))
        password = forms.CharField(label='Пароль', widget=forms.PasswordInput(attrs={'class': 'form-input'}))
    ```
    
2. Создаём шаблон для отображения формы авторизации. В приложении `users` создаем подкаталог `templates`, а в нем подкаталог `users`. В подкаталоге `users` создаем файл `login.html` со следующим содержимым:
    
    ```HTML
    {% extends 'base.html' %}
    
    {% block content %}
    <h1>Авторизация</h1>
    
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <p><button type="submit">Войти</button></p>
    </form>
    
    {% endblock %}
    ```
    
3. Реализуем функцию представления для авторизации пользователя. В файле `users/views.py` создаем функцию `login_user()`:
    
    ```Python
    from django.contrib.auth import authenticate, login
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    	from .forms import LoginUserForm
    
    def login_user(request):
        if request.method == 'POST':
            form = LoginUserForm(request.POST)
            if form.is_valid():
                cd = form.cleaned_data
                user = authenticate(request, username=cd['username'], password=cd['password'])
                if user and user.is_active:
                    login(request, user)
                    return HttpResponseRedirect(reverse('home'))
        else:
            form = LoginUserForm()
        return render(request, 'users/login.html', {'form': form})
    ```
    
4. Реализуем функцию представления для выхода пользователя из системы. В файле `users/views.py` создаем функцию `logout_user()`:
    
    ```Python
    from django.contrib.auth import logout
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    
    def logout_user(request):
        logout(request)
        return HttpResponseRedirect(reverse('users:login'))
    ```
    
5. Функционал авторизации доступен по следующему адресу: `/users/login`, а функционал выхода из системы: `/users/logout`.

<div class="page-break" style="page-break-before: always;"></div>
