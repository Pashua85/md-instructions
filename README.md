# инструкция по работе с микрофронтендами

Все данные о работе с микрофронтами хранятся в классе `MicrofrontendsStore`, который доступен для импорта по пути `@evolenta/processes-module/mf-store`.


## Добавление нового микрофронтенда для работы:

1. **Настройка конфигурации микрофронтенда 

  - В созданном приложении мирофронта устанавливаем порт и хост, где он будет крутиться в файле `angular.json` (пример для локальной работы)

  ```json
    /** angular.json */ 
    {
      ...
      "projects": {
        "example-mf-app": {
          ...
          "architect": {
            ...
            "serve": {
              ...
              "options": {
                "port": 4202,
                "publicHost": "http://localhost:4202",
                ...
              }
            }
            ...
          }
        }
      }
    }
  ```

---
title: The Lottery Ticket
author: Anton C.
date: "2013-03-15 15:00"
template: article.jade
tags:
  - Fiction
  - Russian

---