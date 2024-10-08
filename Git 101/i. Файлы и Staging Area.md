После того как вы инициализировали репозиторий и определились с веткой, можно начинать работать с файлами…)

[![](https://miro.medium.com/v2/da:true/resize:fit:1019/1*qNXGXUSZ_6kztZG783mjdQ.gif)](https://miro.medium.com/v2/da:true/resize:fit:1019/1*qNXGXUSZ_6kztZG783mjdQ.gif)

1. **Просмотр статуса файлов**:
    - Команда `git status` позволяет просматривать статус файлов в вашем репозитории.
    - Вы можете использовать эту команду, чтобы увидеть, какие файлы были изменены, добавлены в область индекса или удалены, и где они находятся в вашем рабочем дереве.
    - Пример:
        
        ```Shell
        git status
        ```
        
2. **Добавление файлов в область индекса**:
    - Чтобы добавить изменения в файлы в область индекса, используйте команду `git add` с именем файла или директории.
    - Пример добавления одного файла:
        
        ```Shell
        git add filename.txt
        ```
        
    - Пример добавления всех измененных файлов в текущей директории и её поддиректориях:
        
        ```Shell
        git add .
        ```
        
    - После выполнения этой команды изменения в указанных файлах будут добавлены в область индекса и будут готовы для коммита.
    - **Флаг** `**-p**`**,** `**--patch**`:
        - Запускает интерактивный режим для выборочного добавления изменений в область индекса.
        - Команда предлагает вам просмотреть изменения в каждом файле по отдельности и выбрать, какие из них добавить.
3. **Удаление файлов из области индекса**:
    
    - Если вы добавили файл в область индекса, но передумали его включать в следующий коммит, вы можете удалить его из области индекса, оставив изменения в рабочем дереве.
    - Для удаления файла из области индекса (staging area) и возврата его к предыдущему состоянию в рабочей директории следует использовать команду `git restore` с флагом `--staged`, чтобы отменить изменения только в staging area.
        
        ```Shell
        git restore --staged <file_name>
        ```
        
    - Эта команда уберет файл из staging area, но оставит его изменения в рабочей директории.
    
    ### Интерактивный режим добавления файлов в staging area
    
    Команда `git add -i` запускает интерактивный режим добавления изменений в область индекса (staging area) в Git. Этот интерактивный режим предоставляет пользователю возможность выбирать, какие изменения в файлах добавлять в staging area. Это полезно, если у вас есть несколько изменений в разных файлах, и вы хотите добавить их по-отдельности или выборочно. Вот как использовать команду `git add -i`:
    
    1. **Запуск интерактивного режима**:
        - Для запуска интерактивного режима добавления изменений выполните команду:
            
            ```Shell
            git add -i
            ```
            
    2. **Выбор действия**:
        - После запуска команды `git add -i` появится интерактивное меню с различными опциями. Пример:
            
            ```Shell
            *** Commands ***
              1: status       2: update       3: revert       4: add untracked
              5: patch        6: diff         7: quit         8: help
            What now>
            ```
            
        - В зависимости от вашей цели выберите соответствующий номер опции.
    3. **Применение выбранного действия**:
        - Выберите нужное действие из меню, например, `5` для просмотра изменений в файлах.
        - Введите номер опции и нажмите Enter.
    4. **Применение изменений**:
        - После выбора действия вам может потребоваться ввести дополнительные сведения, такие как имя файла или номер строки, к которой нужно применить изменения.
        - Введите соответствующие данные и нажмите Enter.
    5. **Завершение работы в интерактивном режиме**:
        - После выполнения нужных действий выберите опцию `7` или введите `quit`, чтобы выйти из интерактивного режима.
    
    Использование команды `git add -i` позволяет более гибко и точечно управлять добавлением изменений в staging area, что удобно в ситуациях, когда требуется выборочное добавление изменений или когда у вас есть несколько изменений в разных файлах.
    <div class="page-break" style="page-break-before: always;"></div>