Если наша форма не связана с моделью, но названия полей совпадают, то следующим образом можно добавить данные, полученные из формы, в базу данных:

```Python
def addpage(request):
    if request.method == 'POST':
        form = AddPostForm(request.POST)
        if form.is_valid():
            try:
                Post.objects.create(**form.cleaned_data)
                return redirect('home')
            except:
                form.add_error(None, 'Ошибка добавления поста')

    return render(request, 'basefunc/addpage.html', {'title': 'Добавление новой статьи', 'form': AddPostForm()})
```

1. `if form.is_valid():`:
    - Этот условный блок проверяет, является ли форма, переданная в представление, действительной (то есть, прошла валидацию).
    - `form.is_valid()` возвращает `True`, если все поля формы прошли валидацию, и `False` в противном случае.
    - это условие **обязательно**, так как только после того, как форма проходит валидацию, для неё будет создан атрибут - `cleaned_data`.
2. `try:`:
    - Этот блок `try-except` используется для обработки возможных ошибок, которые могут возникнуть при создании нового объекта в базе данных.
3. `Post.objects.create(**form.cleaned_data)`:
    - Эта строка пытается создать новый объект модели `Post` в базе данных, используя очищенные данные из формы.
    - `**form.cleaned_data` распаковывает словарь `form.cleaned_data` в виде отдельных аргументов для метода `create()`.
4. `return redirect('home')`:
    - Если объект успешно создан, эта строка перенаправляет пользователя на URL-адрес с именем `'home'`.
5. `except:`:
    - Если во время создания объекта произошла ошибка, отработает блок `except`.
6. `form.add_error(None, 'Ошибка добавления поста')`:
    - Эта строка добавляет общую ошибку форме, которая не связана с конкретным полем.
    - `None` означает, что ошибка не относится к какому-либо конкретному полю, а `'Ошибка добавления поста'` - это текст ошибки.

> [!important]  
> Описанный выше способ будет работать при строгом соответствии полей в форме с теми, которые принимает модель при создании.  

В случае, если мы хотим также создать для объекта связи многие ко многим и отображаем в форме поле подобное этому:

```Python
tags = forms.ModelMultipleChoiceField(queryset=Tags.objects.all(), label='Выберите теги', required=False)
```

Можно обойти проблему следующим образом:

```Python
def addpage(request):
    if request.method == 'POST':
        form = AddPostForm(request.POST)
        if form.is_valid():
            try:
                tags = form.cleaned_data.pop('tags')
                new_woman = Post.objects.create(**form.cleaned_data)
                new_woman.tags.set(tags)
                return redirect('home')
            except:
                form.add_error(None, 'Ошибка добавления поста')

    return render(request, 'basefunc/addpage.html', {'title': 'Добавление новой статьи', 'form': AddPostForm()})
```

Удаляем теги из словаря → сохраняем созданный объект в переменную → создаём связи, используя метод `set`.

<div class="page-break" style="page-break-before: always;"></div>
