---
title: hydrateRoot
---

<Intro>

`hydrateRoot` permite que você exiba componentes React dentro de um nó DOM do navegador cujo conteúdo HTML foi previamente gerado por [`react-dom/server`.](/reference/react-dom/server)

```js
const root = hydrateRoot(domNode, reactNode, options?)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `hydrateRoot(domNode, reactNode, options?)` {/*hydrateroot*/}

Chame `hydrateRoot` para "anexar" o React a um HTML existente que já foi renderizado pelo React em um ambiente de servidor.

```js
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

O React se anexará ao HTML que existe dentro do `domNode` e assumirá o gerenciamento do DOM dentro dele. Um aplicativo totalmente construído com React geralmente terá apenas uma chamada `hydrateRoot` com seu componente raiz.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `domNode`: Um [elemento DOM](https://developer.mozilla.org/en-US/docs/Web/API/Element) que foi renderizado como o elemento raiz no servidor.

* `reactNode`: O "nó React" usado para renderizar o HTML existente. Geralmente será um trecho de JSX como `<App />`, que foi renderizado com um método do `ReactDOM Server`, como `renderToPipeableStream(<App />)`.

* **opcional** `options`: Um objeto com opções para esta raiz React.

  * **opcional** `onCaughtError`: Callback chamado quando o React captura um erro em um Error Boundary. Chamado com o `error` capturado pelo Error Boundary e um objeto `errorInfo` contendo o `componentStack`.
  * **opcional** `onUncaughtError`: Callback chamado quando um erro é lançado e não é capturado por um Error Boundary. Chamado com o `error` que foi lançado e um objeto `errorInfo` contendo o `componentStack`.
  * **opcional** `onRecoverableError`: Callback chamado quando o React se recupera automaticamente de erros. Chamado com o `error` que o React lança e um objeto `errorInfo` contendo o `componentStack`. Alguns erros recuperáveis podem incluir a causa original do erro como `error.cause`.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar múltiplas raízes na mesma página. Deve ser o mesmo prefixo usado no servidor.


#### Retorna {/*returns*/}

`hydrateRoot` retorna um objeto com dois métodos: [`render`](#root-render) e [`unmount`.](#root-unmount)

#### Ressalvas {/*caveats*/}

* `hydrateRoot()` espera que o conteúdo renderizado seja idêntico ao conteúdo renderizado pelo servidor. Você deve tratar as incompatibilidades como erros e corrigi-las.
* No modo de desenvolvimento, o React emite avisos sobre incompatibilidades durante a hidratação. Não há garantias de que as diferenças de atributos serão corrigidas em caso de incompatibilidades. Isso é importante por razões de desempenho, pois na maioria dos aplicativos, as incompatibilidades são raras, e validar toda a marcação seria proibitivamente caro.
* Você provavelmente terá apenas uma chamada `hydrateRoot` em seu aplicativo. Se você usa um framework, ele pode fazer essa chamada para você.
* Se seu aplicativo é renderizado no cliente sem nenhum HTML pré-renderizado, usar `hydrateRoot()` não é suportado. Use [`createRoot()`](/reference/react-dom/client/createRoot) em vez disso.

---

### `root.render(reactNode)` {/*root-render*/}

Chame `root.render` para atualizar um componente React dentro de uma raiz React hidratada para um elemento DOM do navegador.

```js
root.render(<App />);
```

O React atualizará `<App />` na `root` hidratada.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*root-render-parameters*/}

* `reactNode`: Um "nó React" que você deseja atualizar. Geralmente será um trecho de JSX como `<App />`, mas você também pode passar um elemento React construído com [`createElement()`](/reference/react/createElement), uma string, um número, `null` ou `undefined`.


#### Retorna {/*root-render-returns*/}

`root.render` retorna `undefined`.

#### Ressalvas {/*root-render-caveats*/}

* Se você chamar `root.render` antes que a raiz termine de hidratar, o React limpará o conteúdo HTML existente renderizado pelo servidor e mudará toda a raiz para a renderização do lado do cliente.

---

### `root.unmount()` {/*root-unmount*/}

Chame `root.unmount` para destruir uma árvore renderizada dentro de uma raiz React.

```js
root.unmount();
```

Um aplicativo totalmente construído com React geralmente não terá nenhuma chamada para `root.unmount`.

Isso é útil principalmente se o nó DOM da raiz React (ou qualquer um de seus ancestrais) puder ser removido do DOM por algum outro código. Por exemplo, imagine um painel de abas do jQuery que remove abas inativas do DOM. Se uma aba for removida, tudo dentro dela (incluindo as raízes React dentro) também seria removido do DOM. Você precisa dizer ao React para "parar" de gerenciar o conteúdo da raiz removida chamando `root.unmount`. Caso contrário, os componentes dentro da raiz removida não farão a limpeza e liberarão recursos como assinaturas.

Chamar `root.unmount` desmontará todos os componentes na raiz e "desanexará" o React do nó DOM raiz, incluindo a remoção de quaisquer manipuladores de eventos ou estado na árvore.


#### Parâmetros {/*root-unmount-parameters*/}

`root.unmount` não aceita nenhum parâmetro.


#### Retorna {/*root-unmount-returns*/}

`root.unmount` retorna `undefined`.

#### Ressalvas {/*root-unmount-caveats*/}

* Chamar `root.unmount` desmontará todos os componentes na árvore e "desanexará" o React do nó DOM raiz.

* Depois de chamar `root.unmount`, você não poderá mais chamar `root.render` na raiz. Tentar chamar `root.render` em uma raiz desmontada lançará um erro "Cannot update an unmounted root".

---

## Uso {/*usage*/}

### Hidratando HTML renderizado pelo servidor {/*hydrating-server-rendered-html*/}

Se o HTML do seu aplicativo foi gerado por [`react-dom/server`](/reference/react-dom/client/createRoot), você precisa *hidratá-lo* no cliente.

```js [[1, 3, "document.getElementById('root')"], [2, 3, "<App />"]]
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
```

Isso hidratará o HTML do servidor dentro do <CodeStep step={1}>nó DOM do navegador</CodeStep> com o <CodeStep step={2}>componente React</CodeStep> do seu aplicativo. Geralmente, você fará isso uma vez na inicialização. Se você usa um framework, ele pode fazer isso nos bastidores para você.

Para hidratar seu aplicativo, o React "conectará" a lógica dos seus componentes ao HTML inicial gerado pelo servidor. A hidratação transforma o snapshot HTML inicial do servidor em um aplicativo totalmente interativo que é executado no navegador.

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado a partir de App por react-dom/server.
-->
<div id="root"><h1>Hello, world!</h1><button>You clicked me <!-- -->0<!-- --> times</button></div>
```

```js src/index.js active
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

```js src/App.js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <h1>Hello, world!</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      You clicked me {count} times
    </button>
  );
}
```

</Sandpack>

Você não deve precisar chamar `hydrateRoot` novamente ou chamá-lo em mais lugares. A partir deste ponto, o React gerenciará o DOM do seu aplicativo. Para atualizar a interface do usuário, seus componentes usarão [estado](/reference/react/useState) em vez disso.

<Pitfall>

A árvore React que você passa para `hydrateRoot` precisa produzir **o mesmo resultado** que produziu no servidor.

Isso é importante para a experiência do usuário. O usuário passará algum tempo olhando para o HTML gerado pelo servidor antes que seu código JavaScript seja carregado. A renderização do lado do servidor cria a ilusão de que o aplicativo carrega mais rápido, mostrando o snapshot HTML de sua saída. Mostrar repentinamente conteúdo diferente quebra essa ilusão. É por isso que a saída da renderização do servidor deve corresponder à saída da renderização inicial no cliente.

As causas mais comuns que levam a erros de hidratação incluem:

* Espaço em branco extra (como novas linhas) em torno do HTML gerado pelo React dentro do nó raiz.
* Uso de verificações como `typeof window !== 'undefined'` na sua lógica de renderização.
* Uso de APIs exclusivas do navegador como [`window.matchMedia`](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) na sua lógica de renderização.
* Renderização de dados diferentes no servidor e no cliente.

O React se recupera de alguns erros de hidratação, mas **você deve corrigi-los como outros bugs.** Na melhor das hipóteses, eles levarão a uma lentidão; na pior das hipóteses, os manipuladores de eventos podem ser anexados aos elementos errados.

</Pitfall>

---

### Hidratando um documento inteiro {/*hydrating-an-entire-document*/}

Aplicativos totalmente construídos com React podem renderizar o documento inteiro como JSX, incluindo a tag [`<html>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html):

```js {3,13}
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

Para hidratar o documento inteiro, passe o global [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) como o primeiro argumento para `hydrateRoot`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

---

### Suprimindo erros de incompatibilidade de hidratação inevitáveis {/*suppressing-unavoidable-hydration-mismatch-errors*/}

Se o atributo ou conteúdo de texto de um único elemento for inevitavelmente diferente entre o servidor e o cliente (por exemplo, um timestamp), você pode silenciar o aviso de incompatibilidade de hidratação.

Para silenciar avisos de hidratação em um elemento, adicione `suppressHydrationWarning={true}`:

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado a partir de App por react-dom/server.
-->
<div id="root"><h1>Current Date: <!-- -->01/01/2020</h1></div>
```

```js src/index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);
```

```js src/App.js active
export default function App() {
  return (
    <h1 suppressHydrationWarning={true}>
      Current Date: {new Date().toLocaleDateString()}
    </h1>
  );
}
```

</Sandpack>

Isso funciona apenas em um nível de profundidade e destina-se a ser uma saída de emergência. Não o use em excesso. O React **não** tentará corrigir o conteúdo de texto incompatível.

---

### Lidando com conteúdo diferente no cliente e no servidor {/*handling-different-client-and-server-content*/}

Se você precisar intencionalmente renderizar algo diferente no servidor e no cliente, pode fazer uma renderização em duas passagens. Componentes que renderizam algo diferente no cliente podem ler uma [variável de estado](/reference/react/useState) como `isClient`, que você pode definir como `true` em um [Effect](/reference/react/useEffect):

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado a partir de App por react-dom/server.
-->
<div id="root"><h1>Is Server</h1></div>
```

```js src/index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);
```

```js src/App.js active
import { useState, useEffect } from "react";

export default function App() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  return (
    <h1>
      {isClient ? 'Is Client' : 'Is Server'}
    </h1>
  );
}
```

</Sandpack>

Dessa forma, a renderização inicial renderizará o mesmo conteúdo do servidor, evitando incompatibilidades, mas uma passagem adicional ocorrerá de forma síncrona logo após a hidratação.

<Pitfall>

Essa abordagem torna a hidratação mais lenta porque seus componentes precisam renderizar duas vezes. Esteja ciente da experiência do usuário em conexões lentas. O código JavaScript pode carregar significativamente depois da renderização HTML inicial, então renderizar uma UI diferente imediatamente após a hidratação também pode parecer chocante para o usuário.

</Pitfall>

---

### Atualizando um componente raiz hidratado {/*updating-a-hydrated-root-component*/}

Após a conclusão da hidratação da raiz, você pode chamar [`root.render`](#root-render) para atualizar o componente raiz do React. **Ao contrário de [`createRoot`](/reference/react-dom/client/createRoot), você normalmente não precisa fazer isso porque o conteúdo inicial já foi renderizado como HTML.**

Se você chamar `root.render` em algum momento após a hidratação, e a estrutura da árvore de componentes corresponder ao que foi renderizado anteriormente, o React [preservará o estado.](/learn/preserving-and-resetting-state) Observe como você pode digitar no input, o que significa que as atualizações das chamadas repetidas de `render` a cada segundo neste exemplo não são destrutivas:

<Sandpack>

```html public/index.html
<!--
  Todo o conteúdo HTML dentro de <div id="root">...</div> foi
  gerado renderizando <App /> com react-dom/server.
-->
<div id="root"><h1>Hello, world! <!-- -->0</h1><input placeholder="Type something here"/></div>
```

```js src/index.js active
import { hydrateRoot } from 'react-dom/client';
import './styles.css';
import App from './App.js';

const root = hydrateRoot(
  document.getElementById('root'),
  <App counter={0} />
);

let i = 0;
setInterval(() => {
  root.render(<App counter={i} />);
  i++;
}, 1000);
```

```js src/App.js
export default function App({counter}) {
  return (
    <>
      <h1>Hello, world! {counter}</h1>
      <input placeholder="Type something here" />
    </>
  );
}
```

</Sandpack>

É incomum chamar [`root.render`](#root-render) em uma raiz hidratada. Normalmente, você [atualizará o estado](/reference/react/useState) dentro de um dos componentes.

### Registro de erros em produção {/*error-logging-in-production*/}

Por padrão, o React registrará todos os erros no console. Para implementar seu próprio relatório de erros, você pode fornecer as opções opcionais de raiz do manipulador de erros `onUncaughtError`, `onCaughtError` e `onRecoverableError`:

```js [[1, 7, "onCaughtError"], [2, 7, "error", 1], [3, 7, "errorInfo"], [4, 11, "componentStack", 15]]
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import { reportCaughtError } from "./reportError";

const container = document.getElementById("root");
const root = hydrateRoot(container, <App />, {
  onCaughtError: (error, errorInfo) => {
    if (error.message !== "Known error") {
      reportCaughtError({
        error,
        componentStack: errorInfo.componentStack,
      });
    }
  },
});
```

A opção <CodeStep step={1}>onCaughtError</CodeStep> é uma função chamada com dois argumentos:

1. O <CodeStep step={2}>erro</CodeStep> que foi lançado.
2. Um objeto <CodeStep step={3}>errorInfo</CodeStep> que contém o <CodeStep step={4}>componentStack</CodeStep> do erro.

Juntamente com `onUncaughtError` e `onRecoverableError`, você pode implementar seu próprio sistema de relatório de erros:

<Sandpack>

```js src/reportError.js
function reportError({ type, error, errorInfo }) {
  // A implementação específica depende de você.
  // `console.error()` é usado apenas para fins de demonstração.
  console.error(type, error, "Component Stack: ");
  console.error("Component Stack: ", errorInfo.componentStack);
}

export function onCaughtErrorProd(error, errorInfo) {
  if (error.message !== "Known error") {
    reportError({ type: "Caught", error, errorInfo });
  }
}

export function onUncaughtErrorProd(error, errorInfo) {
  reportError({ type: "Uncaught", error, errorInfo });
}

export function onRecoverableErrorProd(error, errorInfo) {
  reportError({ type: "Recoverable", error, errorInfo });
}
```

```js src/index.js active
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import {
  onCaughtErrorProd,
  onRecoverableErrorProd,
  onUncaughtErrorProd,
} from "./reportError";

const container = document.getElementById("root");
hydrateRoot(container, <App />, {
  // Lembre-se de remover essas opções em desenvolvimento para aproveitar
  // os manipuladores padrão do React ou implementar sua própria sobreposição para desenvolvimento.
  // Os manipuladores são especificados incondicionalmente aqui apenas para fins de demonstração.
  onCaughtError: onCaughtErrorProd,
  onRecoverableError: onRecoverableErrorProd,
  onUncaughtError: onUncaughtErrorProd,
});
```

```js src/App.js
import { Component, useState } from "react";

function Boom() {
  foo.bar = "baz";
}

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

export default function App() {
  const [triggerUncaughtError, settriggerUncaughtError] = useState(false);
  const [triggerCaughtError, setTriggerCaughtError] = useState(false);

  return (
    <>
      <button onClick={() => settriggerUncaughtError(true)}>
        Trigger uncaught error
      </button>
      {triggerUncaughtError && <Boom />}
      <button onClick={() => setTriggerCaughtError(true)}>
        Trigger caught error
      </button>
      {triggerCaughtError && (
        <ErrorBoundary>
          <Boom />
        </ErrorBoundary>
      )}
    </>
  );
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>My app</title>
</head>
<body>
<!--
  Propositalmente usando conteúdo HTML que difere do conteúdo renderizado pelo servidor para acionar erros recuperáveis.
-->
<div id="root">Server content before hydration.</div>
</body>
</html>
```
</Sandpack>

## Solução de problemas {/*troubleshooting*/}


### Estou recebendo um erro: "You passed a second argument to root.render" {/*im-getting-an-error-you-passed-a-second-argument-to-root-render*/}

Um erro comum é passar as opções para `hydrateRoot` para `root.render(...)`:

<ConsoleBlock level="error">

Warning: You passed a second argument to root.render(...) but it only accepts one argument.

</ConsoleBlock>

Para corrigir, passe as opções da raiz para `hydrateRoot(...)`, não para `root.render(...)`:
```js {2,5}
// 🚩 Errado: root.render aceita apenas um argumento.
root.render(App, {onUncaughtError});

// ✅ Correto: passe as opções para createRoot.
const root = hydrateRoot(container, <App />, {onUncaughtError});
```