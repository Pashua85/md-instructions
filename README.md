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

1. **Дополнить enum `RemoteComponents` названиями новых удаленных компонентов:**

  ```javascript
    /** projects/processes-module/src/mf-store/enums/remote.component.enum.ts */
    export enum RemoteComponents {
      ...
      EXAMPLE_REMOTE_COMPONENT = 'ExampleRemoteComponent',
      SOME_OTHER_REMOTE_COMPONENT = 'SomeOtherRemoteComponent',
    }
  ```
  
2. **Добавить данные о новом микрофронтенде в константу `data`, которую `MicrofrontendsStore` использует в качестве источника данных:**

  ```javascript
  /** projects/processes-module/src/mf-store/data.ts */
  export const data: Microfrontend[] = [
    {
        webpackMfeName: 'modeler-app',
        /** для работы с удаленной, а не локальной версий микрофронтенда надо поменять путь здесь */
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

3. **Указать входные данные удаленных компонентов и их типы в интерфейсе `ComponentInputMap`:**

  В случае отсутствия инпутов у компонента передать в `ComponentInputMap` пустой объект.

  ```javascript
  /** projects/processes-module/src/mf-store/interfaces/component-input-map.interface.ts */
  import { RemoteComponents } from '../enums';

  /** Входные параметры для компонентов */
  export interface ComponentInputMap {
    ...
    [RemoteComponents.EXAMPLE_REMOTE_COMPONENT]: {
        model: any;
        readonly: boolean;
        changeType: boolean;
    },
    [RemoteComponents.SOME_OTHER_REMOTE_COMPONENT]: {}
  }
  ```

4. **Указать обработчики выходных событий удаленных компонентом, расширив интерфейс `ComponentOutputHandlersMap`**

  По факту важен именно тип аргумента функции-обработчика. В случае отсутствия выходных событий 
  у компонент нужно передать пустой объект.

  ```javascript
    /** Обработчики выходных событий компонента */
    /** projects/processes-module/src/mf-store/interfaces/component-output-map.interface.ts */
    export interface ComponentOutputHandlersMap {
      ...
      [RemoteComponents.EXAMPLE_REMOTE_COMPONENT]: {
          onClose: () => OutputHandlerReturnValueType;
          onDownloadResult?: (event: string) => OutputHandlerReturnValueType;
          onGetResult: (event: IBpmnResult) => OutputHandlerReturnValueType;
      },
      [RemoteComponents.SOME_OTHER_REMOTE_COMPONENT]: {}
    } 
  ```

## Работа `MicrofrontendsStore` в основном приложении (оркестраторе).

Это уже добавлено в текущее приложение. В случае создания нового shell-приложения, работающего с микрофронтендами 
через стор, нужно сделать аналогично.

1. **Добавление `remotes` в конфигурации `webpack`**

  ```javascript
    /** extra-webpack.config.ts */
    const { withModuleFederationPlugin } = require('@angular-architects/module-federation/webpack');
    const { MicrofrontendsStore } = require('@evolenta/processes-module/mf-store');

    const mfStore = new MicrofrontendsStore();
    const remotes = mfStore.getWebpackConfig() ?? {}; 

    const mfConfig = withModuleFederationPlugin({
        remotes,
        ...
    });
    ...
  ```

2. **Добавление в провайдеры данные об удаленных модулях микрофронтов, доступные в DI по всему приложению под токеном `MICROFRONTENDS_TOKEN`**

  ```javascript
    /** app.module.ts */
    import { MICROFRONTENDS_TOKEN } from '@evolenta/core';
    import { MicrofrontendsStore } from '@evolenta/processes-module/mf-store';

    const mfStore = new MicrofrontendsStore();
    const mfModules = mfStore.getAngularModules();

    @NgModule({
      ...
      providers: [
        { provide: MICROFRONTENDS_TOKEN, useValue: mfModules },
        ...
      ],
      ...
    })
    export class AppModule {
      ...
    }
  ```

## Добавление удаленных компонентов в приложении с помощью `MfRemoteService`

1. **В шаблоне компонента добавить контейнер, в котором будет размещен удаленный компонент**

  ```javascript
  /** example.component.html */
    ...
      <div *ngIf="showRemoteComponent">
        <ng-container #placeholder></ng-container>
      </div
    ...
  ```

2. **В компоненте добавить `@ViewChild` для хранения рефа на контейнер**

  ```javascript
  /** example.component.ts */

    export class ExampleComponent {
      /** Контейнер для подключения BpmnModelerComponent из микрофронтенда */
      @ViewChild('placeholder', { read: ViewContainerRef }) public viewContainer!: ViewContainerRef;
      ...
    }
  ```

3. **Добавить в компонент сервис для работы с удаленными компонентами `MfRemoteService`.**

  ```javascript
  /** example.component.ts */
    import { MfRemoteService } from '@evolenta/processes-module';

    export class ExampleComponent {
      ...
      /** Сервис для добавления удаленных компонентов из мокрофронтендов */
      private _mfRemoteService = new MfRemoteService();
      ...
    }
  ```

4. **В методе, добавляющем удаленный компонент вызвать метод `addRemoteComponent` у сервиса и передать ему все необходимые данные.**

  Для того, чтобы быть уверенным, что контейнер будет точно доступен, лучше обернуть вызов метода
  в `setTimeout`. А передаваемые обработчики событий связывать с контекстом компонента.

  ```javascript
  /** example.component.ts */

    ...
    public async openRemoteComponent: Promise<void> {
      this.showRemoteComponent = true;

      setTimeout(() => {
        if (!this.viewContainer) {
          console.error('viewContainer not found');
          return;
        }

        try {
          const isRemoteAdded = await this._mfRemoteService.addRemoteComponent({
            componentKey: RemoteComponents.EXAMPLE_REMOTE_COMPONENT,
            viewContainer: this.viewContainer,
            inputs: {
                model: this.modelData,
                readonly: false,
                changeType: true,
            },
            outputsHandlers: {
                onClose: this.onCloseRemote.bind(this),
                onGetResult: this.onResultRemote.bind(this),
            },
          });
        } catch (error) {
          console.error('Не удалось подключить микрофронтенд', error);
        }
      })
    }
    ...


  ```

