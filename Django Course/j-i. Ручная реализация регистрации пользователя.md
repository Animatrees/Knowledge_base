Для того, чтобы реализовать функционал регистрации пользователя на нашем сайте, нужно выполнить следующие шаги:

1. Добавляем маршрут для регистрации:
    
    ```Python
    from django.urls import path
    from . import views
    from django.contrib.auth.views import LogoutView
    
    app_name = 'users'
    
    urlpatterns = [
        path('login/', views.LoginUser.as_view(), name='login'),
        path('logout/', LogoutView.as_view(next_page='users:login'), name='logout'),
        path('register/', views.register, name='register'),
    ]
    ```
    
2. Создаём функцию представление (пока что в виде заглушки):
    
    ```Python
    def register(request):
        form = RegisterUserForm()
        return render(request, 'users/register.html', {'title': 'Регистрация', 'form': form})
    ```
    
3. Следующим шагом создаём класс для формы и импортируем её в предыдущем файле:
    
    ```Python
    class RegisterUserForm(forms.ModelForm):
        username = forms.CharField(max_length=100, label='*Логин')
        password = forms.CharField(max_length=30, label='*Пароль', widget=forms.PasswordInput())
        password2 = forms.CharField(max_length=30, label='*Повторите пароль', widget=forms.PasswordInput())
    
        class Meta:
            model = get_user_model()
            fields = ('username', 'first_name', 'last_name', 'email', 'password', 'password2')
            labels = {
                'email': 'E-mail',
                'first_name': 'Имя',
                'last_name': 'Фамилия'
            }
    ```
    
    **Важные уточнения:**
    
    1. Так как наша форма должна быть связана с моделью пользователя, мы наследуемся от `ModelForm`, но саму модель указываем через функцию `get_user_model()`, про которую было рассказано [[j-f. Класс AuthenticationForm и get_user_model()]].
    2. В модели `User`, которую мы используем по умолчанию, существуют (а следовательно могут быть использованы в форме), следующие поля:
        
        - username - имя пользователя (обязательное)
        - first_name - имя (необязательное)
        - last_name - фамилия (необязательное)
        - email - адрес электронной почты (необязательное)
        - password - пароль (обязательное)
        - groups - группы, к которым принадлежит пользователь (необязательное, связь m2m)
        - user_permissions - права доступа пользователя (необязательное, связь m2m)
        - is_staff - флаг, указывающий, является ли пользователь сотрудником (необязательное, по умолчанию False)
        - is_active - флаг, указывающий, активен ли пользователь (необязательное, по умолчанию True)
        - is_superuser - флаг, указывающий, является ли пользователь суперпользователем (необязательное, по умолчанию False)
        - last_login - дата и время последнего входа пользователя (необязательное, обновляется автоматически при авторизации пользователя в системе, по умолчанию Null)
        - date_joined - дата и время регистрации пользователя (необязательное, создаётся автоматически, в момент создания пользователя)
        
        Обратите внимание, что поля username и password являются обязательными для заполнения при создании пользователя. Остальные поля являются необязательными и могут быть заполнены по желанию.
        
    3. Поле `password2` не является обязательным, но считается общепринятой практикой, для того, чтобы пользователь мог выявить ошибку в пароле, если он ввёл разные значения. Этот функционал будет реализован далее.
4. Создаём шаблон, для отображения формы - `register.html`
    
    ```HTML
    {% extends 'base.html' %}
    
    {% block content %}
    <h1>{{ title }}</h1>
    
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <p>Поля со * обязательные!</p>
        <p><button type="submit">Зарегистрироваться</button></p>
    </form>
    
    {% endblock %}
    ```
    
5. Не забываем добавить путь в базовом шаблоне:
    
    ```HTML
    <a href="{% url 'users:register' %}">Регистрация</a>
    ```
    

Мы получили данную форму, но пока что не происходит проверка пароля и сохранение пользователя в базе данных.

![[Untitled 13.png|Untitled 13.png]]

1. Добавляем функционал сохранения пользователя в Базе данных:
    
    ```Python
    def register(request):
        if request.method == 'POST':
            form = RegisterUserForm(request.POST)
            if form.is_valid():
                user = form.save(commit=False)
                user.set_password(form.cleaned_data['password'])
                user.save()
                return render(request, 'users/register_done.html', {'title': 'Успешная регистрация!'})
        else:
            form = RegisterUserForm()
        return render(request, 'users/register.html', {'title': 'Регистрация', 'form': form})
    ```
    
    **Важные уточнения:**
    
    - На этом шаге не происходит проверка совпадения паролей (мы ещё не реализовывали данный функционал).
    - Если при создании пользователя не использовать функцию - `set_password`, то Django сохранит пароль в базе данных в нехэшированном виде, но по этим учётным данным будет невозможно авторизоваться из-за того, как реализована проверка пароля в Django при аутентификации. Поэтому нам необходимо добавить вызов этого метода перед сохранением пользователя в БД.
    
    - Подробнее про `set_password`
        
        `set_password()` - это метод модели User в Django, который используется для установки пароля пользователя. Вот как он работает:
        
        1. Когда вызывается метод `set_password(raw_password)`, он принимает один аргумент `raw_password`, который представляет собой пароль в виде необработанной строки.
        2. Внутри метода `set_password()` происходит следующее:
            - Генерируется случайная соль (`salt`) для хеширования пароля. Соль - это дополнительная случайная строка, которая добавляется к паролю перед хешированием для усиления безопасности.
            - Пароль (`raw_password`) вместе со сгенерированной солью передается в функцию хеширования паролей. По умолчанию Django использует алгоритм PBKDF2 с хэшем SHA256 - механизм растягивания паролей, рекомендованный NIST. Этого должно быть достаточно для большинства пользователей: он довольно безопасен и требует огромного количества вычислительного времени для взлома.
            - Результат хеширования сохраняется в поле password модели User в виде хешированного значения пароля.
        3. После вызова set_password() хешированное значение пароля сохраняется в поле password, а исходный пароль (raw_password) не сохраняется в явном виде.
        4. Когда пользователь пытается войти в систему и вводит свой пароль, введенный пароль снова хешируется с использованием той же соли и алгоритма хеширования, и полученное значение сравнивается с сохраненным хешированным значением пароля в базе данных. Если они совпадают, аутентификация пользователя проходит успешно.
    
    - После регистрации есть несколько вариантов куда можно перенаправить пользователя, в нашем случае реализован вариант с отображением страницы, подтвержадющей успешную регистрацию.
2. Создадим шаблон для этой страницы:
    
    ```HTML
    {% extends 'base.html' %}
    
    {% block content %}
    <h1>{{ title }}</h1>
    <p>Для входа в систему необходимо авторизоваться <a href="{% url 'users:login' %}">здесь</a>.
    </p>
    
    {% endblock %}
    ```
    
3. И наконец, реализуем проверку совпадения паролей в классе формы:
    
    ```Python
    class RegisterUserForm(forms.ModelForm):
        username = forms.CharField(max_length=100, label='*Логин')
        password = forms.CharField(max_length=30, label='*Пароль', widget=forms.PasswordInput())
        password2 = forms.CharField(max_length=30, label='*Повторите пароль', widget=forms.PasswordInput())
    
        class Meta:
            model = get_user_model()
            fields = ('username', 'first_name', 'last_name', 'email', 'password', 'password2')
            labels = {
                'email': 'E-mail',
                'first_name': 'Имя',
                'last_name': 'Фамилия'
            }
    
        def clean_password2(self):
            password = self.cleaned_data['password']
            password2 = self.cleaned_data['password2']
            if password != password2:
                raise forms.ValidationError('Пароли не совпадают!')
            return password2
    ```
    
    > [!important]  
    > Обратите внимание, что используется метод clean для password2, так как если попытаться использовать password, то в этот момент password2 ещё не существует в cleaned_data. Также важно учитывать, что при добавлении любых проверок для необязательных полей, в первую очередь нужно убедиться, что само значение было передано) Иначе мы получим ошибку, если пользователь не введёт значение для этого поля.
    
<div class="page-break" style="page-break-before: always;"></div>
