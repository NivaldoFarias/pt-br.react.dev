---
title: useSyncExternalStore
---

<Intro>

`useSyncExternalStore` é um Hook do React que permite que você se inscreva em um armazenamento externo.

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` {/*usesyncexternalstore*/}

Chame `useSyncExternalStore` no nível superior do seu componente para ler um valor de um armazenamento de dados externo.

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

Ele retorna o snapshot dos dados no armazenamento. Você precisa passar duas funções como argumentos:

1.  A função `subscribe` deve se inscrever no armazenamento e retornar uma função que cancele a inscrição.
2.  A função `getSnapshot` deve ler um snapshot dos dados do armazenamento.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

*   `subscribe`: Uma função que recebe um único argumento `callback` e o inscreve no armazenamento. Quando o armazenamento muda, ele deve invocar o `callback` fornecido, o que fará com que o React chame novamente `getSnapshot` e (se necessário) renderize novamente o componente. A função `subscribe` deve retornar uma função que limpa a inscrição.

*   `getSnapshot`: Uma função que retorna um snapshot dos dados no armazenamento que são necessários pelo componente. Enquanto o armazenamento não mudar, as chamadas repetidas para `getSnapshot` devem retornar o mesmo valor. Se o armazenamento mudar e o valor retornado for diferente (conforme comparado por [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), o React renderiza novamente o componente.

*   **opcional** `getServerSnapshot`: Uma função que retorna o snapshot inicial dos dados no armazenamento. Ele será usado apenas durante a renderização no servidor e durante a hidratação do conteúdo renderizado no servidor no cliente. O snapshot do servidor deve ser o mesmo entre o cliente e o servidor, e geralmente é serializado e passado do servidor para o cliente. Se você omitir este argumento, a renderização do componente no servidor lançará um erro.

#### Retorna {/*returns*/}

O snapshot atual do armazenamento que você pode usar em sua lógica de renderização.

#### Ressalvas {/*caveats*/}

*   O snapshot do armazenamento retornado por `getSnapshot` deve ser imutável. Se o armazenamento subjacente tiver dados mutáveis, retorne um novo snapshot imutável se os dados tiverem sido alterados. Caso contrário, retorne um snapshot armazenado em cache.

*   Se uma função `subscribe` diferente for passada durante uma nova renderização, o React se inscreverá novamente no armazenamento usando a função `subscribe` recém-passada. Você pode evitar isso declarando `subscribe` fora do componente.

*   Se o armazenamento for mutado durante uma atualização de [Transição](/reference/react/useTransition) [não bloqueante], o React recorrerá à execução dessa atualização como bloqueante. Especificamente, para cada atualização de Transição, o React chamará `getSnapshot` uma segunda vez logo antes de aplicar as alterações ao DOM. Se ele retornar um valor diferente do que quando foi chamado originalmente, o React reiniciará a atualização do zero, desta vez aplicando-a como uma atualização bloqueante, para garantir que cada componente na tela esteja refletindo a mesma versão do armazenamento.

*   Não é recomendado _suspender_ uma renderização com base em um valor de armazenamento retornado por `useSyncExternalStore`. A razão é que as mutações no armazenamento externo não podem ser marcadas como [atualizações de Transição](/reference/react/useTransition) [não bloqueantes], portanto, elas acionarão o [`Suspense` fallback](/reference/react/Suspense) mais próximo, substituindo o conteúdo já renderizado na tela por um indicador de carregamento, o que normalmente resulta em uma UX ruim.

    Por exemplo, o seguinte é desencorajado:

    ```js
    const LazyProductDetailPage = lazy(() => import('./ProductDetailPage.js'));

    function ShoppingApp() {
      const selectedProductId = useSyncExternalStore(...);

      // ❌ Chamando `use` com uma Promise dependente de `selectedProductId`
      const data = use(fetchItem(selectedProductId))

      // ❌ Renderizando condicionalmente um componente lazy com base em `selectedProductId`
      return selectedProductId != null ? <LazyProductDetailPage /> : <FeaturedProducts />;
    }
    ```

---

## Uso {/*usage*/}

### Inscrevendo-se em um armazenamento externo {/*subscribing-to-an-external-store*/}

A maioria dos seus componentes React apenas lerá dados de suas [props,](/learn/passing-props-to-a-component) [state,](/reference/react/useState) e [context.](/reference/react/useContext) No entanto, às vezes um componente precisa ler alguns dados de algum armazenamento fora do React que muda com o tempo. Isso inclui:

*   Bibliotecas de gerenciamento de estado de terceiros que mantêm o estado fora do React.
*   APIs do navegador que expõem um valor mutável e eventos para se inscrever em suas alterações.

Chame `useSyncExternalStore` no nível superior do seu componente para ler um valor de um armazenamento de dados externo.

```js [[1, 5, "todosStore.subscribe"], [2, 5, "todosStore.getSnapshot"], [3, 5, "todos", 0]]
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

Ele retorna o <CodeStep step={3}>snapshot</CodeStep> dos dados no armazenamento. Você precisa passar duas funções como argumentos:

1.  A <CodeStep step={1}>função `subscribe`</CodeStep> deve se inscrever no armazenamento e retornar uma função que cancele a inscrição.
2.  A <CodeStep step={2}>função `getSnapshot`</CodeStep> deve ler um snapshot dos dados do armazenamento.

O React usará essas funções para manter seu componente inscrito no armazenamento e renderizá-lo novamente em caso de alterações.

Por exemplo, no sandbox abaixo, `todosStore` é implementado como um armazenamento externo que armazena dados fora do React. O componente `TodosApp` se conecta a esse armazenamento externo com o Hook `useSyncExternalStore`.

<Sandpack>

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todoStore.js
// Este é um exemplo de um armazenamento de terceiros
// que você pode precisar integrar com o React.

// Se seu aplicativo for totalmente construído com React,
// recomendamos usar o estado do React em vez disso.

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

</Sandpack>

<Note>

Quando possível, recomendamos usar o estado integrado do React com [`useState`](/reference/react/useState) e [`useReducer`](/reference/react/useReducer) em vez disso. A API `useSyncExternalStore` é principalmente útil se você precisar integrar com código não React existente.

</Note>

---

### Inscrevendo-se em uma API do navegador {/*subscribing-to-a-browser-api*/}

Outra razão para adicionar `useSyncExternalStore` é quando você deseja se inscrever em algum valor exposto pelo navegador que muda com o tempo. Por exemplo, suponha que você queira que seu componente exiba se a conexão de rede está ativa. O navegador expõe essa informação por meio de uma propriedade chamada [`navigator.onLine`.](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine)

Esse valor pode mudar sem o conhecimento do React, então você deve lê-lo com `useSyncExternalStore`.

```js
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

Para implementar a função `getSnapshot`, leia o valor atual da API do navegador:

```js
function getSnapshot() {
  return navigator.onLine;
}
```

Em seguida, você precisa implementar a função `subscribe`. Por exemplo, quando `navigator.onLine` muda, o navegador aciona os eventos [`online`](https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event) e [`offline`](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event) no objeto `window`. Você precisa inscrever o argumento `callback` nos eventos correspondentes e, em seguida, retornar uma função que limpe as inscrições:

```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

Agora, o React sabe como ler o valor da API externa `navigator.onLine` e como se inscrever em suas alterações. Desconecte seu dispositivo da rede e observe que o componente é renderizado novamente em resposta:

<Sandpack>

```js
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

</Sandpack>

---

### Extraindo a lógica para um Hook personalizado {/*extracting-the-logic-to-a-custom-hook*/}

Normalmente, você não escreverá `useSyncExternalStore` diretamente em seus componentes. Em vez disso, você normalmente o chamará do seu próprio Hook personalizado. Isso permite que você use o mesmo armazenamento externo de diferentes componentes.

Por exemplo, este Hook personalizado `useOnlineStatus` rastreia se a rede está online:

```js {3,6}
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  // ...
}

function subscribe(callback) {
  // ...
}
```

Agora, diferentes componentes podem chamar `useOnlineStatus` sem repetir a implementação subjacente:

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

```js src/useOnlineStatus.js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

</Sandpack>

---

### Adicionando suporte para renderização no servidor {/*adding-support-for-server-rendering*/}

Se seu aplicativo React usar [renderização no servidor,](/reference/react-dom/server) seus componentes React também serão executados fora do ambiente do navegador para gerar o HTML inicial. Isso cria alguns desafios ao conectar a um armazenamento externo:

*   Se você estiver se conectando a uma API apenas para o navegador, ela não funcionará porque não existe no servidor.
*   Se você estiver se conectando a um armazenamento de dados de terceiros, precisará que seus dados correspondam entre o servidor e o cliente.

Para resolver esses problemas, passe uma função `getServerSnapshot` como o terceiro argumento para `useSyncExternalStore`:

```js {4,12-14}
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // Sempre mostrar "Online" para HTML gerado pelo servidor
}

function subscribe(callback) {
  // ...
}
```

A função `getServerSnapshot` é semelhante a `getSnapshot`, mas é executada apenas em duas situações:

*   Ele é executado no servidor ao gerar o HTML.
*   Ele é executado no cliente durante a [hidratação](/reference/react-dom/client/hydrateRoot), ou seja, quando o React pega o HTML do servidor e o torna interativo.

Isso permite que você forneça o valor do snapshot inicial que será usado antes que o aplicativo se torne interativo. Se não houver um valor inicial significativo para a renderização no servidor, omita este argumento para [forçar a renderização no cliente.](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

<Note>

Certifique-se de que `getServerSnapshot` retorne exatamente os mesmos dados na renderização inicial do cliente que ele retornou no servidor. Por exemplo, se `getServerSnapshot` retornasse algum conteúdo de armazenamento pré-preenchido no servidor, você precisaria transferir esse conteúdo para o cliente. Uma maneira de fazer isso é emitir uma tag `<script>` durante a renderização no servidor que define uma variável global como `window.MY_STORE_DATA` e ler dessa variável global no cliente em `getServerSnapshot`. Seu armazenamento externo deve fornecer instruções sobre como fazer isso.

</Note>

---

## Solução de problemas {/*troubleshooting*/}

### Estou recebendo um erro: "O resultado de `getSnapshot` deve ser armazenado em cache" {/*im-getting-an-error-the-result-of-getsnapshot-should-be-cached*/}

Este erro significa que sua função `getSnapshot` retorna um novo objeto toda vez que é chamada, por exemplo:

```js {2-5}
function getSnapshot() {
  // 🔴 Não retorne sempre objetos diferentes de getSnapshot
  return {
    todos: myStore.todos
  };
}
```

O React renderizará novamente o componente se o valor de retorno de `getSnapshot` for diferente da última vez. É por isso que, se você sempre retornar um valor diferente, entrará em um loop infinito e receberá esse erro.

Seu objeto `getSnapshot` só deve retornar um objeto diferente se algo realmente tiver mudado. Se seu armazenamento contiver dados imutáveis, você pode retornar esses dados diretamente:

```js {2-3}
function getSnapshot() {
  // ✅ Você pode retornar dados imutáveis
  return myStore.todos;
}
```

Se seus dados de armazenamento forem mutáveis, sua função `getSnapshot` deverá retornar um snapshot imutável dele. Isso significa que *precisa* criar novos objetos, mas não deve fazer isso para cada chamada. Em vez disso, ele deve armazenar o último snapshot calculado e retornar o mesmo snapshot da última vez se os dados no armazenamento não tiverem mudado. Como você determina se os dados mutáveis foram alterados depende de seu armazenamento mutável.

---

### Minha função `subscribe` é chamada após cada nova renderização {/*my-subscribe-function-gets-called-after-every-re-render*/}

Esta função `subscribe` é definida *dentro* de um componente, então ela é diferente em cada nova renderização:

```js {4-7}
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // 🚩 Sempre uma função diferente, então o React vai se reinscrever em cada renderização
  function subscribe() {
    // ...
  }

  // ...
}
```
  
O React se reinscreverá no seu armazenamento se você passar uma função `subscribe` diferente entre as renderizações. Se isso causar problemas de desempenho e você quiser evitar a reinscrição, mova a função `subscribe` para fora:

```js {6-9}
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅ Sempre a mesma função, então o React não precisará se reinscrever
function subscribe() {
  // ...
}
```

Alternativamente, envolva `subscribe` em [`useCallback`](/reference/react/useCallback) para apenas se reinscrever quando algum argumento mudar:

```js {4-8}
function ChatIndicator({ userId }) {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // ✅ Mesma função contanto que userId não mude
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  // ...
}
```
```