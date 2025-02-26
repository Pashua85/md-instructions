# инструкция по работе с микрофронтендами

Все данные о работе с микрофронтами хранятся в классе `MicrofrontendsStore`, который доступен для импорта по пути `@evolenta/processes-module/mf-store`.


## Добавление нового микрофронтенда для работы:

1. **Настройка конфигурации микрофронтенда 

  - В созданном приложении мирофронта устанавливаем порт и хост, где он будет крутиться в файле `angular.json` (пример для локальной работы)

  ```javascript
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

  - В конфигурации webpack указываем какие компоненты и модули отдает микрофронт:

  ```javascript
    /** webpack.config.js */
    const {
      shareAll,
      withModuleFederationPlugin,
    } = require("@angular-architects/module-federation/webpack");

    module.exports = withModuleFederationPlugin({
      name: "modeler-app",

      exposes: {
        './MyModule': './src/app/my-module/my-module.module.ts',
        './Component1': './src/app/component1/component1.component.ts',
        './Component2': './src/app/component2/component2.component.ts',
      },

      shared: {
        ...shareAll({
          singleton: true,
          strictVersion: true,
          requiredVersion: "auto",
        }),
      },
    });
  ```