Основные понятия

Разрешения (Permissions)

Класс PermissionRequiredMixin

Декоратор permission_required

Группы (Groups)

Управление разрешениями и группами программно

Создание пользовательских разрешений

Использование разрешений в шаблонах

*Использование системных сигналов

Права пользователя в Django являются важным аспектом системы аутентификации и авторизации. Они позволяют контролировать доступ пользователей к различным частям веб-приложения. Django предоставляет встроенные механизмы для управления правами, которые можно использовать для защиты данных и функций приложения.

### **Основные понятия**

1. **Разрешения (Permissions)**: Определяют, что пользователь может или не может делать.
2. **Группы (Groups)**: Объединяют пользователей для удобного назначения набора разрешений.

### **Разрешения (Permissions)**

**Встроенные разрешения**

Django автоматически создает четыре базовых разрешения для каждой модели:

- `add` – добавление новых записей.
- `change` – изменение существующих записей.
- `delete` – удаление записей.
- `view` – просмотр записей.

**Пример использования разрешений**

1. **Назначение разрешений через админ-панель**:
    - Откройте профиль пользователя в админ-панели.
    - В нижней части страницы выберите нужные разрешения.
2. **Ограничение доступа к представлениям**:
    - Используйте миксин `PermissionRequiredMixin` в классовых представлениях:
        
        ```Python
        from django.contrib.auth.mixins import PermissionRequiredMixin
        class AddPage(PermissionRequiredMixin, CreateView):
            permission_required = 'app_name.add_modelname'
        ```
        
    - Используйте декоратор `permission_required` для функций представления:
        
        ```Python
        from django.contrib.auth.decorators import permission_required
        
        @permission_required('app_name.view_modelname', raise_exception=True)
        def view_page(request):
            return HttpResponse("Просмотр страницы")
        ```
        

### **Класс PermissionRequiredMixin**

`PermissionRequiredMixin` – это миксин для проверки наличия у пользователя необходимых разрешений для доступа к представлению. Этот миксин аналогичен декоратору `permission_required`, но используется в классовых представлениях.

Наследуется от [[j-g. Параметр next, декоратор login_required, класс LoginRequiredMixin]], который предоставляет базовые механизмы проверки доступа, включая проверку на авторизацию.

**Основные задачи**

- Проверить, имеет ли текущий пользователь указанные разрешения.
- Обработать случай, когда пользователь не имеет необходимых разрешений.

**Атрибуты**

- `permission_required`: одно разрешение в виде строки или несколько разрешений в виде списка или кортежа. По умолчанию `None`.

**Методы**

- `get_permission_required()`: Возвращает список (или кортеж) необходимых разрешений. Если атрибут `permission_required` не задан, вызывает ошибку конфигурации.
- `has_permission()`: Проверяет, имеет ли текущий пользователь все указанные разрешения, возвращает булево значение.
- `dispatch()`: Основной метод, который проверяет разрешения перед вызовом представления. Если разрешений недостаточно, вызывается метод `handle_no_permission()` для обработки отказа в доступе.

**Пример использования**

```Python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import View

class MyView(PermissionRequiredMixin, View):
    permission_required = "polls.add_choice"
    # Или для нескольких разрешений:
    permission_required = ["polls.view_choice", "polls.change_choice"]
    ...
```

### **Декоратор permission_required**

**Основной синтаксис**

```Python
from django.contrib.auth.decorators import permission_required

@permission_required("app_label.permission_codename", login_url=None, raise_exception=False)
def my_view(request):
    # Логика представления
    ...
```

**Атрибуты**

- `perm`: строка с именем разрешения в формате `"app_label.permission_codename"` или список/кортеж разрешений.
- `login_url`: URL для перенаправления на страницу входа, если пользователь не авторизован (по умолчанию берется из `settings.LOGIN_URL`).
- `raise_exception`: если `True`, вызывает `PermissionDenied` и возвращает ошибку 403 (Forbidden), вместо перенаправления на страницу входа.

**Пример использования**

```Python
from django.contrib.auth.decorators import permission_required

@permission_required("polls.add_choice")
def add_choice(request):
    # Логика представления для добавления выбора
    ...
```

**Комбинирование с декоратором** `**login_required**`

Чтобы сначала предоставить пользователю возможность войти в систему, а затем проверять разрешения, используйте декораторы `login_required` и `permission_required` вместе.

```Python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
@permission_required("polls.add_choice", raise_exception=True)
def add_choice(request):
    # Логика представления для добавления выбора
    ...
```

### **Группы (Groups)**

**Создание и назначение групп**

1. **Создание группы в админ-панели**:
    - Перейдите в раздел групп.
    - Создайте новую группу и добавьте необходимые разрешения.
2. **Назначение группы пользователю**:
    - Откройте профиль пользователя в админ-панели.
    - Добавьте пользователя в нужную группу.

**Пример использования групп**

- Если пользователю назначены несколько групп, он получает все разрешения этих групп.
- Удобно использовать группы для назначения одинаковых прав множеству пользователей.

### **Управление разрешениями и группами программно**

1. **Работа с группами**:
    
    ```Shell
    myuser.groups.set([group_list]) # добавление списка групп
    myuser.groups.add(group, group, ...) # добавление одной или нескольких групп
    myuser.groups.remove(group, group, ...) # удаление одной или нескольких групп
    myuser.groups.clear() # очистка всех групп пользователя
    ```
    
    - Пример использования:
        
        ```Python
        from django.contrib.auth.models import User, Group
        user = User.objects.get(username='user1')
        group = Group.objects.get(name='group_name')
        user.groups.add(group)
        ```
        
2. **Работа с разрешениями**:
    
    ```Shell
    myuser.user_permissions.set([permission_list]) # добавление списка разрешений
    myuser.user_permissions.add(permission, permission, ...) # добавление 1 или больше разрешений
    myuser.user_permissions.remove(permission, permission, ...) # удаление 1 или больше разрешений
    myuser.user_permissions.clear() # очистка всех разрешений пользователя
    ```
    
    - Пример использования:
        
        ```Python
        from django.contrib.auth.models import Permission
        permission = Permission.objects.get(codename='add_modelname')
        user.user_permissions.add(permission)
        ```
        
3. **Проверка наличия разрешения:**
    
    ```Shell
    myuser.has_perm(permission)
    ```
    

### **Создание пользовательских разрешений**

Иногда возникает необходимость создания собственных разрешений:

```Shell
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType


content_type = ContentType.objects.get_for_model(User)
permission = Permission.objects.create(
    codename='can_publish',
    name='Can Publish Articles',
    content_type=content_type,
)
```

- Новое разрешение `can_publish` будет добавлено к модели `User`.

### **Использование разрешений в шаблонах**

Django передает коллекцию `perms` в каждый шаблон, содержащую разрешения текущего пользователя:

```HTML
{% if perms.app_name.change_modelname %}
<a href="{% url 'edit_modelname' modelname.id %}">Редактировать</a>
{% endif %}
```

- Ссылка "Редактировать" будет видна только пользователям с разрешением `change_modelname`.

### ***Использование системных сигналов**

Системные сигналы позволяют вам реагировать на изменения, связанные с разрешениями и группами. Например, вы можете автоматически добавлять пользователей в определенные группы при их создании.

**Пример использования сигнала для добавления пользователя в группу:**

```Python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User, Group

@receiver(post_save, sender=User)
def add_to_default_group(sender, instance, created, **kwargs):
    if created:
        default_group = Group.objects.get(name='DefaultGroup')
        instance.groups.add(default_group)
```

Может быть полезно, когда всем новым пользователям нужно выдать определённые права при регистрации)

<div class="page-break" style="page-break-before: always;"></div>
