---
name: platform-symfony
description: Правила для кода в проектах Symfony этого репозитория. Использовать при любой работе с Symfony-кодом — контроллеры, сервисы, Doctrine, формы, конфигурация. Применять вместе с навыком workflow.
---

# Symfony: запреты, идиомы, проверки

Процесс работы задаёт навык `skills/workflow/SKILL.md`. Здесь — как
воплощать его именно в Symfony. Сверяй версии и детали с официальными
Symfony Best Practices для версии проекта (`composer.json`).

## Запреты

- НЕ клади бизнес-логику в контроллеры и шаблоны Twig. Контроллер
  тонкий: принял запрос → вызвал сервис → вернул ответ.
- НЕ обращайся к контейнеру напрямую (`$container->get()`);
  зависимости — только через конструктор (autowiring).
- НЕ пиши сырой SQL мимо Doctrine без явного решения заказчика
  (зафиксированного в плане).
- НЕ создавай God-сервисы "на всё": один сервис — одна ответственность.
- НЕ правь vendor/ никогда.
- НЕ подключай библиотеки, дублирующие компоненты Symfony
  (HTTP-клиент, валидация, сериализация, формы и т.п.).

## Идиомы (как делается типовое)

- Новая функциональность: сервис в `src/Service/` (или доменная папка
  проекта — посмотри, как уже сделано) + тонкий контроллер + Twig.
- Данные: сущность Doctrine + репозиторий; запросы — в репозитории,
  не в контроллере.
- Входные данные: Form/DTO + Validator (атрибуты/аннотации как принято
  в проекте).
- Конфигурация: параметры и DI через `config/`, секреты — через env.
- Перед созданием нового посмотри, как аналогичная вещь уже сделана
  в этом проекте, и повтори стиль. Эталонный модуль указан в
  `docs/MAP.md` (если указан).

### Так / не так

```php
// ТАК: тонкий контроллер
public function __construct(private OrderService $orders) {}
public function create(Request $r): Response {
    $order = $this->orders->createFromRequest($r);
    return $this->redirectToRoute('order_show', ['id' => $order->getId()]);
}

// НЕ ТАК: логика и SQL в контроллере
public function create(Request $r): Response {
    $conn = $this->getDoctrine()->getConnection();
    $conn->executeStatement("INSERT INTO orders ..."); // ❌
    // ...50 строк бизнес-логики... // ❌
}
```

## Автопроверки (гонять перед отчётом шага)

Используй конфиги репозитория; если их нет — предложи заказчику
добавить (отдельным шагом плана):

- `vendor/bin/php-cs-fixer fix --dry-run --diff`
- `vendor/bin/phpstan analyse` (с symfony-расширением)
- `php bin/console lint:twig templates/ && php bin/console lint:yaml config/`
- `php bin/console doctrine:schema:validate`
- `vendor/bin/phpunit`

Результаты — в раздел "Автопроверки" отчёта шага.
