Глобальные обработчики ошибок

Основные глобальные обработчики ошибок

Настройка файла settings.py

Обработка исключений

### Глобальные обработчики ошибок

В Django **обработчики ошибок** - это специальные функции или классы, которые используются для обработки и отображения информации об ошибках, возникающих во время обработки запросов. Эти обработчики позволяют приложению более гибко управлять ошибками и предоставлять пользователю понятную и информативную обратную связь в случае возникновения проблем.

**Глобальные обработчики** ошибок в Django определяются в файле `**urls.py**` проекта и применяются ко всему приложению. Они используются для обработки конкретных типов ошибок на уровне всего приложения, таких как ошибки 404 (страница не найдена), ошибки 500 (внутренняя ошибка сервера) и т.д.

```Python
\#urls.py (for project)

from django.contrib import admin
from django.urls import path, include
from basefunc.views import page_not_found


urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('basefunc.urls')),
]

handler404 = page_not_found
```

Сама функция-представление при этом может быть описана как в файле с представлениями для главного приложения или в отдельном файле, например, `handlers.py`.

```Python
\#views.py (for app)
from django.http import HttpResponse, HttpResponseNotFound
...

def page_not_found(request, exception):
    return HttpResponseNotFound('Страница не существует')
```

### Основные глобальные обработчики ошибок

1. `**handler400**`:
    - Вызывается при ошибке с кодом 400 (неверный запрос).
    - Используется для обработки ситуаций, когда запрос клиента некорректен, например, из-за отсутствия обязательных параметров или некорректного формата данных.
2. `**handler403**`:
    - Вызывается при ошибке с кодом 403 (доступ запрещен).
    - Используется для обработки ситуаций, когда у пользователя нет прав доступа к определенному ресурсу.
3. `**handler404**`:
    - Вызывается при ошибке с кодом 404 (страница не найдена).
    - Используется для обработки ситуаций, когда запрошенный маршрут (URL) или представление не существует.
4. `**handler500**`:
    - Вызывается при внутренней ошибке сервера с кодом 500.
    - Используется для обработки ситуаций, когда происходит критическая ошибка на стороне сервера.

### Настройка файла `settings.py`

В файле `**settings.py**` Django использует параметры `**DEBUG**` и `**ALLOWED_HOSTS**` для обеспечения безопасности и управления поведением приложения в различных средах.

1. **DEBUG**:
    - Параметр `**DEBUG**` определяет, находится ли проект в режиме отладки (True) или в производственном режиме (False).
    - Когда `**DEBUG**` установлен в `**True**`, Django предоставляет дополнительную отладочную информацию при возникновении ошибок, а также автоматически обновляет код и обеспечивает другие возможности, удобные для разработки.
    - В режиме продакшена (`**DEBUG = False**`), Django предоставляет минимум информации об ошибках и не отображает подробности трассировки ошибок пользователю, чтобы предотвратить возможные утечки информации о системе.
2. **ALLOWED_HOSTS**:
    - `**ALLOWED_HOSTS**` - это список строк, представляющих разрешенные хосты (домены), которые могут обращаться к приложению.
    - Этот параметр важен для безопасности приложения. Если `**ALLOWED_HOSTS**` не указан или указан неверно, Django может отклонить запросы от не разрешённых хостов.
    - В режиме продакшена в `**ALLOWED_HOSTS**` должен быть указан домен приложения.

Важно изменять эти параметры для тестирования обработчиков ошибок (и перед паблишем проекта в продакшн), чтобы видеть их вывод, а не вывод отладочной информации из Django.  
Например, в режиме отладки, если произойдет ошибка сервера (500), Django автоматически отобразит отладочную информацию, что удобно для разработчика, но не безопасно для конечного пользователя.  
В производственной среде, где DEBUG должен быть установлен в False, пользователи не должны видеть подобную информацию, и им должна выводиться кастомизированная страница ошибки.  
При переводе проекта в режим продакшена мы также должны указать список разрешённых хостов в  
`**ALLOWED_HOSTS**`, иначе у нас не будет доступа к проекту.

### Обработка исключений

`**Http404**` - это класс исключения в Django, который используется для обработки ситуации, когда запрошенный ресурс (например, страница) не может быть найден. Обычно это возникает, когда пользователь запрашивает URL, который не существует в приложении.

Это исключение можно вызвать в представлении, когда необходимо сообщить клиенту, что запрошенная страница не найдена. Django предоставляет специальную функцию `**django.http.Http404()**` для этого.

Пример использования:

```Python
from django.http import Http404
from django.shortcuts import render


def my_view(request):
    # Проверяем, есть ли у нас запрошенный объект в базе данных
    if not object_exists_in_database:
        # Если объект не найден, вызываем Http404
        raise Http404("Запрошенный объект не найден")
    else:
        # Если объект найден, отображаем его
        return render(request, 'template.html', context)
```

Когда исключение `**Http404**` вызывается в представлении, Django автоматически обрабатывает его и выводит стандартную страницу с сообщением об ошибке 404 ("Страница не найдена") пользователю.

Помимо `**Http404**`, в Django существует ряд других исключений, связанных с обработкой запросов и взаимодействием с HTTP-протоколом. Некоторые из них:

1. `**Http400**`:
    - Исключение `**Http400**` используется для обработки некорректных запросов (например, неправильного формата данных или недостаточных параметров).
    - Обычно вызывается в представлении, когда данные запроса не соответствуют ожидаемому формату или требованиям.
    - Оно может быть использовано для сообщения пользователю о неправильном вводе данных или ошибках при обработке запроса.
2. `**PermissionDenied**`:
    - Исключение `**PermissionDenied**` вызывается, когда у пользователя отсутствуют необходимые разрешения для доступа к определенному ресурсу или выполнения определенного действия.
    - Оно может быть использовано для контроля доступа к чувствительным частям приложения или для предотвращения несанкционированного доступа.
3. `**ValidationError**`:
    - `**ValidationError**` - это общее исключение, которое используется для обработки ошибок валидации данных.
    - Оно может быть вызвано при попытке сохранения некорректных данных в базе данных или при проверке данных в формах.
    - Обычно используется вместе с Django Forms API для проверки и валидации пользовательского ввода.
4. `**SuspiciousOperation**`:
    - Исключение `**SuspiciousOperation**` вызывается, когда обнаружены подозрительные операции или запросы, которые могут указывать на потенциальные атаки на безопасность.
    - Оно помогает защитить приложение от некоторых типов атак, таких как запросы с недействительными или поддельными данными.

<div class="page-break" style="page-break-before: always;"></div>
