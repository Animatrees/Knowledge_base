Что такое пагинация?

Класс Paginator

Описание

Создание объекта Paginator

Пример использования

Атрибуты класса

Методы класса

Класс Page

Описание

Создание объекта Page

Взаимодействие с классом Paginator

Атрибуты класса

Методы класса

Пример использования в функции представлении

Исключения

InvalidPage

PageNotAnInteger

EmptyPage

### Что такое пагинация?

**Пагинация** - это разбиение большого количества данных на несколько страниц. Она применяется для удобства отображения и повышения производительности веб-приложений. Django предоставляет встроенные классы **Paginator** и **Page** для реализации пагинации.

## Класс Paginator

### Описание

Класс `Paginator` в Django используется для разделения списка объектов на отдельные страницы. Он принимает список объектов и количество элементов на странице, после чего позволяет получать объекты `Page`, представляющие отдельные страницы.

### Создание объекта Paginator

Для создания объекта `Paginator` используется следующий синтаксис:

```Python
paginator = Paginator(object_list, per_page, orphans=0, allow_empty_first_page=True)
```

**Параметры:**

- `object_list` (обязательный): список, кортеж, QuerySet или любой другой объект, поддерживающий срезы и имеющий метод `count()` или `__len__()`.
- `per_page` (обязательный): максимальное количество элементов на каждой странице.
- `orphans` (необязательный, по умолчанию 0): минимальное количество элементов, которые могут быть на последней странице. Если количество элементов на последней странице меньше или равно `orphans`, то они добавляются к предыдущей странице.
- `allow_empty_first_page` (необязательный, по умолчанию True): определяет, разрешено ли первой странице быть пустой. Если установлено значение False и `object_list` пуст, то будет возбуждено исключение `EmptyPage`.

### Пример использования

```Python
from django.core.paginator import Paginator

objects = [1, 2, 3, 4, 5, 6, 7, 8, 9]
paginator = Paginator(objects, 3)# 3 элемента на странице

print(paginator.num_pages)# Вывод: 3
page1 = paginator.page(1)
print(page1.object_list)# Вывод: [1, 2, 3]
```

### Атрибуты класса

- `count` - общее количество объектов во всём списке.  
    Пример:  
    `print(paginator.count) # Вывод: 9`
- `num_pages` - общее количество страниц.  
    Пример:  
    `print(paginator.num_pages) # Вывод: 3`
- `page_range` - итератор, возвращающий номера страниц (начиная с 1).  
    Пример:  
    `print(list(paginator.page_range)) # Вывод: [1, 2, 3]`
- `ELLIPSIS` - строка, используемая для замены опущенных номеров страниц в урезанном списке, возвращаемом методом `get_elided_page_range()`. По умолчанию равна '...'.

### Методы класса

- `page(number)` - возвращает объект Page для указанного номера страницы (начиная с 1). Если переданный номер страницы не является целым числом, возбуждается исключение `PageNotAnInteger`. Если страница с указанным номером не существует, возбуждается исключение `EmptyPage`.  
    Пример:  
    `page2 = paginator.page(2)`
- `get_page(number)` - возвращает объект Page для указанного номера страницы (начиная с 1), обрабатывая исключения. Если номер страницы не является числом, возвращается первая страница. Если номер страницы отрицательный или больше количества страниц, возвращается последняя страница. Исключение `EmptyPage` возбуждается только в случае, если `allow_empty_first_page=False` и `object_list` пуст.  
    Пример:  
    `page1 = paginator.get_page(1)`
- `get_elided_page_range(number, on_each_side=3, on_ends=2)` - возвращает урезанный список номеров страниц, который может содержать символ `ELLIPSIS` с одной или обеих сторон от текущего номера страницы, если общее количество страниц велико. Параметр `on_each_side` определяет количество страниц, включаемых с каждой стороны от текущей страницы (по умолчанию 3). Параметр `on_ends` определяет количество страниц, включаемых в начале и конце списка (по умолчанию 2). Вызывает ошибку `InvalidPage`, если заданный номер страницы не существует.  
    Пример:  
    
    ```Shell
    In [1]: from django.core.paginator import Paginator
    
    In [2]: lst = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13]
    
    In [3]: p = Paginator(lst, 2)
    
    In [4]: for page in p.get_elided_page_range(2):
       ...:     print(page)
       ...: 
    1
    2
    3
    4
    5
    …
    12
    13
    ```
    

## Класс Page

### Описание

Класс `Page` представляет отдельную страницу объектов из списка, разделённого с помощью `Paginator`. Объекты `Page` обычно не создаются вручную, а получаются с помощью методов `Paginator`.

### Создание объекта Page

Для создания объекта `Page` используется следующий синтаксис:

```Python
page = Page(object_list, number, paginator)
```

Параметры:

- `object_list` (обязательный): список объектов на текущей странице.
- `number` (обязательный): номер текущей страницы (начиная с 1).
- `paginator` (обязательный): объект `Paginator`, связанный с этой страницей.

### Взаимодействие с классом Paginator

Объект `Paginator` создаёт и возвращает объекты `Page` через свои методы `page()` и `get_page()`.

Пример:

```Python
paginator = Paginator(objects, 3)
page1 = paginator.page(1)# Получение объекта Page из Paginator
```

### Атрибуты класса

- `object_list` - список объектов на текущей странице.  
    Пример:  
    `print(page1.object_list) # Вывод: [1, 2, 3]`
- `number` - номер текущей страницы (начиная с 1).  
    Пример:  
    `print(page1.number) # Вывод: 1`
- `paginator` - объект `Paginator`, связанный с этой страницей (тот, что её породил).  
    Пример:  
    `print(page1.paginator) # Вывод: <django.core.paginator.Paginator object at 0x...>`

### Методы класса

- `has_next()` - возвращает True, если есть следующая страница, и False в противном случае.  
    Пример:  
    `print(page1.has_next()) # Вывод: True`
- `has_previous()` - возвращает True, если есть предыдущая страница, и False в противном случае.  
    Пример:  
    `print(page3.has_previous()) # Вывод: True`
- `has_other_pages()` - возвращает True, если есть другие страницы (предыдущая или следующая), и False, если текущая страница является единственной.  
    Пример:  
    `print(page2.has_other_pages()) # Вывод: True`
- `next_page_number()` - возвращает номер следующей страницы. Если следующей страницы нет, возбуждает исключение `InvalidPage`.  
    Пример:  
    `print(page1.next_page_number()) # Вывод: 2`
- `previous_page_number()` - возвращает номер предыдущей страницы. Если предыдущей страницы нет, возбуждает исключение `InvalidPage`.  
    Пример:  
    `print(page2.previous_page_number()) # Вывод: 1`
- `start_index()` - возвращает индекс первого объекта на текущей странице относительно всего списка объектов в объекте `Paginator` (начиная с 1).  
    Пример:  
    `print(page2.start_index()) # Вывод: 4`
- `end_index()` - возвращает индекс последнего объекта на текущей странице относительно всего списка объектов в объекте `Paginator` (начиная с 1).  
    Пример:  
    `print(page2.end_index()) # Вывод: 6`

Класс `Page` также поддерживает итерацию и определение длины, что позволяет обращаться к объектам на странице как к последовательности:

```Python
for obj in page:
    print(obj)

print(len(page))# Вывод: 3 (количество объектов на странице)
```

## Пример использования в функции представлении

```Python
from django.core.paginator import Paginator
from django.shortcuts import render


def show_posts(request):
    posts = Post.published.all()
    paginator = Paginator(posts, 10)
    page_number = request.GET.get('page')
    page = paginator.get_page(page_number)

    return render(request, 'basefunc/list_posts.html', {'title': 'Все посты', 'page': page})
```

И в шаблоне отображение постов + отображение номеров страниц:

```HTML
{% extends 'base.html' %}

{% block content %}
{% for post in page %}
	<p>{{ post.title }}</p>
	<p>{{ post.content }}</p>
{% endfor %}
{% endblock %}

{% block navigation %}
<nav>
    <ul>
        {% for page_number in page.paginator.page_range %}
        <li>
            <a href="?page={{ page_number }}">{{ page_number }}</a>
        </li>
        {% endfor %}
    </ul>
</nav>
{% endblock %}
```

## Исключения

При работе с классами `Paginator` и `Page` могут возникать следующие исключения:

### InvalidPage

`InvalidPage` - базовый класс для исключений, возбуждаемых при передаче некорректного номера страницы в методы `page()` или `get_page()` класса `Paginator`, а также при вызове методов `next_page_number()` или `previous_page_number()` класса `Page`, когда следующая или предыдущая страница отсутствует.

Пример:

```Python
objects = [1, 2, 3, 4, 5, 6, 7, 8, 9]
paginator = Paginator(objects, 3)
try:
    page = paginator.page(5)
except InvalidPage:
    print("Некорректный номер страницы")
```

### PageNotAnInteger

`PageNotAnInteger` - исключение, возбуждаемое при передаче нецелочисленного значения в качестве номера страницы в метод `page()` класса `Paginator`.

Пример:

```Python
paginator = Paginator(objects, 3)
try:
    page = paginator.page("2")
except PageNotAnInteger:
    print("Номер страницы должен быть целым числом")
```

### EmptyPage

`EmptyPage` - исключение, возбуждаемое при передаче номера страницы, для которой не существует объектов, в метод `page()` класса `Paginator`.

Пример:

```Python
paginator = Paginator(objects, 3)
try:
    page = paginator.page(10)
except EmptyPage:
    print("Запрошенная страница не существует")
```

Исключения `PageNotAnInteger` и `EmptyPage` являются подклассами `InvalidPage`, поэтому их можно обрабатывать как вместе, так и по отдельности:

```Python
paginator = Paginator(objects, 3)
try:
    page = paginator.page(10)
except PageNotAnInteger:
    print("Номер страницы должен быть целым числом")
except EmptyPage:
    print("Запрошенная страница не существует")
except InvalidPage:
    print("Некорректный номер страницы")
```

Метод `get_page()` класса `Paginator` автоматически обрабатывает исключения `PageNotAnInteger` и `EmptyPage`, возвращая первую или последнюю страницу соответственно, поэтому явная обработка этих исключений не требуется при использовании данного метода.

<div class="page-break" style="page-break-before: always;"></div>
