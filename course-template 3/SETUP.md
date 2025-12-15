# 🛠️ Инструкция для куратора

Как настроить репозитории для участников курса.

---

## Предварительная настройка (один раз)

### 1. Создайте GitHub Organization

Если ещё нет:
1. GitHub → `+` → `New organization`
2. Выберите Free план
3. Название: например `your-course-name`

### 2. Добавьте Organization Secret

```
Organization → Settings → Secrets and variables → Actions → New organization secret
```

| Поле | Значение |
|------|----------|
| Name | `ANTHROPIC_API_KEY` |
| Value | `sk-ant-api03-...` |
| Repository access | `All repositories` или `Selected repositories` |

Теперь все репозитории в организации будут использовать этот ключ.

### 3. Загрузите Template Repository

1. Создайте новый репозиторий в организации: `course-template`
2. Сделайте его **Template repository**:
   - Settings → General → ✅ Template repository
3. Загрузите туда все файлы из этого шаблона

---

## Создание репозитория для участника

### Вариант A: Через GitHub UI

1. Откройте template repository
2. Нажмите `Use this template` → `Create a new repository`
3. Настройки:
   - Owner: `your-organization`
   - Name: `participant-ivanov` (имя участника)
   - ✅ **Private**
4. Нажмите `Create repository`
5. Добавьте участника:
   - Settings → Collaborators → Add people
   - Роль: `Write` (чтобы мог создавать PR)

### Вариант B: Через CLI (быстрее для множества участников)

```bash
# Установите GitHub CLI если нет
# https://cli.github.com/

# Создание репозитория
gh repo create your-org/participant-ivanov \
  --template your-org/course-template \
  --private

# Добавление участника
gh api repos/your-org/participant-ivanov/collaborators/username \
  -X PUT \
  -f permission=push
```

### Вариант C: Скрипт для массового создания

```bash
#!/bin/bash
# create_repos.sh

ORG="your-organization"
TEMPLATE="$ORG/course-template"

# Список участников: username,repo_name
PARTICIPANTS=(
  "ivanov,participant-ivanov"
  "petrov,participant-petrov"
  "sidorov,participant-sidorov"
)

for entry in "${PARTICIPANTS[@]}"; do
  IFS=',' read -r username repo_name <<< "$entry"
  
  echo "Creating repo for $username..."
  
  # Создаём репо из шаблона
  gh repo create "$ORG/$repo_name" \
    --template "$TEMPLATE" \
    --private
  
  # Добавляем участника
  gh api "repos/$ORG/$repo_name/collaborators/$username" \
    -X PUT \
    -f permission=push
  
  echo "✅ Done: $repo_name"
done
```

---

## Чеклист для каждого участника

- [ ] Репозиторий создан из шаблона
- [ ] Репозиторий приватный
- [ ] Участник добавлен как collaborator (Write)
- [ ] Organization secret доступен (проверить в Settings → Secrets)
- [ ] Участник получил ссылку на репозиторий

---

## Проверка работы

### Тест GitHub Action

1. Склонируйте репозиторий участника
2. Создайте тестовую ветку
3. Добавьте файл `texts/test.md`
4. Создайте PR
5. Проверьте что Action запустился и оставил комментарий

### Логи Action

```
Repository → Actions → Последний запуск → Логи
```

---

## Структура Organization

После настройки:

```
your-organization/
├── course-template          # Template (публичный или приватный)
├── participant-ivanov       # Приватный
├── participant-petrov       # Приватный
└── participant-sidorov      # Приватный
```

Каждый участник видит только свой репозиторий.

---

## Стоимость

### Anthropic API

~$0.05 за один SWOT-анализ (Claude Sonnet).

При 10 участниках × 5 заданий × 3 итерации = 150 анализов = **~$7.50**

### GitHub

- Organization Free: бесплатно
- Private repos: бесплатно (до 3 collaborators на repo)
- Actions: 2000 минут/месяц бесплатно

---

## Troubleshooting

### Action не запускается

1. Проверьте что файл в `texts/` и `.md`
2. Проверьте что PR создан (не push в main)
3. Проверьте логи: Actions → Failed run

### "ANTHROPIC_API_KEY not set"

Organization secret не доступен репозиторию:
1. Organization → Settings → Secrets → ANTHROPIC_API_KEY
2. Repository access → добавьте репозиторий

### Участник не видит репозиторий

1. Проверьте что добавлен как collaborator
2. Участник должен принять приглашение (email или GitHub notifications)

### Бот не оставляет комментарий

Проверьте permissions в workflow:
```yaml
permissions:
  contents: write
  pull-requests: write
```

---

## Обновление шаблона

Если нужно обновить задания или скрипты:

1. Обновите `course-template`
2. Для существующих репозиториев:
   - Вручную скопировать изменения
   - Или пересоздать репозиторий

---

## Контакты

[Ваши контакты для вопросов]
