[![](https://miro.medium.com/v2/resize:fit:935/1*0Eqod1YY6zOW8G7rCB4qbg.jpeg)](https://miro.medium.com/v2/resize:fit:935/1*0Eqod1YY6zOW8G7rCB4qbg.jpeg)

В Git существуют различные статусы файлов, которые отражают их текущее состояние в рабочей области проекта.

1. **Untracked (Не отслеживаемый)**:
    - Файлы, которые Git не отслеживает.
    - Это могут быть новые файлы, которые вы создали в рабочей области, но которые еще не были добавлены в репозиторий с помощью команды `git add`.
    - Файлы с не отслеживаемым статусом не включены в следующий коммит, пока вы явно не добавите их в область индекса.
2. **Staged (Готовый к коммиту)**:
    - Файлы, которые были добавлены в область индекса с помощью команды `git add`.
    - Эти файлы подготовлены к включению в следующий коммит.
    - Когда вы используете команду `git commit`, Git зафиксирует состояние всех файлов, находящихся в области индекса.
3. **Unmodified (Неизмененный)**:
    - Файлы, которые находятся в рабочей области и не были изменены с момента последнего коммита.
    - Эти файлы не требуют дополнительной работы и не будут включены в следующий коммит, если они останутся без изменений.
4. **Modified (Измененный)**:
    - Файлы, которые находятся в рабочей области и были изменены с момента последнего коммита.
    - Эти файлы содержат изменения, которые еще не были добавлены в область индекса.
    - Если измененные файлы были добавлены в область индекса, они будут иметь статус "Staged". Если вы продолжите вносить изменения, они останутся снова с статусом "Modified".

Изменения в статусах файлов отражают ваш прогресс в работе с проектом и помогают в управлении версиями с помощью Git. Статусы файлов также являются основой для принятия решений о том, какие изменения включать в следующий коммит.

Вы можете проверить статус всех файлов в рабочей области с помощью команды `git status`.

```Shell
$ git status
On branch master
Untracked files:
    (use "git add <file>..." to include in what will be committed)
    README.md

Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
    new file:   app.py

No changes to tracked files.
```

**В этом примере:**

- `README.md` - неотслеживаемый файл.
- `app.py` - добавлен в область подготовки.
- Файлов с изменениями, но не добавленных в область подготовки, нет.
<div class="page-break" style="page-break-before: always;"></div>
