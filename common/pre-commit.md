# Как создать pre-commit hook в проекте на Git

pre-commit - это скрипт, который Git запускает перед исполнением команды "commit". Это позволяет автоматически запускать проверки перед каждым коммитом (lint, форматирование, тесты и т.д.), и если эти проверки не будут пройдены, коммит не будет создан.

## С использованием средств Git

Hook — это обычный shell-скрипт, который должен иметь специальное имя: по нему git сможет определить, при каком действии его запускать.
Hook должен находиться в .git\hooks. В этой папке уже есть примеры на каждое возможное действие (например, commit-msg.sample). Если убрать расширение sample — hook заработает.

Минусы:
- нужно версионировать папку .git
- читать shell-скрипты неудобно
- не кросс-платформенный подход (на Windows часто возникают проблемы)

Решение — утилита pre-commit, позволяет конфигурировать hook’и с помощью yaml-файла.

- установить библиотеку pre-commit:
```pip install pre-commit```
- настроить yaml-файл с названием .pre-commit-config.yaml, в котором нужно указать hook’и для использования. 
- ввести команду, которая создаст нужные файлы в .git\hooks:
```pre-commit install```
Также можно запустить все hook’и с помощью ```pre-commit run```.
Есть много готовых hook’ов на гитхабе (hook’и для поиска кредов, приватных ключей, исправления опечаток и т.д.)

## С помощью Husky

### Шаг 1. Установка husky
```npm install husky --save-dev```

### Шаг 2. Инициализация husky
```npx husky init```
Эта команда создаст директорию .husky и добавит скрипт prepare в package.json, который будет автоматически запускаться при установке пакетов

```
{
  "scripts": {
    "prepare": "husky install"
  }
}
```

### Шаг 3. Настройка скрипта pre-commit
При инициализации Husky в директории .husky будет создан файл pre-commit. В него можно добавлять нужные команды, например:

```
  #!/usr/bin/env sh
  . "$(dirname -- "$0")/_/husky.sh"

  npm run lint
  npm run format
```

Cкрипты lint и format должны быть определены в package.json:

```
  {
    "scripts": {
      "lint": "eslint .",
      "format": "prettier --write ."
    }
  }
```

Скрипт pre-commit также можно настроить с помощью команды
```npx husky add .husky/pre-commit "npm run lint && npm run format"```

С помощью lint-staged можно проверять только те файлы, которые попадают в коммит, что значительно ускоряет процесс

### Шаг 1. Установка
```npm install lint-staged --save-dev```

### Шаг 2. Настройка package.json

```
  {
    "lint-staged": {
      "*.{js,jsx,ts,tsx}": [
        "eslint --fix",
        "prettier --write"
      ]
    }
  }
```

### Шаг 3. Обновление скрипта pre-commit в .husky/pre-commit, чтобы использовать lint-staged

```
  #!/usr/bin/env sh
  . "$(dirname -- "$0")/_/husky.sh"

  npx lint-staged
```

или ```npx husky add .husky/pre-commit "npx lint-staged"```


При необходимости можно добавить в конце команды флаг --no-verify, тогда pre-commit не будет выполняться:
```git commit -m "The roof is on fire" --no-verify```
