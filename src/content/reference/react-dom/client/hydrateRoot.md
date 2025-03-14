html
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

Chame `hydrateRoot` para “anexar” o React ao HTML existente que já foi renderizado pelo React em um ambiente de servidor.

```js
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

O React irá anexar ao HTML que existe dentro do `domNode` e assumir o controle de gerenciamento do DOM dentro dele. Um aplicativo totalmente construído com o React geralmente terá apenas uma chamada `hydrateRoot` com seu componente raiz.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `domNode`: Um [elemento DOM](https://developer.mozilla.org/en-US/docs/Web/API/Element) que foi renderizado como o elemento raiz no servidor.

* `reactNode`: O “nó React” usado para renderizar o HTML existente. Isso geralmente será um trecho de JSX como `<App />` que foi renderizado com um método `ReactDOM Server` como `renderToPipeableStream(<App />)`.

* **opcional** `options`: Um objeto com opções para esta raiz React.

  * **opcional** `onCaughtError`: Callback chamado quando o React captura um erro em um Error Boundary. Chamado com o `error` capturado pelo Error Boundary e um objeto `errorInfo` contendo a `componentStack`.
  * **opcional** `onUncaughtError`: Callback chamado quando um erro é lançado e não é capturado por um Error Boundary. Chamado com o `error` que foi lançado e um objeto `errorInfo` contendo a `componentStack`.
  * **opcional** `onRecoverableError`: Callback chamado quando o React se recupera automaticamente de erros. Chamado com o `error` que o React lança e um objeto `errorInfo` contendo a `componentStack`. Alguns erros recuperáveis podem incluir a causa original do erro como `error.cause`.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar múltiplas raízes na mesma página. Deve ser o mesmo prefixo usado no servidor.

#### Retorna {/*returns*/}

`hydrateRoot` retorna um objeto com dois métodos: [`render`](#root-render) e [`unmount`.](#root-unmount)

#### Ressalvas {/*caveats*/}

* `hydrateRoot()` espera que o conteúdo renderizado seja idêntico ao conteúdo renderizado no servidor. Você deve tratar as incompatibilidades como erros e corrigi-los.
* No modo de desenvolvimento, o React adverte sobre incompatibilidades durante a hidratação. Não há garantias de que as diferenças de atributos serão corrigidas em caso de incompatibilidades. Isso é importante por razões de desempenho, pois na maioria dos aplicativos, as incompatibilidades são raras e, portanto, validar toda a marcação seria proibitivamente caro.
* Você provavelmente terá apenas uma chamada `hydrateRoot` em seu aplicativo. Se você usar um framework, ele poderá fazer essa chamada por você.
* Se seu aplicativo for renderizado no cliente sem HTML já renderizado, o uso de `hydrateRoot()` não é compatível. Use [`createRoot()`](/reference/react-dom/client/createRoot) em vez disso.

---

### `root.render(reactNode)` {/*root-render*/}

Chame `root.render` para atualizar um componente React dentro de uma raiz React hidratada para um elemento DOM do navegador.

```js
root.render(<App />);
```

O React atualizará `<App />` na `root` hidratada.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*root-render-parameters*/}

* `reactNode`: Um “nó React” que você deseja atualizar. Isso geralmente será um trecho de JSX como `<App />`, mas você também pode passar um elemento React construído com [`createElement()`](/reference/react/createElement), uma string, um número, `null` ou `undefined`.

#### Retorna {/*root-render-returns*/}

`root.render` retorna `undefined`.

#### Ressalvas {/*root-render-caveats*/}

* Se você chamar `root.render` antes que a raiz termine a hidratação, o React irá limpar o conteúdo HTML existente renderizado no servidor e mudar toda a raiz para renderização do cliente.

---

### `root.unmount()` {/*root-unmount*/}

Chame `root.unmount` para destruir uma árvore renderizada dentro de uma raiz React.

```js
root.unmount();
```

Um aplicativo totalmente construído com React geralmente não terá nenhuma chamada para `root.unmount`.

Isso é útil principalmente se o nó DOM da raiz React (ou qualquer um de seus ancestrais) puder ser removido do DOM por algum outro código. Por exemplo, imagine um painel de abas jQuery que remove abas inativas do DOM. Se uma aba for removida, tudo dentro dela (incluindo as raízes React dentro) também será removido do DOM. Você precisa dizer ao React para "parar" de gerenciar o conteúdo da raiz removida chamando `root.unmount`. Caso contrário, os componentes dentro da raiz removida não limparão e liberarão recursos como assinaturas.

Chamar `root.unmount` desmontará todos os componentes na raiz e "desanexará" o React do nó DOM da raiz, incluindo a remoção de qualquer manipulador de eventos ou estado na árvore.

#### Parâmetros {/*root-unmount-parameters*/}

`root.unmount` não aceita nenhum parâmetro.

#### Retorna {/*root-unmount-returns*/}

`root.unmount` retorna `undefined`.

#### Ressalvas {/*root-unmount-caveats*/}

* Chamar `root.unmount` desmontará todos os componentes na árvore e "desanexará" o React do nó DOM raiz.

* Depois de chamar `root.unmount`, você não pode chamar `root.render` novamente na raiz. Tentar chamar `root.render` em uma raiz desmontada lançará um erro "Não é possível atualizar uma raiz desmontada".

---

## Uso {/*usage*/}

### Hidratando HTML renderizado no servidor {/*hydrating-server-rendered-html*/}

Se o HTML do seu aplicativo foi gerado por [`react-dom/server`](/reference/react-dom/client/createRoot), você precisa *hidratá-lo* no cliente.

```js [[1, 3, "document.getElementById('root')"], [2, 3, "<App />"]]
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
```

Isso irá hidratar o HTML do servidor dentro do <CodeStep step={1}>nó DOM do navegador</CodeStep> com o <CodeStep step={2}>componente React</CodeStep> para seu aplicativo. Geralmente, você fará isso uma vez na inicialização. Se você usar um framework, ele poderá fazer isso nos bastidores para você.

Para hidratar seu aplicativo, o React irá “anexar” a lógica dos seus componentes ao HTML gerado inicialmente do servidor. A hidratação transforma o instantâneo HTML inicial do servidor em um aplicativo totalmente interativo que é executado no navegador.

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado pelo App por react-dom/server.
-->
<div id="root"><h1>Olá, mundo!</h1><button>Você clicou em mim <!-- -->0<!-- --> vezes</button></div>
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
      <h1>Olá, mundo!</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Você clicou em mim {count} vezes
    </button>
  );
}
```

</Sandpack>

Você não deve precisar chamar `hydrateRoot` novamente ou chamá-lo em mais lugares. A partir deste ponto, o React estará gerenciando o DOM da sua aplicação. Para atualizar a UI, seus componentes [usarão o state](/reference/react/useState) em vez disso.

<Pitfall>

A árvore React que você passa para `hydrateRoot` precisa produzir **a mesma saída** que produziu no servidor.

Isso é importante para a experiência do usuário. O usuário passará algum tempo olhando para o HTML gerado pelo servidor antes que seu código JavaScript seja carregado. A renderização no servidor cria a ilusão de que o aplicativo carrega mais rápido, mostrando o instantâneo HTML de sua saída. Mostrar de repente um conteúdo diferente quebra essa ilusão. É por isso que a saída da renderização do servidor deve corresponder à saída da renderização inicial no cliente.

As causas mais comuns que levam a erros de hidratação incluem:

* Espaços em branco extras (como quebras de linha) em torno do HTML gerado pelo React dentro do nó raiz.
* Usar verificações como `typeof window !== 'undefined'` na sua lógica de renderização.
* Usar APIs somente do navegador como [`window.matchMedia`](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) na sua lógica de renderização.
* Renderizar dados diferentes no servidor e no cliente.

O React se recupera de alguns erros de hidratação, mas **você deve corrigi-los como outros erros.** No melhor dos casos, eles levarão a uma lentidão; no pior dos casos, os manipuladores de eventos podem ser anexados aos elementos errados.

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
        <title>Meu aplicativo</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

Para hidratar todo o documento, passe o global [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) como o primeiro argumento para `hydrateRoot`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

---

### Suprimindo erros de incompatibilidade de hidratação inevitáveis {/*suppressing-unavoidable-hydration-mismatch-errors*/}

Se o atributo ou conteúdo de texto de um único elemento for inevitavelmente diferente entre o servidor e o cliente (por exemplo, um timestamp), você poderá silenciar o aviso de incompatibilidade de hidratação.

Para silenciar os avisos de hidratação em um elemento, adicione `suppressHydrationWarning={true}`:

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado pelo App por react-dom/server.
-->
<div id="root"><h1>Data atual: <!-- -->01/01/2020</h1></div>
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
      Data atual: {new Date().toLocaleDateString()}
    </h1>
  );
}
```

</Sandpack>

Isso só funciona em um nível e foi projetado para ser uma forma de escapar. Não use em excesso. A menos que seja conteúdo de texto, o React ainda não tentará corrigi-lo, portanto, pode permanecer inconsistente até futuras atualizações.

---

### Lidando com conteúdo diferente do cliente e do servidor {/*handling-different-client-and-server-content*/}

Se você intencionalmente precisar renderizar algo diferente no servidor e no cliente, poderá fazer uma renderização em duas passagens. Os componentes que renderizam algo diferente no cliente podem ler um [variável de estado](/reference/react/useState) como `isClient`, que você pode definir como `true` em um [Effect](/reference/react/useEffect):

<Sandpack>

```html public/index.html
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado pelo App por react-dom/server.
-->
<div id="root"><h1>É Servidor</h1></div>
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
      {isClient ? 'É Cliente' : 'É Servidor'}
    </h1>
  );
}
```

</Sandpack>

Dessa forma, a passagem de renderização inicial renderizará o mesmo conteúdo que o servidor, evitando incompatibilidades, mas uma passagem adicional acontecerá de forma síncrona logo após a hidratação.

<Pitfall>

Essa abordagem torna a hidratação mais lenta porque seus componentes precisam renderizar duas vezes. Esteja atento à experiência do usuário em conexões lentas. O código JavaScript pode carregar significativamente mais tarde do que a renderização HTML inicial, portanto, renderizar uma UI diferente imediatamente após a hidratação também pode parecer chocante para o usuário.

</Pitfall>

---

### Atualizando um componente raiz hidratado {/*updating-a-hydrated-root-component*/}

Depois que a raiz terminar a hidratação, você pode chamar [`root.render`](#root-render) para atualizar o componente React raiz. **Ao contrário do [`createRoot`](/reference/react-dom/client/createRoot), você geralmente não precisa fazer isso porque o conteúdo inicial já foi renderizado como HTML.**

Se você chamar `root.render` em algum momento após a hidratação, e a estrutura da árvore de componentes corresponder ao que foi renderizado anteriormente, o React irá [preservar o state.](/learn/preserving-and-resetting-state) Observe como você pode digitar na entrada, o que significa que as atualizações de chamadas `render` repetidas a cada segundo neste exemplo não são destrutivas:

<Sandpack>

```html public/index.html
<!--
  Todo o conteúdo HTML dentro de <div id="root">...</div> foi
  gerado pela renderização <App /> com react-dom/server.
-->
<div id="root"><h1>Olá, mundo! <!-- -->0</h1><input placeholder="Digite algo aqui" /></div>
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
      <h1>Olá, mundo! {counter}</h1>
      <input placeholder="Digite algo aqui" />
    </>
  );
}
```

</Sandpack>

É incomum chamar [`root.render`](#root-render) em uma raiz hidratada. Normalmente, você [irá atualizar o state](/reference/react/useState) dentro de um dos componentes em vez disso.

### Mostrar uma caixa de diálogo para erros não detectados {/*show-a-dialog-for-uncaught-errors*/}

Por padrão, o React registrará todos os erros não detectados no console. Para implementar seu próprio relatório de erros, você pode fornecer a opção de raiz opcional `onUncaughtError`:

```js [[1, 7, "onUncaughtError"], [2, 7, "error", 1], [3, 7, "errorInfo"], [4, 11, "componentStack"]]
import { hydrateRoot } from 'react-dom/client';

const root = hydrateRoot(
  document.getElementById('root'),
  <App />,
  {
    onUncaughtError: (error, errorInfo) => {
      console.error(
        'Erro não detectado',
        error,
        errorInfo.componentStack
      );
    }
  }
);
root.render(<App />);
```

A opção <CodeStep step={1}>onUncaughtError</CodeStep> é uma função chamada com dois argumentos:

1. O <CodeStep step={2}>error</CodeStep> que foi lançado.
2. Um objeto <CodeStep step={3}>errorInfo</CodeStep> que contém o <CodeStep step={4}>componentStack</CodeStep> do erro.

Você pode usar a opção de raiz `onUncaughtError` para exibir caixas de diálogo de erro:

<Sandpack>

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>Meu aplicativo</title>
</head>
<body>
<!--
  Caixa de diálogo de erro em HTML bruto
  já que um erro no aplicativo React pode travar.
-->
<div id="error-dialog" class="hidden">
  <h1 id="error-title" class="text-red"></h1>
  <h3>
    <pre id="error-message"></pre>
  </h3>
  <p>
    <pre id="error-body"></pre>
  </p>
  <h4 class="-mb-20">Este erro ocorreu em:</h4>
  <pre id="error-component-stack" class="nowrap"></pre>
  <h4 class="mb-0">Rastreamento de chamadas:</h4>
  <pre id="error-stack" class="nowrap"></pre>
  <div id="error-cause">
    <h4 class="mb-0">Causado por:</h4>
    <pre id="error-cause-message"></pre>
    <pre id="error-cause-stack" class="nowrap"></pre>
  </div>
  <button
    id="error-close"
    class="mb-10"
    onclick="document.getElementById('error-dialog').classList.add('hidden')"
  >
    Fechar
  </button>
  <h3 id="error-not-dismissible">Este erro não pode ser dispensado.</h3>
</div>
<!--
  O conteúdo HTML dentro de <div id="root">...</div>
  foi gerado pelo App por react-dom/server.
-->
<div id="root"><div><span>Este erro mostra a caixa de diálogo de erro:</span><button>Lançar erro</button></div></div>
</body>
</html>
```

```css src/styles.css active
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }

#error-dialog {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: white;
  padding: 15px;
  opacity: 0.9;
  text-wrap: wrap;
  overflow: scroll;
}

.text-red {
  color: red;
}

.-mb-20 {
  margin-bottom: -20px;
}

.mb-0 {
  margin-bottom: 0;
}

.mb-10 {
  margin-bottom: 10px;
}

pre {
  text-wrap: wrap;
}

pre.nowrap {
  text-wrap: nowrap;
}

.hidden {
 display: none;  
}
```

```js src/reportError.js hidden
function reportError({ title, error, componentStack, dismissable }) {
  const errorDialog = document.getElementById("error-dialog");
  const errorTitle = document.getElementById("error-title");
  const errorMessage = document.getElementById("error-message");
  const errorBody = document.getElementById("error-body");
  const errorComponentStack = document.getElementById("error-component-stack");
  const errorStack = document.getElementById("error-stack");
  const errorClose = document.getElementById("error-close");
  const errorCause = document.getElementById("error-cause");
  const errorCauseMessage = document.getElementById("error-cause-message");
  const errorCauseStack = document.getElementById("error-cause-stack");
  const errorNotDismissible = document.getElementById("error-not-dismissible");
  
  // Definir o título
  errorTitle.innerText = title;
  
  // Exibir mensagem de erro e corpo
  const [heading, body] = error.message.split(/\n(.*)/s);
  errorMessage.innerText = heading;
  if (body) {
    errorBody.innerText = body;
  } else {
    errorBody.innerText = '';
  }
``````js
  // Display error message and body
  const [heading, body] = error.message.split(/\n(.*)/s);
  errorMessage.innerText = heading;
  if (body) {
    errorBody.innerText = body;
  } else {
    errorBody.innerText = '';
  }
```

```js src/index.js active
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import {reportUncaughtError} from "./reportError";
import "./styles.css";
import {renderToString} from 'react-dom/server';

const container = document.getElementById("root");
const root = hydrateRoot(container, <App />, {
  onUncaughtError: (error, errorInfo) => {
    if (error.message !== 'Known error') {
      reportUncaughtError({
        error,
        componentStack: errorInfo.componentStack
      });
    }
  }
});
```

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [throwError, setThrowError] = useState(false);
  
  if (throwError) {
    foo.bar = 'baz';
  }
  
  return (
    <div>
      <span>This error shows the error dialog:</span>
      <button onClick={() => setThrowError(true)}>
        Throw error
      </button>
    </div>
  );
}
```

</Sandpack>

### Exibindo erros de Error Boundary {/*displaying-error-boundary-errors*/}

Por padrão, o React registrará todos os erros capturados pelo Error Boundary no `console.error`. Para substituir esse comportamento, você pode fornecer a opção opcional `onCaughtError` da raiz para erros capturados por um [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary):

```js [[1, 7, "onCaughtError"], [2, 7, "error", 1], [3, 7, "errorInfo"], [4, 11, "componentStack"]]
import { hydrateRoot } from 'react-dom/client';

const root = hydrateRoot(
  document.getElementById('root'),
  <App />,
  {
    onCaughtError: (error, errorInfo) => {
      console.error(
        'Caught error',
        error,
        errorInfo.componentStack
      );
    }
  }
);
root.render(<App />);
```

A opção <CodeStep step={1}>onCaughtError</CodeStep> é uma função chamada com dois argumentos:

1.  O <CodeStep step={2}>erro</CodeStep> que foi capturado pela boundary.
2.  Um objeto <CodeStep step={3}>errorInfo</CodeStep> que contém o <CodeStep step={4}>componentStack</CodeStep> do erro.

Você pode usar a opção de raiz `onCaughtError` para exibir diálogos de erro ou filtrar erros conhecidos do log:

<Sandpack>

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>My app</title>
</head>
<body>
<!--
  Error dialog in raw HTML
  since an error in the React app may crash.
-->
<div id="error-dialog" class="hidden">
  <h1 id="error-title" class="text-red"></h1>
  <h3>
    <pre id="error-message"></pre>
  </h3>
  <p>
    <pre id="error-body"></pre>
  </p>
  <h4 class="-mb-20">This error occurred at:</h4>
  <pre id="error-component-stack" class="nowrap"></pre>
  <h4 class="mb-0">Call stack:</h4>
  <pre id="error-stack" class="nowrap"></pre>
  <div id="error-cause">
    <h4 class="mb-0">Caused by:</h4>
    <pre id="error-cause-message"></pre>
    <pre id="error-cause-stack" class="nowrap"></pre>
  </div>
  <button
    id="error-close"
    class="mb-10"
    onclick="document.getElementById('error-dialog').classList.add('hidden')"
  >
    Close
  </button>
  <h3 id="error-not-dismissible">This error is not dismissible.</h3>
</div>
<!--
  HTML content inside <div id="root">...</div>
  was generated from App by react-dom/server.
-->
<div id="root"><span>This error will not show the error dialog:</span><button>Throw known error</button><span>This error will show the error dialog:</span><button>Throw unknown error</button></div>
</body>
</html>
```

```css src/styles.css active
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }

#error-dialog {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: white;
  padding: 15px;
  opacity: 0.9;
  text-wrap: wrap;
  overflow: scroll;
}

.text-red {
  color: red;
}

.-mb-20 {
  margin-bottom: -20px;
}

.mb-0 {
  margin-bottom: 0;
}

.mb-10 {
  margin-bottom: 10px;
}

pre {
  text-wrap: wrap;
}

pre.nowrap {
  text-wrap: nowrap;
}

.hidden {
 display: none;  
}
```

```js src/reportError.js hidden
function reportError({ title, error, componentStack, dismissable }) {
  const errorDialog = document.getElementById("error-dialog");
  const errorTitle = document.getElementById("error-title");
  const errorMessage = document.getElementById("error-message");
  const errorBody = document.getElementById("error-body");
  const errorComponentStack = document.getElementById("error-component-stack");
  const errorStack = document.getElementById("error-stack");
  const errorClose = document.getElementById("error-close");
  const errorCause = document.getElementById("error-cause");
  const errorCauseMessage = document.getElementById("error-cause-message");
  const errorCauseStack = document.getElementById("error-cause-stack");
  const errorNotDismissible = document.getElementById("error-not-dismissible");
  
  // Set the title
  errorTitle.innerText = title;
  
  // Display error message and body
  const [heading, body] = error.message.split(/\n(.*)/s);
  errorMessage.innerText = heading;
  if (body) {
    errorBody.innerText = body;
  } else {
    errorBody.innerText = '';
  }

  // Display component stack
  errorComponentStack.innerText = componentStack;

  // Display the call stack
  // Since we already displayed the message, strip it, and the first Error: line.
  errorStack.innerText = error.stack.replace(error.message, '').split(/\n(.*)/s)[1];
  
  // Display the cause, if available
  if (error.cause) {
    errorCauseMessage.innerText = error.cause.message;
    errorCauseStack.innerText = error.cause.stack;
    errorCause.classList.remove('hidden');
  } else {
    errorCause.classList.add('hidden');
  }
  // Display the close button, if dismissible
  if (dismissable) {
    errorNotDismissible.classList.add('hidden');
    errorClose.classList.remove("hidden");
  } else {
    errorNotDismissible.classList.remove('hidden');
    errorClose.classList.add("hidden");
  }
  
  // Show the dialog
  errorDialog.classList.remove("hidden");
}

export function reportCaughtError({error, cause, componentStack}) {
  reportError({ title: "Caught Error", error, componentStack,  dismissable: true});
}

export function reportUncaughtError({error, cause, componentStack}) {
  reportError({ title: "Uncaught Error", error, componentStack, dismissable: false });
}

export function reportRecoverableError({error, cause, componentStack}) {
  reportError({ title: "Recoverable Error", error, componentStack,  dismissable: true });
}
```

```js src/index.js active
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import {reportCaughtError} from "./reportError";
import "./styles.css";

const container = document.getElementById("root");
const root = hydrateRoot(container, <App />, {
  onCaughtError: (error, errorInfo) => {
    if (error.message !== 'Known error') {
      reportCaughtError({
        error,
        componentStack: errorInfo.componentStack
      });
    }
  }
});
```

```js src/App.js
import { useState } from 'react';
import { ErrorBoundary } from "react-error-boundary";

export default function App() {
  const [error, setError] = useState(null);
  
  function handleUnknown() {
    setError("unknown");
  }

  function handleKnown() {
    setError("known");
  }
  
  return (
    <>
      <ErrorBoundary
        fallbackRender={fallbackRender}
        onReset={(details) => {
          setError(null);
        }}
      >
        {error != null && <Throw error={error} />}
        <span>This error will not show the error dialog:</span>
        <button onClick={handleKnown}>
          Throw known error
        </button>
        <span>This error will show the error dialog:</span>
        <button onClick={handleUnknown}>
          Throw unknown error
        </button>
      </ErrorBoundary>
      
    </>
  );
}

function fallbackRender({ resetErrorBoundary }) {
  return (
    <div role="alert">
      <h3>Error Boundary</h3>
      <p>Something went wrong.</p>
      <button onClick={resetErrorBoundary}>Reset</button>
    </div>
  );
}

function Throw({error}) {
  if (error === "known") {
    throw new Error('Known error')
  } else {
    foo.bar = 'baz';
  }
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0-rc-3edc000d-20240926",
    "react-dom": "19.0.0-rc-3edc000d-20240926",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```

</Sandpack>

### Mostrar um diálogo para erros de incompatibilidade de hidratação recuperáveis {/*show-a-dialog-for-recoverable-hydration-mismatch-errors*/}

Quando o React encontra uma incompatibilidade de hidratação, ele tentará automaticamente se recuperar, renderizando no cliente. Por padrão, o React registrará os erros de incompatibilidade de hidratação no `console.error`. Para substituir esse comportamento, você pode fornecer a opção opcional `onRecoverableError` da raiz:

```js [[1, 7, "onRecoverableError"], [2, 7, "error", 1], [3, 11, "error.cause", 1], [4, 7, "errorInfo"], [5, 12, "componentStack"]]
import { hydrateRoot } from 'react-dom/client';

const root = hydrateRoot(
  document.getElementById('root'),
  <App />,
  {
    onRecoverableError: (error, errorInfo) => {
      console.error(
        'Caught error',
        error,
        error.cause,
        errorInfo.componentStack
      );
    }
  }
);
```

A opção <CodeStep step={1}>onRecoverableError</CodeStep> é uma função chamada com dois argumentos:

1.  O <CodeStep step={2}>erro</CodeStep> que o React lança. Alguns erros podem incluir a causa original como <CodeStep step={3}>error.cause</CodeStep>.
2.  Um objeto <CodeStep step={4}>errorInfo</CodeStep> que contém o <CodeStep step={5}>componentStack</CodeStep> do erro.

Você pode usar a opção da raiz `onRecoverableError` para exibir diálogos de erro para incompatibilidades de hidratação:

<Sandpack>

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>My app</title>
</head>
<body>
<!--
  Error dialog in raw HTML
  since an error in the React app may crash.
-->
<div id="error-dialog" class="hidden">
  <h1 id="error-title" class="text-red"></h1>
  <h3>
    <pre id="error-message"></pre>
  </h3>
  <p>
    <pre id="error-body"></pre>
  </p>
  <h4 class="-mb-20">This error occurred at:</h4>
  <pre id="error-component-stack" class="nowrap"></pre>
  <h4 class="mb-0">Call stack:</h4>
  <pre id="error-stack" class="nowrap"></pre>
  <div id="error-cause">
    <h4 class="mb-0">Caused by:</h4>
    <pre id="error-cause-message"></pre>
    <pre id="error-cause-stack" class="nowrap"></pre>
  </div>
  <button
    id="error-close"
    class="mb-10"
    onclick="document.getElementById('error-dialog').classList.add('hidden')"
  >
    Close
  </button>
  <h3 id="error-not-dismissible">This error is not dismissible.</h3>
</div>
<!--
  HTML content inside <div id="root">...</div>
  was generated from App by react-dom/server.
-->
<div id="root"><span>Server</span></div>
</body>
</html>
```

```css src/styles.css active
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }

#error-dialog {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: white;
  padding: 15px;
  opacity: 0.9;
  text-wrap: wrap;
  overflow: scroll;
}

.text-red {
  color: red;
}

.-mb-20 {
  margin-bottom: -20px;
}

.mb-0 {
  margin-bottom: 0;
}

.mb-10 {
  margin-bottom: 10px;
}

pre {
  text-wrap: wrap;
}

pre.nowrap {
  text-wrap: nowrap;
}

.hidden {
 display: none;  
}
```

```js src/reportError.js hidden
function reportError({ title, error, componentStack, dismissable }) {
  const errorDialog = document.getElementById("error-dialog");
  const errorTitle = document.getElementById("error-title");
  const errorMessage = document.getElementById("error-message");
  const errorBody = document.getElementById("error-body");
  const errorComponentStack = document.getElementById("error-component-stack");
  const errorStack = document.getElementById("error-stack");
  const errorClose = document.getElementById("error-close");
  const errorCause = document.getElementById("error-cause");
  const errorCauseMessage = document.getElementById("error-cause-message");
  const errorCauseStack = document.getElementById("error-cause-stack");
  const errorNotDismissible = document.getElementById("error-not-dismissible");
  
  // Set the title
  errorTitle.innerText = title;
  
  // Display error message and body
  const [heading, body] = error.message.split(/\n(.*)/s);
  errorMessage.innerText = heading;
  if (body) {
    errorBody.innerText = body;
  } else {
    errorBody.innerText = '';
  }

  // Display component stack
  errorComponentStack.innerText = componentStack;

  // Display the call stack
  // Since we already displayed the message, strip it, and the first Error: line.
  errorStack.innerText = error.stack.replace(error.message, '').split(/\n(.*)/s)[1];
  
  // Display the cause, if available
  if (error.cause) {
    errorCauseMessage.innerText = error.cause.message;
    errorCauseStack.innerText = error.cause.stack;
    errorCause.classList.remove('hidden');
  } else {
    errorCause.classList.add('hidden');
  }
  // Display the close button, if dismissible
  if (dismissable) {
    errorNotDismissible.classList.add('hidden');
    errorClose.classList.remove("hidden");
  } else {
    errorNotDismissible.classList.remove('hidden');
    errorClose.classList.add("hidden");
  }
  
  // Show the dialog
  errorDialog.classList.remove("hidden");
}

export function reportCaughtError({error, cause, componentStack}) {
  reportError({ title: "Caught Error", error, componentStack,  dismissable: true});
}

export function reportUncaughtError({error, cause, componentStack}) {
  reportError({ title: "Uncaught Error", error, componentStack, dismissable: false });
}

export function reportRecoverableError({error, cause, componentStack}) {
  reportError({ title: "Recoverable Error", error, componentStack,  dismissable: true });
}
```

```js src/index.js active
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import {reportRecoverableError} from "./reportError";
import "./styles.css";

const container = document.getElementById("root");
const root = hydrateRoot(container, <App />, {
  onRecoverableError: (error, errorInfo) => {
    reportRecoverableError({
      error,
      cause: error.cause,
      componentStack: errorInfo.componentStack
    });
  }
});
```

```js src/App.js
import { useState } from 'react';
import { ErrorBoundary } from "react-error-boundary";

export default function App() {
  const [error, setError] = useState(null);
  
  function handleUnknown() {
    setError("unknown");
  }

  function handleKnown() {
    setError("known");
  }
  
  return (
    <span>{typeof window !== 'undefined' ? 'Client' : 'Server'}</span>
  );
}

function fallbackRender({ resetErrorBoundary }) {
  return (
    <div role="alert">
      <h3>Error Boundary</h3>
      <p>Something went wrong.</p>
      <button onClick={resetErrorBoundary}>Reset</button>
    </div>
  );
}

function Throw({error}) {
  if (error === "known") {
    throw new Error('Known error')
  } else {
    foo.bar = 'baz';
  }
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0-rc-3edc000d-20240926",
    "react-dom": "19.0.0-rc-3edc000d-20240926",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```

</Sandpack>

## Solução de problemas {/*troubleshooting*/}

### Estou recebendo um erro: "Você passou um segundo argumento para root.render" {/*im-getting-an-error-you-passed-a-second-argument-to-root-render*/}

Um erro comum é passar as opções para `hydrateRoot` para `root.render(...)`:

<ConsoleBlock level="error">

Aviso: Você passou um segundo argumento para root.render(...) mas ele aceita apenas um argumento.

</ConsoleBlock>

Para corrigir, passe as opções raiz para `hydrateRoot(...)`, não `root.render(...)`:

```js {2,5}
// 🚩 Errado: root.render recebe apenas um argumento.
root.render(App, {onUncaughtError});

// ✅ Correto: passe opções para createRoot.
const root = hydrateRoot(container, <App />, {onUncaughtError});
```
