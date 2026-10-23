---
name: platform-wordpress
description: Правила для кода WordPress в этом репозитории — темы, плагины, хуки. Использовать при любой работе с WordPress-кодом. Применять вместе с навыком workflow.
---

# WordPress: запреты, идиомы, проверки

Процесс задаёт `skills/workflow/SKILL.md`. Здесь — идиомы WordPress.
ВАЖНО: не переноси сюда "современный PHP-стиль" (сервисные слои,
репозитории, DI-контейнеры) — это неидиоматично и усложнит поддержку
следующему WP-разработчику. Чистота здесь = следование WordPress
Coding Standards и API ядра.

## Запреты

- НЕ правь ядро WordPress, чужие темы и чужие плагины. Вся
  кастомизация — через хуки, дочернюю тему или собственный плагин.
- НЕ пиши запросы к БД мимо `$wpdb` / `WP_Query` / `get_posts()`.
  Через `$wpdb` — только с `$wpdb->prepare()`.
- НЕ выводи данные без экранирования: `esc_html()`, `esc_attr()`,
  `esc_url()`, `wp_kses_post()`.
- НЕ принимай ввод без санитизации (`sanitize_text_field()` и
  родственные) и без проверки nonce + capability
  (`check_admin_referer()`, `current_user_can()`).
- НЕ подключай jQuery/скрипты/стили напрямую в шаблоне —
  только `wp_enqueue_script()` / `wp_enqueue_style()`.
- НЕ изобретай крон, HTTP-клиент, опции, кэш: есть WP-Cron,
  `wp_remote_get()`, Options/Settings API, Transients.

## Идиомы (как делается типовое)

- Новое поведение: `add_action()` / `add_filter()` в своём плагине
  (`wp-content/plugins/<наш-плагин>/`) или в functions.php дочерней темы.
- Своя сущность: Custom Post Type + мета-поля
  (`register_post_type`, `register_post_meta`), не своя таблица,
  пока заказчик явно не решил иначе.
- Админ-страница: Settings API; AJAX — `wp_ajax_*` или REST API
  (`register_rest_route`) с permission_callback.
- Именование: snake_case функций с уникальным префиксом проекта.
- Переводимые строки: `__()` / `_e()` с текст-доменом проекта.
- Перед созданием нового посмотри, как аналогичное уже сделано
  в нашем плагине/теме, и повтори стиль.

### Так / не так

```php
// ТАК: хук + подготовленный запрос + экранирование
add_filter('the_title', 'myproj_mark_sale_title', 10, 2);
function myproj_mark_sale_title($title, $post_id) {
    if (get_post_meta($post_id, '_myproj_sale', true)) {
        $title .= ' ' . esc_html__('(акция)', 'myproj');
    }
    return $title;
}

// НЕ ТАК: правка файла темы-родителя и сырой SQL
// single.php родительской темы: $wpdb->query("UPDATE wp_posts ...") ❌
```

## Автопроверки (гонять перед отчётом шага)

- `vendor/bin/phpcs --standard=WordPress <путь>` (WordPress Coding
  Standards; конфиг в репо)
- `vendor/bin/phpstan analyse` (с расширением szepeviktor/phpstan-wordpress)
- Если настроено: `wp eval 'echo "ok";'` на тестовой площадке,
  PHPUnit с WP-тест-сьютом.

Результаты — в раздел "Автопроверки" отчёта шага.
