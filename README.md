# Оптимізація обробки запитів в умовах високого навантаження

Цей репозиторій демонструє базовий приклад, де при 100 RPS (запитів за секунду) необхідно оптимізувати виклик повільної функції `BlackBox::getBar()`.

## Приклад

```php
class Foo
{
    // TODO: оптимізувати загальний час роботи при 100 rps
    public function getBar()
    {
        $result = Cache::get('bar');
        if ($result !== null) {
            return $result;
        }

        $result = BlackBox::getBar(); // 1s
        Cache::put('bar', $result);

        return $result;
    }
}
```

### Оптимізація

- Можливе використання Redis або Memcache (RAM) для кешування.
- Додати TTL для кешу:

```php
Cache::put('bar', $result, 600); // наприклад, 10 хвилин
```

### Архітектурне покращення

Потрібно перейти до **Event Driven Architecture**:

```php
class Foo
{
    public function getBar()
    {
        $result = Cache::get('bar');
        if ($result !== null) {
            return $result;
        }

        // Відправляємо завдання в чергу
        $result = ProcessBlackBox::dispatch(BlackBox::getBar());
        Cache::put('bar', $result);

        return $result;
    }
}
```

Використовуємо:
- **Event Broker**: RabbitMQ або Kafka
- **Async Queue Workers** для обробки

---

## План дій:

1. **Аналіз даних**:
   - Які дані, їх зв’язки, частота оновлення
   - Розмір, актуальність, життєвий цикл

2. **Архітектура зберігання**:
   - MySQL для структурованих даних
   - Redis для кешу
   - RabbitMQ для черг

3. **Інструменти**:
   - Queue Workers (Supervisor)
   - Пріоритизація задач
   - Моніторинг продуктивності черги
   - Кешування обчислень / API-запитів

---

## Патерни HighLoad:

- Кешування
- Горизонтальне / вертикальне масштабування
- Паралельна обробка
- Денормалізація
- Конвеєрна обробка

---

## Моніторинг та конфігурація

- Supervisor: обмежити `max-time`, `max-jobs`
- Пріоритети задач
- Аналіз черг: CPU, RAM, мережа
- Логування помилок і невдалих задач
