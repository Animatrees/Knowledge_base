Бэкенды аутентификации

authenticate()

login()

logout()

### Бэкенды аутентификации

Django позволяет использовать разные способы аутентификации пользователей, которые называются "бэкендами аутентификации". Бэкенды аутентификации - это просто классы или модули, которые умеют проверять учетные данные пользователя.

Django имеет некоторые встроенные бэкенды, такие как `ModelBackend` (который проверяет учетные данные в базе данных) и `RemoteUserBackend` (который доверяет удаленному веб-серверу для аутентификации пользователя). Вы также можете создавать собственные бэкенды аутентификации, если у вас есть специфические требования.

### `authenticate()`

Функция `authenticate` в Django используется для проверки учетных данных пользователя (обычно имени пользователя и пароля). Она проверяет, являются ли предоставленные учетные данные действительными, и возвращает объект `User`, если аутентификация прошла успешно, или `None`, если учетные данные недействительны.

**Синтаксис:**

```Python
authenticate(request=None, **credentials)
```

- `request`: Необязательный параметр, представляющий объект запроса (request). Он может быть передан в бэкенды аутентификации для дополнительной проверки.
- `*credentials`: Именованные аргументы, представляющие учетные данные пользователя. Обычно это `username` и `password`, но могут быть и другие поля в зависимости от настроек вашего приложения.

Вы можете передавать позиционные аргументы в функцию `authenticate`, но они будут интерпретироваться как значения именованных аргументов (`**credentials`). Например:

```Python
authenticate(request, username, password)
```

В этом случае `request` будет привязан к параметру `request`, а `username` и `password` будут собраны в словарь `credentials` как `credentials['username']` и `credentials['password']` соответственно.

Однако, следует отметить, что передача позиционных аргументов в этом случае может быть менее читаемой и более подверженной ошибкам, чем явная передача именованных аргументов. Поэтому рекомендуется использовать именованные аргументы для явной передачи учетных данных:

```Python
authenticate(request, username=username, password=password)
```

**Процессы при вызове:**

Когда вы вызываете функцию `authenticate`, Django перебирает все зарегистрированные бэкенды аутентификации один за другим. Для каждого бэкенда функция `authenticate` проверяет, может ли бэкенд принять предоставленные учетные данные. Если бэкенд может принять учетные данные, он пытается аутентифицировать пользователя с помощью этих учетных данных.

Если бэкенд успешно аутентифицирует пользователя, функция `authenticate` возвращает объект `User`, представляющий аутентифицированного пользователя. Если ни один из бэкендов не смог аутентифицировать учетные данные или если бэкенд явно отклоняет учетные данные (например, если пользователь заблокирован), функция `authenticate` возвращает `None`, указывая, что аутентификация не удалась.

> [!important]  
> Функция authenticate не выполняет вход пользователя, а только проверяет учетные данные. Для выполнения входа пользователя после успешной аутентификации вы можете использовать функцию login() или классы представлений, такие как LoginView.  

**Пример использования:**

```Python
from django.contrib.auth import authenticate

user = authenticate(username='john', password='secret')
if user is not None:
# Аутентификация успешна
    print('Пользователь', user.username, 'аутентифицирован')
else:
# Аутентификация не удалась
    print('Неверные учетные данные')
```

В этом примере мы вызываем функцию `authenticate` с именем пользователя 'john' и паролем 'secret'. Если учетные данные действительны, мы получаем объект `User` и можем выполнить дальнейшие действия, такие как вход пользователя. Если учетные данные недействительны, мы получаем `None` и можем обработать ошибку аутентификации соответствующим образом.

*С 5 версии Django добавлена функция `aauthenticate()` для асинхронного кода.

### `login()`

Функция `login()` в Django используется для входа пользователя в систему после успешной аутентификации.

Эта функция сохраняет идентификатор пользователя и бэкенд аутентификации в сессии запроса. Это позволяет пользователю оставаться аутентифицированным на протяжении нескольких запросов без необходимости повторной аутентификации. Функция `login()` не возвращает явного значения.

**Синтаксис:**

```Python
login(request, user, backend=None)
```

- `request`: Объект `HttpRequest`, представляющий текущий запрос.
- `user`: Объект `User`, представляющий пользователя, которого нужно залогинить.
- `backend`: Необязательный аргумент, указывающий на бэкенд аутентификации, используемый для аутентификации пользователя. Если не указан, Django попытается определить бэкенд автоматически.

**Процессы при вызове:**

Когда вызывается функция `login()`, она проверяет наличие переданного пользователя (`user`). Если аргумент `user` равен `None`, функция использует пользователя из объекта запроса (`request.user`). Затем она проверяет наличие атрибута `get_session_auth_hash` у объекта пользователя и получает хеш сессии аутентификации, если он доступен. Этот хеш используется для проверки целостности сессии.

Функция `login()` проверяет наличие ключа `SESSION_KEY` в сессии запроса. Если ключ уже существует, функция сравнивает его значение с идентификатором текущего пользователя и хешем сессии. Если они не совпадают, сессия очищается во избежание повторного использования сессии другого пользователя. Это гарантирует, что каждый пользователь имеет свою собственную сессию.

Если бэкенд аутентификации не указан явно при вызове функции `login()`, Django пытается определить его автоматически. Сначала он проверяет наличие атрибута `backend` у объекта пользователя (`user.backend`). Если атрибут отсутствует, Django использует список бэкендов аутентификации, указанных в настройках (`AUTHENTICATION_BACKENDS`). Если в списке только один бэкенд, он используется по умолчанию. В противном случае возникает исключение `ValueError`, указывающее на необходимость явного указания бэкенда.

После определения бэкенда аутентификации функция `login()` сохраняет идентификатор пользователя, бэкенд и хеш сессии в сессии запроса. Она также обновляет атрибут `user` объекта запроса текущим пользователем, если он доступен. Затем вызывается функция `rotate_token()` для обновления токена сессии в целях безопасности. В конце функция `login()` отправляет сигнал `user_logged_in`, указывающий на успешный вход пользователя.

> [!important]  
> Функцию login() обычно используют в сочетании с authenticate() для выполнения полного процесса аутентификации и входа пользователя.  

**Пример использования:**

```Python
from django.contrib.auth import authenticate, login

def my_view(request):
    username = request.POST["username"]
    password = request.POST["password"]
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
# Перенаправление на success_url
        ...
    else:
# Возврат сообщения об ошибке неверного логина
        ...
```

В этом примере после успешной аутентификации пользователя с помощью `authenticate()` вызывается функция `login()` для входа пользователя. Затем можно выполнить перенаправление на success_url или выполнить другие необходимые действия.

*В 5 версии Django была добавлена `alogin()`.

### `logout()`

Функция `logout()` в Django используется для выхода пользователя из системы и очистки данных сессии.

Она удаляет идентификатор аутентифицированного пользователя из запроса и очищает данные сессии. Функция `logout()` не возвращает никакого значения.

**Синтаксис:**

```Python
logout(request)
```

- `request`: Объект `HttpRequest`, представляющий текущий запрос.

**Процессы при вызове:  
  
**Перед тем как пользователь будет разлогинен, функция `logout()` отправляет сигнал `user_logged_out`, чтобы получатели сигнала могли узнать, кто именно вышел из системы. Она получает текущего пользователя из атрибута `request.user`, если он доступен, и проверяет, является ли пользователь аутентифицированным. Если пользователь не аутентифицирован, значение `user` устанавливается в `None`.

Затем функция `logout()` вызывает метод `flush()` сессии запроса, который полностью очищает все данные сессии для текущего запроса. Все существующие данные удаляются. Это делается для предотвращения доступа другого пользователя к данным сессии предыдущего пользователя при использовании того же веб-браузера.

После очистки данных сессии функция `logout()` проверяет наличие атрибута `user` в объекте запроса. Если атрибут существует, она заменяет значение `request.user` на экземпляр класса `AnonymousUser()`, который представляет неаутентифицированного пользователя. Это гарантирует, что после выхода из системы пользователь будет считаться анонимным.

**Дополнительная информация:**

- Функция `logout()` не вызывает никаких ошибок, если пользователь не был залогинен.
- После вызова `logout()` все данные сессии для текущего запроса полностью очищаются. Если нужно сохранить какие-либо данные в сессии, которые будут доступны пользователю сразу после выхода из системы, это следует делать после вызова `logout()`.

**Пример использования:**

```Python
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
# Перенаправление на success_url
    ...
```

В этом примере представлено использование функции `logout()` внутри представления Django. После вызова `logout(request)` пользователь будет разлогинен, и его данные сессии будут очищены. Затем можно выполнить перенаправление на success_url или другие необходимые действия.

*`alogout()` 😏

<div class="page-break" style="page-break-before: always;"></div>
