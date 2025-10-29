# Copilot for Rider v2.0

**Версия:** 2.0
**Дата:** 2025‑10‑29
**Аудитория:** Разработчики, тимлиды, техлиды, интеграторы Unity/DI/UniRx/UniTask
**Цель:** создать единый стандарт работы с **GitHub Copilot** и **Copilot Chat** в JetBrains Rider — аналог Cursor-плейбука для Rider.

---

## 1) Быстрый старт (10 минут)

**Цель:** проверить установку, связать IDE с GitHub, прогреть контекст проекта.

1. **Установи плагины** в Rider:
   *File → Settings → Plugins → Marketplace → «GitHub Copilot» и «GitHub Copilot Chat» → Install.*
2. Войди в GitHub → Settings → Copilot → включи поддержку для Rider.
3. Проверь работу: создай файл и напиши `// suggest test method`, Copilot предложит автодополнение.
4. В проекте создай папку `/docs/copilot/` и добавь:

   * `ProjectBrief.md`, `Architecture.md`, `CodeStyle.md`, `DI.md`, `Testing.md`, `DoD.md`, `Prompts.md`, `Security.md`, `Glossary.md`.
5. Создай ветку `feat/copilot-bootstrap` и закоммить первые версии этих файлов.
6. Открой **Copilot Chat** (Alt+) и выполни пробный промпт:

   ```
   CONTEXT: см. @docs/copilot/Architecture.md
   TASK: перечисли основные слои и модули проекта.
   ```

---

## 2) Режимы работы Copilot

| Режим                                 | Когда использовать                         |     Время | Качество | Примечания                       |
| ------------------------------------- | ------------------------------------------ | --------: | -------: | -------------------------------- |
| **Inline Suggestion**                 | Быстрая генерация кода, автодополнение     | мгновенно |  среднее | Используй для boilerplate        |
| **Copilot Chat**                      | Обсуждение задач, рефакторинг, архитектура |  1–10 мин |  высокое | Поддерживает контекст проекта    |
| **/команды (refactor, fix, explain)** | При анализе кода                           |    быстро |  высокое | Можно указывать правила из @docs |

---

## 3) Контекст и структура проекта

**Copilot не имеет «Context Pack»,** но мы создаём аналог: `/docs/copilot/` — пакет знаний проекта.

Храним:

* **ProjectBrief.md** — цели и домен проекта.
* **Architecture.md** — слои, зависимости, паттерны.
* **CodeStyle.md** — стиль и нейминг.
* **DI.md** — правила внедрения зависимостей.
* **Testing.md** — структура тестов и coverage.
* **DoD.md** — Definition of Done.
* **Prompts.md** — шаблоны промптов Copilot Chat.
* **Security.md** — политика работы с секретами.
* **Glossary.md** — термины и сокращения проекта.

**Использование контекста:**

```
CONTEXT: см. @docs/copilot/Architecture.md и @docs/copilot/CodeStyle.md
TASK: Создать InventoryModule с DI и MVVM.
REQUIREMENTS: Data/Domain/Application/Presentation, Tests ≥ 70%.
SELF-CHECK: компиляция, тесты, зависимости.
```

---

## 4) Промпты и шаблоны Copilot Chat

### Шаблон «Создание Unity‑модуля (MVVM, DI)»

```
CONTEXT: Unity 2022.3, DI (method injection), UniRx, UniTask.
TASK: Создать [ModuleName] с функциями [описание].
REQUIREMENTS:
- Domain: интерфейсы/сущности без Unity API.
- Application: UseCases, Plain C#.
- Data: репозитории, без PlayerPrefs.
- Presentation: ViewModel (UniRx), Views (MonoBehaviour).
- Installers: BindInterfacesTo/AsSingle.
- Tests: NUnit; coverage ≥ 70%.
- AssemblyDefinitions: зависимости только вниз.
SELF-CHECK: компиляция, тесты, DI корректен.
```

### Шаблон «Рефакторинг»

```
CONTEXT: см. @docs/copilot/CodeStyle.md
TASK: оптимизировать код, убрать дубли, ввести интерфейсы.
REQUIREMENTS: без регрессий; добавить/обновить тесты.
SELF-CHECK: API совместим, тесты зелёные, покрытие сохранено.
```

### Шаблон «Bugfix»

```
CONTEXT: шаги воспроизведения и логи.
TASK: исправить ошибку и добавить тест, который падал до фикса.
SELF-CHECK: тест красный→зелёный, нет изменений вне scope.
```

---

## 5) Интеграция с Rider

* **Code Suggestions:** появляются inline по мере ввода.
* **Copilot Chat:** открывается через `Alt+\` или `View → Tool Windows → GitHub Copilot Chat`.
* **Commands:** `/explain`, `/refactor`, `/fix`, `/test`, `/docs`.
* **Live Templates:** добавь шаблоны промптов в `Settings → Editor → Live Templates → Add Template Group 'Copilot'`.

---

## 6) Качество и DoD

**Definition of Done:**

* Build без ошибок и предупреждений.
* Тесты зелёные, coverage ≥ 70%.
* DI и архитектура соблюдены.
* Комментарии к публичным API.
* Нет секретов в коде или логах.
* PR оформлен с описанием и скриншотами (если UI).

---

## 7) Лучшая практика промптов

| Тип задачи      | Формат                | Пример                                                    |
| --------------- | --------------------- | --------------------------------------------------------- |
| Создание модуля | контекст + требования | «Создай InventoryModule по @docs/copilot/Architecture.md» |
| Рефакторинг     | контекст + цель       | «/refactor согласно CodeStyle и DI.md»                    |
| Анализ          | `/explain`            | «/explain почему этот метод async»                        |
| Тестирование    | `/test`               | «/test добавь unit-тесты для этого класса»                |
| Исправление     | `/fix`                | «/fix null reference и добавь проверку на null»           |

---

## 8) Troubleshooting

* **Copilot не отвечает** → Проверь GitHub авторизацию.
* **Нет подсказок** → Убедись, что Copilot включён в статус-баре.
* **Неподходящие предложения** → Укажи контекст явно или добавь ссылки `@docs/copilot/...`.
* **Токены истекли** → Перевойди в GitHub.

---

## 9) Best/Worst кейсы

**Лучше всего:**

* Boilerplate код (DI, Installers, ViewModels).
* Рефакторинг шаблонных паттернов.
* Автогенерация тестов и документации.

**Избегать:**

* Изменений без контекста проекта.
* Оптимизаций без профилинга.
* Работы с безопасностью и сетью без ручного аудита.

---

## 10) Пример PR-шаблона (`.github/pull_request_template.md`)

```markdown
### Что сделано
- ...

### Как проверять
- ...

### Definition of Done
- [ ] Build + Tests OK
- [ ] DI и архитектура соблюдены
- [ ] Нет секретов
- [ ] Обновлены docs/ и CHANGELOG (если нужно)

### Скриншоты / Видео
```

---

## 11) Итог

Copilot в Rider — это аналог Cursor Agent + Chat, но с прямой интеграцией в IDE.
Поддерживай `/docs/copilot/` актуальным, используй шаблоны промптов и проводи ревью промптов раз в месяц.
Главное правило: **Copilot эффективен, если он обучен на контексте твоего проекта.**
