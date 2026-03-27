# AbortController
Вот пример использования `AbortController` в TypeScript для отмены асинхронного запроса:

```typescript
// Создаем новый AbortController
const controller = new AbortController();
const signal = controller.signal;

// Функция для выполнения асинхронного запроса
async function fetchData(url: string) {
    try {
        const response = await fetch(url, { signal });
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        console.log(data);
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log('Запрос был отменен');
        } else {
            console.error('Произошла ошибка:', error);
        }
    }
}

// Запускаем запрос
fetchData('https://api.example.com/data');

// Отмена запроса через 2 секунды
setTimeout(() => {
    controller.abort();
}, 2000);
```

### Объяснение кода:
-  Создается экземпляр `AbortController`, который позволяет отменять запросы.
-  Используется свойство `signal` для передачи сигнала отмены в функцию `fetch`.
-  Если запрос отменяется, ловится ошибка с именем `AbortError`, и выводится соответствующее сообщение.