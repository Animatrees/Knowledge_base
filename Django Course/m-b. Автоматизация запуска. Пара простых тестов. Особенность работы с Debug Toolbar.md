Автоматизация

Пишем тесты

Особенность работы с Debug Toolbar

### Автоматизация

1. Заходим в редактирование конфигураций запуска и для `sitewomen` создаём копию.
2. Переименуем - `test_get_pages`.
3. В параметрах вместо `runserver` пишем:
    
    ```Shell
    test women.tests.GetPagesTestCase
    ```
    

Теперь можно быстро запускать группу тестов класса `GetPagesTestCase`, в том числе и в режиме отладки.

### Пишем тесты

Для проверки получения страниц сайта создаем класс `GetPagesTestCase`, который наследуется от `TestCase`.

- Первый тест - `test_mainpage`, проверяет успешное возвращение главной страницы:
    
    ```Python
    class GetPagesTestCase(TestCase):
        def test_mainpage(self):
            path = reverse('home')
            response = self.client.get(path)
            self.assertEqual(response.status_code, HTTPStatus.OK)
            self.assertTemplateUsed(response, 'women/index.html')
            self.assertEqual(response.context_data['title'], 'Главная страница')
    ```
    
    1. Получает URL-адрес главной страницы с помощью `reverse('home')`.
    2. Выполняет GET-запрос на этот URL-адрес через `self.client.get(path)`.
    3. Проверяет, что код состояния ответа равен 200 (`HTTPStatus.OK`) с помощью `self.assertEqual(response.status_code, HTTPStatus.OK)`.
    4. Проверяет, что используется правильный шаблон 'women/index.html' с помощью `self.assertTemplateUsed(response, 'women/index.html')`.
    5. Проверяет, что значение 'title' в context_data соответствует 'Главная страница' с помощью `self.assertEqual(response.context_data['title'], 'Главная страница')`.
- Дополнительный тест - `test_redirect_addpage`, (в том же классе GetPagesTestCase) проверяет перенаправление со страницы /addpage/ на страницу авторизации для неавторизованных пользователей.
    
    ```Python
    def test_redirect_addpage(self):
            path = reverse('add_page')
            redirect_uri = reverse('users:login') + '?next=' + path
            response = self.client.get(path)
            self.assertEqual(response.status_code, HTTPStatus.FOUND)
            self.assertRedirects(response, redirect_uri)
    ```
    
    1. Получает URL-адрес страницы /addpage/ с помощью `reverse('add_page')`.
    2. Формирует ожидаемый URL-адрес перенаправления (redirect_uri) с помощью `reverse('users:login')` и добавления параметра '?next=' с URL-адресом /addpage/.
    3. Выполняет GET-запрос на URL-адрес /addpage/ через `self.client.get(path)`.
    4. Проверяет, что код состояния ответа равен 302 (HTTP_FOUND) с помощью `self.assertEqual(response.status_code, HTTPStatus.FOUND)`.
    5. Проверяет, что перенаправление происходит на правильный URL-адрес (redirect_uri) с помощью `self.assertRedirects(response, redirect_uri)`.

### Особенность работы с Debug Toolbar

Django Debug Toolbar по умолчанию не используется при запуске с `DEBUG=False`. Это значит, что если `DEBUG=False`, то Debug Toolbar просто игнорируется.

Однако, когда запускаются тесты, Debug Toolbar выполняет дополнительную проверку. Если он обнаруживает, что запущены тесты (по наличию 'test' в `sys.argv`), то выбрасывает ошибку о невозможности своей работы при `DEBUG=False`.

За это отвечает следующая настройка из конфигурации Toolbar-a:

`DEBUG_TOOLBAR_CONFIG = { 'IS_RUNNING_TESTS': True }`

**Есть два способа решения проблемы:**

1. Добавляем данную конфигурацию в `settings.py`, но меняем её значение на False
    
    ```Python
    DEBUG_TOOLBAR_CONFIG = {
        'IS_RUNNING_TESTS': False
    }
    ```
    
    **Что это нам даёт?**
    
    Возможность использовать Debug Toolbar во время запуска тестов.  
    Недостаток: Потенциальный риск утечки отладочной информации при запуске тестов в production.  
    
2. Изменяем логику добавления Toolbar в `settings.py` и `urls.py`.
    
    ```Python
    # settings.py
    
    import sys
    
    INSTALLED_APPS = [
        ...
    ]
    
    MIDDLEWARE = [
        ...
    ]
    
    ENABLE_DEBUG_TOOLBAR = DEBUG and "test" not in sys.argv
    if ENABLE_DEBUG_TOOLBAR:
        INSTALLED_APPS += [
            "debug_toolbar",
        ]
        MIDDLEWARE += [
            "debug_toolbar.middleware.DebugToolbarMiddleware",
        ]
        # Customize the config to support turbo and htmx boosting.
        DEBUG_TOOLBAR_CONFIG = {"ROOT_TAG_EXTRA_ATTRS": "data-turbo-permanent hx-preserve"}
        
    # urls.py 
    from django.conf import settings
    
    urlpatterns = [
        ...
    ]
    
    if settings.ENABLE_DEBUG_TOOLBAR:
        urlpatterns += [
            path("__debug__/", include("debug_toolbar.urls")),
        ]
    ```
    
    **Что даёт?**
    
    Гарантированное отсутствие утечки отладочной информации при запуске тестов, а также отсутствие ошибок при запуске тестов)  
    Недостаток: Отсутствие возможности использовать Debug Toolbar при запуске тестов для отладки. Критично для случаев, когда нам нужен toolbar в начале списка, а не в конце.  
    

P.S. В документации советуют использовать второй способ)

<div class="page-break" style="page-break-before: always;"></div>
