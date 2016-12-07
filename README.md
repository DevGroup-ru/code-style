# Соглашения по написанию кода для репозиториев DevGroup

- [Стиль кода](#Стиль-кода)
- [Миграции](#Миграции)
- [Таблицы](#Таблицы)
- [Переводы](#Переводы)
- [Контроллеры для backend](#Контроллеры-для-backend)
- [Модули](#Модули)
- [Тестирование](#Тестирование)
- [Git](#git)

## Стиль кода

Код должен писаться в соответствии со стандартами PSR-0, PSR-1, PSR-2, PSR-3, PSR-4. Описание стандартов [здесь](https://github.com/php-fig/fig-standards/tree/master/accepted).

Кроме этого Yii содержит ряд своих [правил](https://github.com/yiisoft/yii2/blob/master/docs/internals/core-code-style.md).

Для облегчения проверки кода можно настроить сниффер со стандартом Yii2. Самый простой способ установки - композер. Просто выполните консольную команду `composer global require yiisoft/yii2-coding-standards "~2.0.0"`.

Теперь для проверки кода достаточно выполнить

```bash
~/.composer/vendor/squizlabs/php_codesniffer/scripts/phpcs --extensions=php --standard=~/.composer/vendor/yiisoft/yii2-coding-standards/Yii2/ path/to/project/sources
```

Для автоматического рефакторинга можно выполнить

```bash
~/.composer/vendor/squizlabs/php_codesniffer/scripts/phpcbf --extensions=php --standard=~/.composer/vendor/yiisoft/yii2-coding-standards/Yii2/ path/to/project/sources
```

> **Важно!** С авторефакторингом надо работать аккуратно, поэтому рекомендую следующий алгоритм.

1. Коммитим изменения;
2. Запускаем авторефакторинг;
3. Проверяем тестами или руками;
4. Добавляем изменения в предыдущий коммит (`git commit --amend -a`) или создаем новый;
5. Пушим на сервер.

Более крутой вариант - настроить PhpStorm. Для этого

1. `File->Settings->Languages & Frameworks->PHP->Code Sniffer->Configuration->...` В открывшемся окне в поле PHP code sniffer path указываем `/path/to/home/user/directory/.composer/vendor/squizlabs/php_codesniffer/scripts/phpcs`;
2. `File->Settings->Editor->Inspections->PHP->PHP Code Sniffer validation`. Отмечаем галочкой и в `Coding standard` выбираем `Custom` и нажимаем `...`. Там указываем путь `/path/to/home/user/directory/.composer/vendor/yiisoft/yii2-coding-standards/Yii2/`;
3. Применяем изменения. Теперь редактор будет подсвечивать все ошибки.

Автоматический рефакторинг можно вызвать прямо из главного меню IDE `Code->Reformat Code`.

В случае необходимости написания sql-запросов рекомендуется разделять его на несколько строк для удобочитаемости.

Пример:

```sql
# Пример №1
SELECT DISTINCT
    `id`, 
    `name`,
    `parent_id` -- если полей мало - можно в одну строку
FROM
    `table_name` `tn`
JOIN
    `join_table` `jt`
    ON
        `tn`.`id` = `jt`.`join_id`
WHERE
    (
        `tn`.`parent_id` = 12
        AND `tn`.`param` LIKE 'slug%'
    )
    OR `tn`.`id` IN
    (
        subquery
    )
GROUP BY
    `tn`.`slug`
HAVING
    count(*) > 1;

# Альтернатива
SELECT DISTINCT `id`, `name`, `parent_id`
FROM `table_name` `tn`
JOIN `join_table` `jt` ON `tn`.`id` = `jt`.`join_id`
WHERE
    (
        `tn`.`parent_id` = 12
        AND `tn`.`param` LIKE 'slug%'
    )
    OR `tn`.`id` IN
    (
        subquery
    )
GROUP BY `tn`.`slug`
HAVING count(*) > 1
```

В коде представлений и шаблонов необходимо использовать [альтернативный синтаксис](http://php.net/manual/ru/control-structures.alternative-syntax.php) управляющих структур. Использование стандартных допускается только в верхнем блоке файла до любого вывода html.

Пример:

```php
<?php
/**
 * @var array $data
 * @var string $header
 * @var \yii\web\View $this
 */

$items = [];
foreach ($data as $row) {
	$item[] = [
		'label' => $row['label'],
		'minCount' => min($row['numbers']),
	];
}
?>
<div id="content">
	<h1><?= $header ?></h1>
	<ul>
		<?php foreach ($items as $item): ?>
		<li><?= $item['label'] ?> <span class="count"><?= $item['minCount'] ?></span></li>
		<?php endforeach; ?>
	</ul>
</div>

```

## Миграции

Название миграции должно отображать ее суть и начинаться с названия расширения или модуля. Для выделения отдельных слов используется знак подчеркивания. Но важно помнить, что максимальная длина названия миграции 180 символов включая префикс с датой и временем.

Примеры:

- m151225_140300_users_module_create_administrator_user
- m151225_140300_users_module_update_user_table
- m151225_140300_events_system_init
- m151225_140300_events_system_update_structure

## Таблицы

Название таблицы БД должно отображать ее суть и начинаться с названия расширения или модуля.

- Названия таблиц и ее колонок должны быть написан в единственном числе;
- Для выделения отдельных слов в названии таблиц и колонок используется знак подчеркивания;
- Все расширения типа dotplant-extension должны начинаться с префикса `dotplant_`, а все расширения типа `yii2-extension` в рамках организации DevGroup должны начинаться с `devgroup_`;
- Таблицы должны иметь возможность добавления префикса через конфигурацию компонента БД;
- Таблицы связей должны содержать названия связываемых страниц;
- Колонки связывающие с другой таблицей должны содержать название внешней таблицы и название ее колонки;
- Везде, где это возможно, необходимо использовать внешние ключи с режимом `CASCADE`. Это избавит от необходимости удаления связанных записей средствами php.

Примеры:

- {{%dotplant_user}}
- {{%dotplant_currency}}
- {{%devgroup_measure}}
- {{%dotplant_product_category}}

## Переводы

- Везде, где используется вывод текста пользователю, неодходимо использовать метод `Yii::t()`;
- Для каждого расширения должна быть использовата своя категория перевода;
- Категория перевода состоит из названия расширения;
- Переводы для расширений хранятся в файле с соответствующим название и бутстрапятся в приложение.

Примеры:

- dotplant.users
- devgroup.measure

## Контроллеры для backend

- Если расширение имеет один контроллер для редактирования, то он должен называться ManageController;
- Если расширение имеет несколько контроллеров для редактирования, то к названию добавляется префикс в виде названия модели;
- Роут до экшена должен содержить название модуля;
- Название модуля в конфигурационном файле и название модели в наименовании контроллера должны быть во множественном числе если это возможно.

Примеры роутов:

- /measures/manage
- /users/users-manage
- /users/rbac-manage


## Модули

- создание экземпляров модулей и компонентов рекомендуется выполнить через `Yii::createObject('Vendor\Package\ModuleClass', ['ModuleId'])` вместо `new MyModule()`.

## Тестирование

- рекомендуется писать тесты одновременно с кодом;
- рекомендуется сразу добавить `.travis.yml` для автоматического тестирования всех коммитов и пулл-реквестов;
- предпочтительно использование подхода TDD;
- рекомендуется использование фикстур вместо дампа из sql;
- не рекомендуется запускать и отменять миграции в коде теста. Миграции должны быть применены до начала работы с тестами.


## Git

- один коммит должен решать одну задачу;
- обязательно использование информативных заголовков коммитов;
- заголовок коммита (первая строка commit message'а) **должен** быть в повелительном наклонении (никаких `fixed`, `added` etc). В больинстве случаев подойдёт такой подход: заголовок коммита должен служить продолжением предложения "If applied, this commit will " (`fix bug #123`, `add ping command`, `refactor shop/product/list.php view`). Больше информации и обоснований [тут](http://chris.beams.io/posts/git-commit/)
- коммиты не должны содержать автоматических коммитов наподобие `Merge branch 'branch-name' of github.com:user-name/repo-name`. Для этого отлично подходит [git flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow/) или хотябы `git stash`;
- перед коммитом необходимо проверить работоспособность с помощью тестов;
- issue должен обязательно содержать описание проблемы с примером;
- pull request должен обязательно содержать описание проблемы с примером и ее решение;
- описания к коммитам должны быть на русском языке для внутренних проектов и на английском для общих.
