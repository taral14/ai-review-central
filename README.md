# ai-review-central

Центральні правила і reusable workflow для open-code-review. Один репозиторій на організацію, жодного дублювання правил у сервісах.

## Три рівні правил

| Рівень | Де живе | Хто змінює |
|---|---|---|
| Організація | `rules/org.rule.json` | платформенна команда, через PR сюди |
| Група репозиторіїв | `rules/<profile>.rule.json`, наприклад `php-services`, `frontend` | власники групи, через PR сюди |
| Один репозиторій | `.opencodereview/rule.json` у самому репозиторії | команда репозиторію |

Під час ревʼю три файли зливаються в один: записи репозиторію йдуть першими, потім профіль, потім організація. open-code-review бере перше правило, чий glob збігся з файлом, тому порядок і є пріоритетом. Списки `exclude` об'єднуються. Локальний файл читається з base-гілки, PR не може підмінити правила, за якими його ревʼюють.

## Підключити репозиторій

Один файл `.github/workflows/ai-review.yml` у репозиторії:

```yaml
name: AI review
on:
  pull_request_target:
    types: [opened, reopened, ready_for_review]
  issue_comment:
    types: [created]
permissions:
  contents: read
  pull-requests: write
jobs:
  review:
    uses: <owner>/ai-review-central/.github/workflows/ai-review.yml@main
    with:
      profile: php-services        # або frontend, або прибрати рядок для org-правил
    secrets: inherit
```

Це все. Виключення (draft, бот-автори, `chore(deps)`, лейбл `skip-ai-review`, форки), версія інструмента, модель і бюджет живуть тут і змінюються одним PR для всіх.

## Секрети та змінні

Задаються на рівні організації один раз, або в кожному репозиторії для тесту:

| Ім'я | Тип | Що |
|---|---|---|
| `LLM_OPENAI_BASE` | variable або secret | OpenAI-сумісна база, закінчується на `/v1` |
| `LLM_MODEL` | variable або secret | модель, яку endpoint приймає в полі `model` |
| `LLM_API_KEY` | secret | ключ |
| `CENTRAL_REPO_TOKEN` | secret, лише якщо цей репозиторій приватний | fine-grained PAT з правом Contents: read на цей репозиторій |
| `AI_REVIEW_DISABLED` | variable, необов'язково | `true` вимикає ревʼю для org або репозиторію |

## Доступ до reusable workflow

- Цей репозиторій **публічний**: нічого налаштовувати не треба.
- **Приватний**: Settings → Actions → General → Access → «Accessible from repositories in the organization» (для персонального акаунта «owned by the user»). Плюс `CENTRAL_REPO_TOKEN`, бо `GITHUB_TOKEN` caller-а не читає файли іншого приватного репозиторію.

## Перевизначення для одного репозиторію

Через `with:` у caller: `model`, `max_tokens_budget`, `effort`, `route_severity_below`. Через файл `.opencodereview/rule.json` у репозиторії: правила і виключення шляхів.

## Оновлення інструмента

Версія `open-code-review` запінена в `env.OCR_VERSION` і в `uses:`. Оновлюється одним PR сюди. Автооновлення npm-обгортки вимкнене через `OCR_NO_UPDATE=1`.
