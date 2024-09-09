Тест отображения формы

Тест успешной регистрации пользователя

Тест ошибки при несовпадении паролей

Тест ошибки при существующем пользователе

Использование метода setUp() для инициализации данных

Все тесты данного раздела пишутся в файле `users/tests.py`

### Тест отображения формы

```Python
class RegisterUserTestCase(TestCase):
    def test_form_registration_get(self):
        path = reverse('users:register')
        response = self.client.get(path)
        self.assertEqual(response.status_code, HTTPStatus.OK)
        self.assertTemplateUsed(response, 'users/register.html')
```

Этот тест проверяет, что страница регистрации доступна, возвращает корректный статус-код и использует ожидаемый шаблон при GET-запросе.

### Тест успешной регистрации пользователя

```Python
def test_user_registration_success(self):
    data = {
        'username': 'user_1',
        'email': 'user1@sitewomen.ru',
        'first_name': 'Sergey',
        'last_name': 'Balakirev',
        'password1': '12345678Aa',
        'password2': '12345678Aa',
    }
    user_model = get_user_model()
    path = reverse('users:register')
    response = self.client.post(path, data)
    self.assertEqual(response.status_code, HTTPStatus.FOUND)
    self.assertRedirects(response, reverse('users:login'))
    self.assertTrue(user_model.objects.filter(username=data['username']).exists())
```

Этот тест проверяет, что:

- Регистрация с валидными данными проходит успешно
- После регистрации происходит перенаправление на страницу входа
- Новый пользователь действительно создается в базе данных

### Тест ошибки при несовпадении паролей

```Python
def test_user_registration_password_error(self):
    data = {
        'username': 'user_1',
        'email': 'user1@sitewomen.ru',
        'first_name': 'Sergey',
        'last_name': 'Balakirev',
        'password1': '12345678Aa',
        'password2': '12345678A',
    }
    path = reverse('users:register')
    response = self.client.post(path, data)
    self.assertEqual(response.status_code, HTTPStatus.OK)
    self.assertContains(response, "Введенные пароли не совпадают.")
```

Этот тест проверяет, что:

- При попытке регистрации с несовпадающими паролями, регистрация не происходит
- Сервер возвращает код 200 (OK), а не перенаправляет на страницу входа, как в предыдущем случае
- На странице отображается сообщение об ошибке "Введенные пароли не совпадают."

### Тест ошибки при существующем пользователе

```Python
def test_user_registration_user_exists_error(self):
	data = {
	        'username': 'user_1',
	        'email': 'user1@sitewomen.ru',
	        'first_name': 'Sergey',
	        'last_name': 'Balakirev',
	        'password1': '12345678Aa',
	        'password2': '12345678Aa',
	    }
    user_model = get_user_model()
    user_model.objects.create(username=data['username'])
    path = reverse('users:register')
    response = self.client.post(path, data)
    self.assertEqual(response.status_code, HTTPStatus.OK)
    self.assertContains(response, "Пользователь с таким именем уже существует.")
```

Этот тест проверяет, что:

- При попытке регистрации с уже существующим именем пользователя, регистрация не будет выполнена (предварительно созданный пользователь с таким же именем не перезаписывается)
- Сервер возвращает код 200 (OK), а не перенаправляет на страницу входа
- На странице отображается сообщение об ошибке "Пользователь с таким именем уже существует."

### Использование метода setUp() для инициализации данных

```Python
def setUp(self):
    self.data = {
        'username': 'user_1',
        'email': 'user1@sitewomen.ru',
        'first_name': 'Sergey',
        'last_name': 'Balakirev',
        'password1': '12345678Aa',
        'password2': '12345678Aa',
    }
```

Для того, чтобы не дублировать во всех тестирующих методах словарь с данными пользователя, можно использовать метод `setUp()`. Уже писала ранее про логику его работы, но продублирую. Метод вызывается перед запуском каждого теста, то есть перед каждым тестом словарь с данными будет проинициализирован заново, что даёт нам возможность изменять его содержимое как угодно в каждом тесте, при этом не влияя на остальные тесты)

<div class="page-break" style="page-break-before: always;"></div>
