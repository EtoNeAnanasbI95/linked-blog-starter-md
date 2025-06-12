
![[2025-06-12-12-36-00-554.jpg]]

📡 apimocker — простой мокер REST API без бэкенда

apimocker — это легкий TUI-инструмент на Go, который поднимает фейковый API из YAML/JSON за секунды. Идеален для фронтенда, тестов и прототипов.

🛠 Что умеет:
• Динамичные JSON-ответы с шаблонами (`"id": "uuid"`, `"email": "email"`)
• Задержки, ошибки с вероятностью
• Отдача файлов (изображений и др.)
• Логирование (plain/json)
• TUI-интерфейс с активными маршрутами

💡 Подходит для:
• Быстрого мокинга
• Демонстраций и тестов
• Изоляции от реального API

🚀 Установка:
```

yay -S apimocker  # Arch
# или:
git clone https://github.com/Hanashiko/apimocker.git
cd apimocker && go build -o apimocker main.go
sudo mv apimocker /usr/bin/
```
🔗 https://github.com/Hanashiko/apimocker

Нужен API-сервер без сервера? Используй apimocker.