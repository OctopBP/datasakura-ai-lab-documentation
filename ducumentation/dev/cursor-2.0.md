# Cursor v2.0
**Версия:** 2.0  
**Дата:** 2025‑09‑04  
**Аудитория:** Разработчики, тимлиды, техлиды, интеграторы Unity/DI/UniRx/UniTask  
**Цель:** Дать единый, прикладной стандарт работы с Cursor (Agent/Chat/Background), MCP и Unity‑проектами (MVVM, DI), чтобы ускорять разработку без деградации качества и безопасности.

---
## Оглавление
1. Быстрый старт (10 минут)
2. Режимы Cursor: матрица выбора
3. Установка и окружение
4. Context Pack — пакет знаний проекта
5. Структура репозитория и модулей (Unity)
6. Политики ветвления, PR и CI
7. Сильные промпты: шаблоны и само‑проверки
8. Background‑задачи: протокол запуска
9. Unity‑специфика: MVVM, DI, UniRx/UniTask, Addressables
10. MCP: подключение, права, безопасность
11. Безопасность и приватность
12. Качество: Definition of Done (DoD)
13. Тестирование: NUnit/PlayMode/coverage
14. Стоимость/токены: управление контекстом
15. Best/Worst кейсы
16. Troubleshooting & FAQ
17. Чек‑листы (до Background, до merge)
18. Готовые шаблоны (PR, CODEOWNERS, MCP, prompts)

---
## 1) Быстрый старт (10 минут)
**Цель:** проверить установку, связать репозитории, прогреть контекст.

1. **Установите Cursor** (Windows/macOS/Linux) → **Sign in**.  
2. **Settings → Accounts:** войдите в GitHub/GitLab, разрешите доступ к нужным репозиториям.  
3. Откройте целевой проект в Cursor и дождитесь индексирования.  
4. создайте папку `/docs/cursor/` и добавьте минимум:
   - `ProjectBrief.md`, `Architecture.md`, `CodeStyle.md`, `UnityGuidelines.md`, `DI.md`, `Testing.md`, `DoD.md`, `Prompts.md`, `Security.md`.
5. Откройте **Agent**‑чат и выполните «пробный» промпт:
   ```
   Сканируй проект и перечисли верхнеуровневые директории. Подтверди, что видишь @docs/cursor/*.md
   ```
6. Создайте ветку `feat/cursor-playbook-bootstrap` и закоммитьте Context Pack.

---
## 2) Режимы Cursor: матрица выбора
| Режим | Когда использовать | Время | Качество | Примечания |
|---|---|---:|---:|---|
| **Agent** | Основная работа: код + рассуждения, задачи 10–60 мин | Среднее | Высокое | Текущая ветка, итеративная разработка |
| **Chat** | Быстрые вопросы/обсуждения, уточнения | Быстрое | Среднее | Не генерить крупные фичи |
| **Background** | Генерация модулей/проектов, массовый рефакторинг | Дольше | Очень высокое | Автоветка `bg/...`, PR, CI, строго заданный scope |

**Анти‑паттерны:** запуск Background без правил проекта; сложная оптимизация без профилинга; секреты в истории чата.

---
## 3) Установка и окружение
**Предпосылки:** доступ к GitHub/GitLab, VPN/прокси (если требуется), установленный Python/Node (для отдельных MCP), Slack (опционально для нотификаций).

- **Cursor → Settings → Accounts:** привяжите GitHub/GitLab.
- **Проверка проекта:** убедитесь, что Cursor индексирует и находит файлы.
- **Workspace Profile:** держите `/docs/cursor/` в репозитории (ниже — структура Context Pack).

---
## 4) Context Pack — пакет знаний проекта
Храним в `/docs/cursor/`:
- **ProjectBrief.md** — цель, домен, ключевые модули, библиотеки + версии.
- **Architecture.md** — слои, зависимости, правила модульности.
- **CodeStyle.md** — форматирование (EditorConfig), нейминг (PascalCase), суффиксы (`Service`, `View`, `ViewModel`, `Controller`, `Repository`, `Installer`).
- **UnityGuidelines.md** — Assembly Definitions, Addressables‑правила, ScriptableObjects, разделение Data/Domain/Application/Presentation, правило *Plain C# когда возможно*.
- **DI.md** — DI: method injection, Bindings, Installers, примеры.
- **Testing.md** — NUnit/PlayMode, coverage‑минимумы, структура папок `Tests/`.
- **DoD.md** — Definition of Done для любого таска (см. §12).
- **Prompts.md** — «золотые» шаблоны промптов (см. §7 и §18).
- **Security.md** — .env/secret manager, маскирование логов, запреты.
- **Glossary.md** — термины проекта для снижения неоднозначности.

В promt‑ах ссылаться как `@docs/cursor/Architecture.md`.

---
## 5) Структура репозитория и модулей (Unity)
**Единые правила для модульной архитектуры (MVVM, DI, SOLID):**
- Каждый модуль: папки **`Data/ Domain/ Application/ Presentation/ Installers/ Tests/`**.
- **Assembly Definitions** на каждый слой; зависимости строго вниз (Presentation → Application → Domain; Data — параллельно Application, без прямых ссылок из UI).
- Суффиксы классов: `*Service`, `*View`, `*ViewModel`, `*Controller`, `*Repository`, `*Installer`.
- **Запрещено:** PlayerPrefs напрямую (используем StorageModule); прямой доступ UI → Data; бизнес‑логика в MonoBehaviour.
- **Addressables:** только через AddressablesModule; никаких `Resources.Load`.
- **Installer‑ы:** регистрируют контракты интерфейсов, method injection в DI.
- **Tests:** Unit (Domain/Application), PlayMode (Presentation); быстрый запуск из CI.
- Метаданные модуля `module.json`: name, version, authors, dependencies, changelog.

---
## 6) Политики ветвления, PR и CI
- **Ветки:**
  - Фичи: `feat/<scope>`
  - Багфиксы: `fix/<scope>`
  - Background: `bg/<module>-<task>-<ticket>`
- **PR‑шаблон:** `.github/pull_request_template.md` (см. §18).
- **CODEOWNERS:** владелцы критичных областей (см. §18).
- **CI Checks:** build, unit/PlayMode tests, linters, code style, размер диффа.
- **Merge‑политика:** squash с conventional commits (`feat:`, `fix:`), обязательное описание миграций.

---
## 7) Сильные промпты: шаблоны и само‑проверки
**Базовая формула промпта:**
```
КОНТЕКСТ: [проект/стек/правила + ссылки @docs/...]
ЗАДАЧА: [что сделать и зачем]
ТРЕБОВАНИЯ: [список критериев]
СТИЛЬ/АРХИТЕКТУРА: [слои, DI, паттерны, нейминг]
ВХОД/ВЫХОД: [контракты интерфейсов, события]
ТЕСТЫ: [кейсы и где лежат]
НЕ ДЕЛАТЬ: [жёсткие запреты]
SELF-CHECK: [чек‑лист, который агент подтверждает]
```

**Шаблон: «Создание Unity‑модуля (MVVM, DI)»**
```
КОНТЕКСТ: Unity 2022.3, URP, DI (method injection), UniRx, UniTask.
Слои: Data/Domain/Application/Presentation/Installers; AssemblyDefinitions на слой.
См. @docs/cursor/Architecture.md, @docs/cursor/UnityGuidelines.md

ЗАДАЧА: Создать [ModuleName] с [функциями].

ТРЕБОВАНИЯ:
- Domain: интерфейсы/сущности (Plain C#), без Unity API.
- Application: сервисы/UseCases, Plain C#.
- Data: репозитории; без PlayerPrefs — StorageModule.
- Presentation: ViewModel (UniRx), Views (MonoBehaviour), биндинги.
- Installers: BindInterfacesTo/AsSingle, method injection.
- Tests: NUnit для Domain/Application, PlayMode для UI; coverage ≥ 70%.
- AssemblyDefinitions: зависимости только вниз.

НЕ ДЕЛАТЬ: статическая бизнес‑логика; прямые обращения UI→Data; Resources.Load.

SELF-CHECK:
- [ ] Компиляция ок
- [ ] Тесты зелёные, coverage ≥ 70%
- [ ] Installer регистрирует все интерфейсы
- [ ] Зависимости слоёв корректны
- [ ] Ограничения соблюдены
```

**Шаблон: «Рефакторинг»** и **«Bugfix»** — см. §18 (готовые вставки).

---
## 8) Background‑задачи: протокол запуска
**Перед запуском:**
1. Определить scope: пути, которые можно менять (напр. `only_touch: /Modules/Quest/*`).
2. Подготовить Context Pack и точный промпт (см. §7).
3. Создать ветку `bg/...`, активировать CI‑чеки.

**Запуск:** в режиме **Background**, промпт фиксирует scope, DoD, тесты.  
**Итерации:** отвечайте в том же чате; требуйте список изменённых файлов и мотивацию.  
**Финиш:** PR с чек‑листом DoD, ревью владельцами (CODEOWNERS).

---
## 9) Unity‑специфика: MVVM, DI, UniRx/UniTask, Addressables
- **MVVM:** ViewModel — Plain C# (UniRx), View — MonoBehaviour + подписки, без бизнес‑логики.
- **DI:** методическая инъекция; Installers по слоям; BindInterfacesTo/AsSingle; никаких сервис‑локаторов.
- **UniRx/UniTask:** реактивные потоки в VM/сервисах; async‑операции без блокировок UI; избегать горячих циклов.
- **Assembly Definitions:** слой на модуль; явные зависимости.
- **Addressables:** загрузка ресурсов только через AddressablesModule; управление памятью и ссылками централизовано.
- **Storage:** только через StorageModule; запрет PlayerPrefs.
- **Нейминг префабов/объектов:** PascalCase; без автогенерируемых имён; структура от общего к частному.

---
## 10) MCP: подключение, права, безопасность
**Пример конфигурации (минимально необходимые права):**
```json
{
  "mcpServers": {
    "Unity MCP": {
      "command": "uv",
      "args": ["--directory", "/usr/local/bin/UnityMCP/UnityMcpServer/src", "run", "server.py"],
      "env": {
        "UNITY_PROJECT_PATH": "./",
        "ALLOW_WRITE_PATHS": "Assets/Modules;ProjectSettings"
      }
    }
  }
}
```
**Правила:** минимум прав (least privilege), секреты через .env/CI‑vault, маскирование логов, dry‑run при поддержке, версионирование плагинов/серверов, whitelist операций (чтение сцен, ограниченная запись префабов, без удалений по умолчанию).

### Список MCP который ускоряют работу в unity:
1. Unity MCP: https://github.com/CoplayDev/unity-mcp
   Может управлять ассетами, контролировать сцены, редактировать скрипты и автоматизировать задачи в Unity.
   
   Установка:
   - Установите плагин в Unity: https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity
   - Перейдите в `Window -> Unity MCP -> Auto Setup` или добавьте конфигурацию вручную:
      ```json
      "unityMCP": {
         "command": "uv",
         "args": ["--directory","<ABSOLUTE_PATH_TO>/UnityMcpServer/src","run","server.py"],
         "type": "stdio"
       }
      ```
2. Task master: https://github.com/eyaltoledano/claude-task-master
   Позволяет планоровать задачи и выполнять их по очереди
   
   Установка:
   - Добавьте конфигурацию в cursor:
      ```json
      "task-master-ai": {
         "command": "npx",
         "args": ["-y", "task-master-ai"],
         "env": {
           "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY_HERE",
           "PERPLEXITY_API_KEY": "YOUR_PERPLEXITY_API_KEY_HERE",
           "OPENAI_API_KEY": "YOUR_OPENAI_KEY_HERE",
           "GOOGLE_API_KEY": "YOUR_GOOGLE_KEY_HERE",
           "MISTRAL_API_KEY": "YOUR_MISTRAL_KEY_HERE",
           "GROQ_API_KEY": "YOUR_GROQ_KEY_HERE",
           "OPENROUTER_API_KEY": "YOUR_OPENROUTER_KEY_HERE",
           "XAI_API_KEY": "YOUR_XAI_KEY_HERE",
           "AZURE_OPENAI_API_KEY": "YOUR_AZURE_KEY_HERE",
           "OLLAMA_API_KEY": "YOUR_OLLAMA_API_KEY_HERE"
         }
       }
      ```
      Ключи опциональны, если их добавить task master сможет профодить ресёрч на этапе планирования
3. Context 7: https://github.com/upstash/context7
   Позволяет cursor получать доступ к большой базе данных документаций
   
   Установка:
   - Добавьте конфигурацию в cursor:
      ```json
      "context7": {
        "args": ["-y", "@upstash/context7-mcp"],
        "command": "npx"
      }
      ```
     
---
## 11) Безопасность и приватность
- Не вставлять ключи/токены в чаты.
- Секреты — только в Secret Manager/CI‑vault.
- Маскирование в логах; запрет дампов пользовательских данных.
- Ревью кода, касающегося сети/файлов/криптографии.
- Политика датасетов: анонимизация, лицензии, хранение.

---
## 12) Качество: Definition of Done (DoD)
**Единый DoD для задач:**
- Компиляция без ошибок/предупреждений.
- Тесты (unit/PlayMode) зелёные; coverage ≥ X% (по проекту).
- Статический анализ/линтеры без новых ошибок.
- Архитектура и DI соблюдены; Installers актуальны.
- Документация: комментарии публичных API, CHANGELOG при необходимости.
- Перформанс: без деградаций ключевых метрик (если затрагивается runtime).
- Безопасность: нет секретов/лишних прав, MCP‑ограничения соблюдены.
- PR оформлен, приложены скриншоты UI (если есть изменения UI).

---
## 13) Тестирование: NUnit/PlayMode/coverage
- **Unit:** Domain/Application, изолированные от Unity API.
- **PlayMode:** Presentation/UI, жизненные сценарии, биндинги.
- **Структура:** `ModuleX/Tests/Unit`, `ModuleX/Tests/PlayMode`.
- **CI:** быстрый запуск, артефакты отчётов, пороги покрытия.

---
## 14) Стоимость/токены: управление контекстом
- В **Agent/Chat** — короткие промпты, ссылки на Context Pack вместо длинных вставок.
- В **Background** — строгая постановка, ограничения по путям, «план → код → тесты».
- Поддерживайте «золотые» промпты в `Prompts.md` — переиспользуйте.

---
## 15) Best/Worst кейсы
**Лучше всего:**
- Генерация boilerplate (CRUD, Installers, ViewModel‑прослойки), шаблонных UI.
- Рефакторинг повторяющихся паттернов; написание автотестов.
- Миграции однотипных мест (переезд на AddressablesModule/StorageModule).

**Осторожно:**
- Низкоуровневая оптимизация без профилинга.
- Безопасность/криптография/сеть — нужен ручной аудит.
- Изменения вне заданного scope.

---
## 16) Troubleshooting & FAQ
- **«Ссылается на несуществующие API»** → попросите список импортов/версий; приложите ссылки на оф. доки; потребуйте «реестр методов».
- **«Меняет файлы вне задачи»** → ограничьте область в промпте (`only_touch`), проверяйте список изменённых файлов.
- **«MonoBehaviour для бизнес‑логики»** → перегенерация: логика в Plain C#, View отдельно.
- **«Лезет в PlayerPrefs»** → жёсткий запрет; использовать StorageModule.
- **«Горячие циклы в UniTask»** → переписать на реактивные потоки/await без busy‑loop.

---
## 17) Чек‑листы
**Перед запуском Background**
- [ ] Ветка `bg/...` создана, CI включен.
- [ ] Context Pack в `/docs/cursor/` актуален.
- [ ] Промпт с DoD и ограничениями по путям готов.
- [ ] MCP ограничён правами; dry‑run (если есть).
- [ ] Включены нотификации (Slack/Email) на PR.

**Перед merge**
- [ ] Build OK, тесты зелёные, coverage ≥ X%.
- [ ] Архитектура/DI/Installers соблюдены.
- [ ] Нет секретов и лишних изменений.
- [ ] PR описан, скриншоты UI приложены.

---
## 18) Готовые шаблоны
### 18.1 PR‑шаблон (`.github/pull_request_template.md`)
```markdown
### Что сделано
- ...

### Как проверять
- ...

### Definition of Done
- [ ] Build + Tests OK
- [ ] Архитектурные слои и DI соблюдены
- [ ] Нет секретов в диффе
- [ ] Обновлены docs/ и CHANGELOG (если нужно)

### Скриншоты / Видео
```

### 18.2 CODEOWNERS (пример)
```
/Modules/Quests/*        @team/gameplay @lead-arch
/Modules/Storage/*       @team/platform
/Assets/Addressables/*   @team/content
```

### 18.3 MCP‑конфиг (минимум прав)
```json
{
  "mcpServers": {
    "Unity MCP": {
      "command": "uv",
      "args": ["--directory", "/usr/local/bin/UnityMCP/UnityMcpServer/src", "run", "server.py"],
      "env": {
        "UNITY_PROJECT_PATH": "./",
        "ALLOW_WRITE_PATHS": "Assets/Modules;ProjectSettings"
      }
    }
  }
}
```

### 18.4 Шаблон промпта «Создание модуля» (короткая версия)
```
КОНТЕКСТ: Unity 2022.3, DI, UniRx, UniTask. См. @docs/cursor/*
ЗАДАЧА: Создать [ModuleName] с [функциями].
ТРЕБОВАНИЯ: слои, AssemblyDefinitions, Tests (coverage ≥ 70%), Installers.
НЕ ДЕЛАТЬ: PlayerPrefs, Resources.Load, логика в MonoBehaviour.
ОГРАНИЧЕНИЯ: изменять только /Modules/[ModuleName]/*
SELF-CHECK: компиляция, тесты, зависимости, Installer, PR‑заметки.
```

### 18.5 Шаблон промпта «Рефакторинг»
```
КОНТЕКСТ: Код в /Modules/[X]/[paths], следуем @docs/cursor/CodeStyle.md
ЗАДАЧА: Удалить дубли, ввести интерфейсы, упростить зависимости.
ТРЕБОВАНИЯ: без функциональных регрессий; добавить/обновить тесты; измерить метрики до/после.
SELF-CHECK: API совместим, покрытие не ниже, нет глобальных стейтов.
```

### 18.6 Шаблон промпта «Bugfix»
```
КОНТЕКСТ: [что ломается], шаги воспроизведения, ожидаемое/фактическое, логи.
ЗАДАЧА: Найти корень, пофиксить, добавить тест, который падал до фикса и проходит после.
SELF-CHECK: тест красный→зелёный, нет побочных эффектов, нет изменений вне scope.
```

### 18.7 Блок «Self‑Check» для агента
```
Подтвердите перед выдачей:
- [ ] Список изменённых файлов + зачем каждый
- [ ] Краткая диаграмма зависимостей слоёв
- [ ] Покрытие тестами ключевых кейсов
- [ ] Ничего не тронуто вне заданного scope
```

---
