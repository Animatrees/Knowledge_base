1. Включаем в настройках почты доступ по IMAP протоколу.
    
    ![[Untitled 15.png|Untitled 15.png]]
    
    ![[Untitled 1 8.png|Untitled 1 8.png]]
    
    ![[Untitled 2 6.png|Untitled 2 6.png]]
    
    Далее следуем инструкциям для подтверждения личности)
    
2. Чтобы сгенерировать пароль приложения переходим в настройки аккаунта → безопасность. Включаем двухэтапную аутентификацию.
    
    ![[Untitled 3 3.png|Untitled 3 3.png]]
    
3. По документации из гугла после этого шага внутри пункта “Двухэтапная аутентификация” должен появиться пункт “Пароли приложений”. Но в моём случае этого так и не произошло) Помогло перейти по прямой [ссылке](https://myaccount.google.com/apppasswords?pli=1&rapt=AEjHL4MCH_RP5c7IpVHIsFke5hYDgABDtapI-9MSKfT40Im9faV3ZD7FQFNLnfsv_5MckWWOjq_c4mMw5nOUuAVzY2AqJOR6Y9Z7Njj0jCybPss8VeaHTYY) для добавления паролей. Ссылка предоставлена [официальной поддержкой гугла](https://support.google.com/mail/thread/4477145/no-app-passwords-under-security-signing-in-to-google-panel?hl=en))
    
    ![[Untitled 4 3.png|Untitled 4 3.png]]
    
4. Добавляем в `settings.py`
    
    ```HTML
    EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_PORT = 587
    EMAIL_USE_TLS = True
    EMAIL_HOST_USER = 'your@gmail.com'
    EMAIL_HOST_PASSWORD = 'yourpassword'
    ```
    
5. Done! Well done!)

![[Untitled 5 3.png|Untitled 5 3.png]]

<div class="page-break" style="page-break-before: always;"></div>
