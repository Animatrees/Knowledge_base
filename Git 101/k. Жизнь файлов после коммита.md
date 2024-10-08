[![](https://images.velog.io/images/ppnrn/post/f407e541-8ebd-40d2-822e-87ef0155bea6/GIF 2021-03-21 오전 6-05-43.gif)](https://images.velog.io/images/ppnrn/post/f407e541-8ebd-40d2-822e-87ef0155bea6/GIF 2021-03-21 오전 6-05-43.gif)

После первого коммита репозиторий начинает отслеживать файлы, добавленные в этом коммите и у нас появляются новые возможности (и новая ответственность =))

1. `**git rm**`:
    - Команда `git rm <file_name>` используется для удаления файла из рабочей директории и индекса (staging area). Git не удаляет файлы из репозитория напрямую. Вместо этого он помечает файлы для удаления. Помеченные файлы будут удалены из репозитория при следующей команде `git commit`.
    - Использование флага `--cached` позволяет сохранить файл в рабочей директории, но отметить в индексе, чтобы он больше не отслеживался Git.
    - Пример:
        
        ```Shell
        git rm file_to_delete.txt
        ```
        
2. `**git mv**`:
    - Команда `git mv <old_file_name> <new_file_name>` используется для переименования файла в рабочей директории и обновления индекса.
    - Пример:
        
        ```Shell
        git mv old_file_name.txt new_file_name.txt
        ```
        
3. `**git blame**`:
    - Команда `git blame <file_name>` позволяет просмотреть, кто и когда последний раз изменял каждую строку в указанном файле.
    - Это полезно для выявления авторства конкретных изменений и отслеживания, кто внес какие изменения.
4. `**git restore**`:
    - Команда `git restore <file_name>` используется для восстановления файла до его состояния в последнем коммите (т.е. отменяет изменения, которые были сделаны после последнего коммита).
    - `git restore <хэш_коммита> <имя_файла>`: Восстанавливает файл из указанного коммита.
5. `**git diff**`:
    - Команда `git diff` предназначена для сравнения изменений между различными состояниями отслеживаемых файлов.
    - Если файл не отслеживается Git (то есть он не был добавлен в репозиторий с помощью `git add` и не включен в коммит), то `git diff` не покажет различий для этого файла.
    -  `git diff` без флагов показывает разницу между рабочим каталогом и последним коммитом. Это позволяет увидеть все изменения, которые еще не были проиндексированы.
    - `git diff --staged` (или `git diff --cached`) показывает разницу между изменениями, которые находятся в промежуточной области (индексе) и последним коммитом. Это позволяет просмотреть, какие изменения будут включены в следующий коммит.
    - Вывод `git diff` показывает изменения пофайлово в специальном формате:
        
        - `+` обозначает добавленную строку.
        -  обозначает удаленную строку.
        - Изменения в строке обозначаются с выделением старых и новых участков.
        
        ```Shell
        $ git status
        On branch master
        Changes to be committed:
                modified:   app.py
        
        Changes not staged for commit:
                modified:   README.md
        
        $ git diff --staged   # Посмотрим, что поставлено в staging area 
        diff --git a/app.py b/app.py
        index 76c891a..8868d65 100644
        --- a/app.py
        +++ b/app.py
        @@ -1,3 +1,4 @@
        -import random
        +import random 
        +import time
        
         def hello():
                print("Hello, world!")
        
        $ git diff   # Посмотрим, какие изменений есть вообще (и в staging area, и без)
        diff --git a/README.md b/README.md
        index 5c891ab..3368b45c 100644
        --- a/README.md
        +++ b/README.md
        @@ -1,2 +1,3 @@
        -Git Tutorial 
        +# Git Tutorial
        +Version 1.0
        ```
        
6. `**git grep**`: Используется для поиска строк текста в файлах вашего Git-репозитория. Это может быть полезным для поиска определенного текста, строки кода, имени функции и т. д. в вашем проекте.
    
    Пример использования:
    
    ```Shell
    git grep "search_text"
    ```
    
    Эта команда вернет список файлов и строк, в которых найдено совпадение с указанным текстом. Вы также можете использовать регулярные выражения для более гибкого поиска.
    
7. `**git clean -fd**`:
    - Удаляет все файлы и директории из рабочей области, которые не отслеживаются Git.
    - Флаг `n` позволяет сначала просмотреть список файлов, которые будут удалены, без их фактического удаления.
    - Пример:
        
        ```Shell
        git clean -fd
        ```
<div class="page-break" style="page-break-before: always;"></div>