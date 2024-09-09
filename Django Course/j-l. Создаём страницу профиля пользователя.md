1. Создаем шаблон профиля пользователя в файле `users/profile.html`:
    
    ```HTML
    {% extends 'base.html' %}
    
    {% block content %}
    <h1>Профиль</h1>
    
    <form method="post">
        {% csrf_token %}
        <div class="form-error">{{ form.non_field_errors }}</div>
        {% for f in form %}
        <p><label class="form-label" for="{{ f.id_for_label }}">{{f.label}}: </label>{{ f }}</p>
        <div class="form-error">{{ f.errors }}</div>
        {% endfor %}
    
        <p><button type="submit">Сохранить</button></p>
    </form>
    
    {% endblock %}
    ```
    
2. Создаем класс формы `ProfileUserForm` в файле `users/forms.py`:
    
    ```Python
    class ProfileUserForm(forms.ModelForm):
        username = forms.CharField(disabled=True, label='Логин', widget=forms.TextInput(attrs={'class': 'form-input'}))
        email = forms.CharField(disabled=True, label='E-mail', widget=forms.TextInput(attrs={'class': 'form-input'}))
    
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
    ```
    
    У полей `username` и `email` указываем атрибут `disabled=True`, чтобы сделать их недоступными для редактирования.
    
3. Создаем класс представления `ProfileUser` в файле `users/view.py`:
    
    ```Python
    class ProfileUser(LoginRequiredMixin, UpdateView):
        form_class = ProfileUserForm
        template_name = 'users/profile.html'
        extra_context = {'title': "Профиль пользователя"}
    
        def get_success_url(self):
            return reverse_lazy('users:profile', args=[self.request.user.pk])
    ```
    
4. Связываем класс ProfileUser с маршрутом в файле users/urls.py:
    
    ```Python
    path('profile/<int:pk>/', views.ProfileUser.as_view(), name='profile'),
    ```
    
    На этом шаге мы можем увидеть, что страница пользователя работает, но с такой реализацией мы получаем серьёзную проблему с безопасностью. Так как любой авторизованный пользователь может изменить `id` в url-маршруте и получить доступ к чужому профилю пользователя.
    
    Чтобы это исправить:
    
5. Убираем отбор записей из таблицы `user` по идентификатору в файле `users/urls.py`:
    
    ```Python
    path('profile/', views.ProfileUser.as_view(), name='profile'),
    ```
    
6. Добавляем метод `get_object()` в класс `ProfileUser` в файле `users/view.py` и не забываем обновить метод `get_success_url`:
    
    ```Python
    
    def get_success_url(self):
            return reverse_lazy('users:profile')
    
    def get_object(self, queryset=None):
        return self.request.user
    ```
    
7. Добавляем ссылку на профиль в главное меню в шаблоне base.html:
    
    ```HTML
    <li class="last"> <a href="{% url 'users:profile' %}">{{user.username}}</a> | <a href="{% url 'users:logout' %}">Выйти</a></li>
    ```

<div class="page-break" style="page-break-before: always;"></div>
