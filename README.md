# инструкция по работе с микрофронтендами

Все данные о работе с микрофронтами хранятся в классе `MicrofrontendsStore`, который доступен для импорта по пути `@evolenta/processes-module/mf-store`.


## Добавление нового микрофронтенда для работы:

**Настройка конфигурации микрофронтенда** 

  - В созданном приложении микрофронта устанавливаем порт и хост, где он будет крутиться в файле `angular.json` (пример для локальной работы)

  ```javascript
    /** angular.json в приложении микрофронта */ 
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
    /** webpack.config.js в приложении микрофронта */
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

  **ВАЖНО: надо следить за совпадением версий библиотек в микрофронте и shell-приложении**

## Добавление данных о новом микрофронтенде в MicrofrontendsStore

  В данный момент все файлы для организации работы с микрофронтендами находятся в библиотеке `"@evolenta/processes-module"`

1. **Нужно дополнить enum `RemoteComponents` названиями новых удаленных компонентов:**

  ```javascript
    export enum RemoteComponents {
      ...
      EXAMPLE_REMOTE_COMPONENT = 'ExampleRemoteComponent',
      SOME_OTHER_REMOTE_COMPONENT = 'SomeOtherRemoteComponent',
    }
  ```
  
2. **Нужно добавить данные о новом микрофронтенде в константу `data`, которую `MicrofrontendsStore` использует в качестве источника данных:**

  ```javascript
  export const data: Microfrontend[] = [
    {
        webpackMfeName: 'modeler-app',
        loadPath: 'http://localhost:4202/remoteEntry.js',
        modules: [
            /**
             * Данные нужны для создания константы, передаваемой root-приложением 
             * по DI под токеном MICROFRONTENDS_TOKEN 
             */
            {
                uniqueKey: 'microfrontend',
                selectName: 'Демо микрофронт',
                exposedModule: 'Module',
                entryPoint: 'Mfe1Module',
            },
        ],
        components: [
            {
                uniqueKey: RemoteComponents.EXAMPLE_REMOTE_COMPONENT,
                /** Внимание! название компонента из mf указывается без "./" 
                 * в отличие от ключа exposes в самом микрофронте */
                exposedModule: 'Component1',
            },
            {
                uniqueKey: RemoteComponents.SOME_OTHER_REMOTE_COMPONENT,
                exposedModule: 'Component2',
            },
        ],
    },
  ];

  ```