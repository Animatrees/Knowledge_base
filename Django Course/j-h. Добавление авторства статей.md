1. В модели Women добавить поле author со связью ForeignKey к модели User:
    
    ```Python
    author = models.ForeignKey(get_user_model(), on_delete=models.SET_NULL, related_name='posts', null=True, default=None)
    ```
    
2. Создать миграции для обновления структуры базы данных:
    
    ```Shell
    python manage.py makemigrations
    ```
    
3. Применить миграции для внесения изменений в базу данных:
    
    ```Shell
    python manage.py migrate
    ```
    
4. В классе AddPage добавить метод form_valid() для сохранения автора статьи:
    
    ```Python
    def form_valid(self, form):
        w = form.save(commit=False)
        w.author = self.request.user
        return super().form_valid(form)
    ```
    
5. Изменить строку отображения категории и автора статьи:
    
    ```HTML
    <p class="first">Категория: {{p.cat.name}} | автор: {{p.author.username|default:"неизвестен"}}</p>
    ```
    
6. Сохранить все изменения.
7. Перезапустить сервер разработки Django, если он уже запущен.

<div class="page-break" style="page-break-before: always;"></div>
