[![](https://get.czh.me/slide-git-internals/images/11.png)](https://get.czh.me/slide-git-internals/images/11.png)

В Git существуют различные типы объектов, которые составляют его внутреннюю структуру.

1. **Blob (Блоб)**:
    - Блоб представляет собой контент конкретного файла в репозитории.
    - Каждый файл в Git хранится как блоб, который представляет собой набор байтов, содержащий фактический текст файла.
2. **Tree (Дерево)**:
    - Дерево представляет собой структуру каталогов в репозитории и содержит ссылки на блобы и другие деревья.
    - Одно дерево может содержать ссылки на несколько блобов и других деревьев, что позволяет Git хранить структуру файлов и каталогов.
3. **Commit (Коммит)**:
    - Коммит представляет собой состояние проекта в определённый момент времени.
    - Он содержит ссылку на дерево, которое описывает структуру файлов и каталогов в этом состоянии проекта, а также метаданные, такие как автор коммита, дата и сообщение о коммите.
4. **Branch (Ветка)**:
    - Ветка представляет собой метку на конкретном коммите в истории проекта.
    - Она позволяет отслеживать изменения в проекте на различных параллельных ветвях разработки.
    - Создание новой ветки создаёт указатель на последний коммит текущей ветви, а затем можно безопасно вносить изменения, не затрагивая основную ветку.
5. **Тег (Tag)**:
    - Тег представляет собой метку на конкретном коммите в истории проекта, аналогично ветке, но используется для фиксации определенного состояния проекта на момент времени.
    - Он обычно используется для пометки определенных релизов или важных точек в истории проекта, таких как версии программного обеспечения.
    - Создание нового тега создает метку на определенном коммите, который можно легко идентифицировать по метке в дальнейшем.
    - Теги обычно используются для обозначения стабильных версий проекта и облегчения навигации по истории изменений.
6. **HEAD**:
    - HEAD - это указатель на текущую активную ветку в репозитории.
    - Он обычно указывает на последний коммит в выбранной ветке.
    - HEAD также может указывать напрямую на конкретный коммит, что означает, что репозиторий находится в состоянии "detached HEAD", где изменения не будут привязаны к какой-либо ветке.

Взаимодействие между этими типами объектов обеспечивает возможность отслеживания и управления версиями проекта в Git. Блобы хранят контент файлов, деревья описывают структуру каталогов, коммиты фиксируют состояния проекта, ветки позволяют работать с параллельными версиями проекта, а HEAD указывает на текущее состояние репозитория.

### **Хэш-идентификаторы в Git**

Хэш-идентификатор, также известный как хэш или SHA-1, представляет собой уникальную строку символов, которая используется Git для идентификации объектов, таких как коммиты, ветки, теги и файлы.

**Зачем нужны?**

- Хэши гарантируют целостность данных. Если даже один бит в файле изменится, его хэш также значительно изменится.
- Хэши позволяют Git быстро и эффективно находить объекты в репозитории.
- Хэши используются для создания ссылок между объектами Git.

**Как используются?**

- Git использует хэши для создания ссылок между коммитами.
- Когда вы создаете коммит, Git вычисляет хэш коммита и использует его для ссылки на родительский коммит.
- Git также использует хэши для создания ссылок на ветки и теги.
- Вы можете использовать хэши для поиска объектов Git.
<div class="page-break" style="page-break-before: always;"></div>
