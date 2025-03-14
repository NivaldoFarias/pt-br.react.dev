js
      showNotification('Connected!', theme);
```

One way to fix this is to *extract* the non-reactive logic out from the Effect, and put it into a separate <CodeStep step={1}>event handler</CodeStep>:

```js {4-6}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    function handleConnect() {
      showNotification('Connected!', theme);
    }
    connection.on('connected', handleConnect);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ No theme in the dependencies
  // ...
```

Now, the `handleConnect` <CodeStep step={1}>event handler</CodeStep> is declared inside the Effect, but is not itself reactive. Only the `roomId` matters:

1.  When the connection is established, `handleConnect` will run.
2.  Because `handleConnect` reads the `theme` prop, it "closes over" the current value of `theme`.
3.  Because `handleConnect` is defined inside of the `useEffect`, the `handleConnect` will use the value of `theme` available during the last render.
    *   If you switch the theme after the connection is established and while you're in the chat room, it doesn't matter. The `handleConnect` function has already been defined, so switching themes will not trigger the notification to re-render.

However, this is a bit of a verbose approach. There is a more elegant solution.

## Effect Events {/*effect-events*/}

React provides a built-in mechanism for extracting non-reactive logic out of an Effect, and it is called an <CodeStep step={1}>Effect Event</CodeStep>. *Effect Events* make it easy to perform non-reactive operations from inside an Effect.

Here's how you would rewrite this code using an Effect Event:

```js {4,6}
import { useState, useEffect, useEvent } from 'react';
// ...
function ChatRoom({ roomId, theme }) {
  const handleConnect = useEvent(() => {
    showNotification('Connected!', theme);
  });
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', handleConnect);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

Here, `useEvent` is a Hook that allows you to declare an <CodeStep step={1}>Effect Event</CodeStep>:

1.  The `useEvent` Hook accepts a function as its argument. This function represents the logic you want to run when an event occurs. Because it uses `theme`, you can think of `handleConnect` "closing over" `theme`.
2.  The `handleConnect` function is a "snapshot" of your `theme` prop at the time of the last render. Even if the `theme` value changes later, the `handleConnect` function will always "remember" the correct value of `theme` from *that* time in the past.
3.  `handleConnect` is not reactive: it does not need to be in your Effect's dependencies. It will always use the most recent props and state from the last render.
4.  The `handleConnect` function is stable: it doesn't change between re-renders.

This pattern is useful anytime you want some logic **triggered by an Effect** but **not be reactive**.

### Effect Events "close over" props and state {/*effect-events-close-over-props-and-state*/}

You might be wondering how the code inside of an <CodeStep step={1}>Effect Event</CodeStep> can possibly "remember" the recent value of props or state.

The answer is actually a little bit clever:

1.  React keeps track of the props and state that are used to render each Effect.
2.  When an Effect Event like `handleConnect` is triggered, React passes the correct value of `theme` to the `showNotification` <CodeStep step={1}>Effect Event</CodeStep>.

In this example, you can think of your code as if it were written like this (this is very simplified, but conceptually correct):

```js
function ChatRoom({ roomId, theme }) {
  const handleConnect = () => {
    showNotification('Connected!', theme);
  };
  // ...
```

By using an <CodeStep step={1}>Effect Event</CodeStep>, you're telling React to "remember" the value of `theme` during the last rendering of the `ChatRoom` component. You don't need to declare `theme` as an Effect dependency.

<Note>

Effect Events are a new feature that is available from React 18.3.

</Note>

### Reading the latest props and state from Effects with Effect Events {/*reading-the-latest-props-and-state-from-effects-with-effect-events*/}

Here is another example where an <CodeStep step={1}>Effect Event</CodeStep> is useful.

Imagine you are implementing a component that lets you update the title of the document. You store the title in the `documentTitle` state. You also want the updated title to be saved on local storage in the background. You might try to do it like this:

```js {3-5,8}
import { useState, useEffect } from 'react';

function DocumentTitle() {
  const [documentTitle, setDocumentTitle] = useState('');

  useEffect(() => {
    document.title = documentTitle;
    localStorage.setItem('documentTitle', documentTitle);
  }, [documentTitle]);
  // ...
}
```

This code correctly updates the document title when the `documentTitle` state changes. However, saving to local storage is a performance bottleneck. You don't want to run `localStorage.setItem` on *every* keystroke.

You would want to update local storage only after the user pauses typing for a little while. You might try to add a `setTimeout` call:

```js {3-7,10-12}
import { useState, useEffect } from 'react';

function DocumentTitle() {
  const [documentTitle, setDocumentTitle] = useState('');

  useEffect(() => {
    document.title = documentTitle;
    const timeoutId = setTimeout(() => {
      localStorage.setItem('documentTitle', documentTitle);
    }, 300);
    return () => {
      clearTimeout(timeoutId);
    };
  }, [documentTitle]);
  // ...
}
```

This code correctly "debounces" the local storage update. However, there's a problem: After the user stops typing, the `documentTitle` won't be the most up to date. For example:

1.  The user types "Hello".
2.  `documentTitle` becomes `"Hello"`.
3.  After a 300ms delay, `localStorage.setItem` runs. But, during those 300ms, the user may have continued typing, so `documentTitle` may no longer be `"Hello"`.
4.  The user is surprised to see that the saved title is `"Hello"` instead of `"Hello, world"`.

To fix this, you will need a way to read the latest value of `documentTitle` when `localStorage.setItem` executes. This is what an <CodeStep step={1}>Effect Event</CodeStep> lets you do:

```js {4-7,14-15}
import { useState, useEffect, useEvent } from 'react';

function DocumentTitle() {
  const [documentTitle, setDocumentTitle] = useState('');

  const handleSave = useEvent(() => {
    localStorage.setItem('documentTitle', documentTitle);
  });

  useEffect(() => {
    document.title = documentTitle;
    const timeoutId = setTimeout(() => {
      handleSave();
    }, 300);
    return () => {
      clearTimeout(timeoutId);
    };
  }, [documentTitle]);
  // ...
}
```

Now, even if the `documentTitle` state changes after `setTimeout` has been called, the `handleSave` <CodeStep step={1}>Effect Event</CodeStep> will read the **current** value of `documentTitle` at the time that it is executed. This resolves this problem.

<Recap>

- Event handlers run in response to specific user interactions.
- Effects run in response to changes in reactive dependencies.
- Event handlers handle non-reactive logic.
- Extract non-reactive logic out of an Effect and use an Effect Event to perform this logic.
- Effect Events "close over" the current prop and state values when the event is triggered by an effect.
- You can use Effect Events to read the latest props and state from Effects.

</Recap>
```js
      // ...
      showNotification('Connected!', theme);
      // ...
```

Você precisa de uma forma de separar essa lógica não reativa do `Effect` reativo ao seu redor.

### Declarando um Evento de Effect {/*declaring-an-effect-event*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Use um Hook especial chamado [`useEffectEvent`](/reference/react/experimental_useEffectEvent) para extrair essa lógica não reativa do seu `Effect`:

```js {1,4-6}
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

Aqui, `onConnected` é chamado de um *Evento de Effect.* É uma parte da sua lógica de `Effect`, mas se comporta muito mais como um manipulador de eventos (event handler). A lógica dentro dele não é reativa e sempre "vê" os valores mais recentes de suas props e estado.

Agora você pode chamar o Evento de `Effect` `onConnected` de dentro do seu `Effect`:

```js {2-4,9,13}
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Isso resolve o problema. Observe que você teve que *remover* `onConnected` da lista de dependências do seu `Effect`. **Eventos de Effect não são reativos e devem ser omitidos das dependências.**

Verifique se o novo comportamento funciona como você espera:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Use dark theme
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

Você pode pensar nos Eventos de Effect como sendo muito semelhantes aos manipuladores de eventos. A principal diferença é que os manipuladores de eventos são executados em resposta às interações do usuário, enquanto os Eventos de Effect são acionados por você a partir dos Effects. Os Eventos de Effect permitem que você "quebre a cadeia" entre a reatividade dos Effects e o código que não deve ser reativo.

### Lendo as props e o estado mais recentes com Eventos de Effect {/*reading-latest-props-and-state-with-effect-events*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Os Eventos de Effect permitem que você corrija muitos padrões em que você pode ser tentado a suprimir o linter de dependência.

Por exemplo, digamos que você tenha um `Effect` para registrar as visitas à página:

```js
function Page() {
  useEffect(() => {
    logVisit();
  }, []);
  // ...
}
```

Mais tarde, você adiciona várias rotas ao seu site. Agora, seu componente `Page` recebe uma prop `url` com o caminho atual. Você deseja passar a `url` como parte da sua chamada `logVisit`, mas o linter de dependência reclama:

```js {1,3}
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'url'
  // ...
}
```

Pense sobre o que você quer que o código faça. Você *quer* registrar uma visita separada para URLs diferentes, pois cada URL representa uma página diferente. Em outras palavras, essa chamada `logVisit` *deveria* ser reativa com relação à `url`. É por isso que, nesse caso, faz sentido seguir o linter de dependência e adicionar a `url` como uma dependência:

```js {4}
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // ✅ Todas as dependências declaradas
  // ...
}
```

Agora, digamos que você queira incluir o número de itens no carrinho de compras junto com cada visita à página:

```js {2-3,6}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```

Você usou `numberOfItems` dentro do `Effect`, então o linter pede que você o adicione como uma dependência. No entanto, você *não* deseja que a chamada `logVisit` seja reativa em relação a `numberOfItems`. Se o usuário colocar algo no carrinho de compras e o `numberOfItems` mudar, isso *não significa* que o usuário visitou a página novamente. Em outras palavras, *visitar a página* é, em certo sentido, um "evento". Acontece em um momento preciso no tempo.

Divida o código em duas partes:

```js {5-7,10}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ Todas as dependências declaradas
  // ...
}
```

Aqui, `onVisit` é um Evento de Effect. O código dentro dele não é reativo. É por isso que você pode usar `numberOfItems` (ou qualquer outro valor reativo!) sem se preocupar que isso fará com que o código ao redor seja executado novamente em caso de alterações.

Por outro lado, o próprio `Effect` permanece reativo. O código dentro do `Effect` usa a prop `url`, então o `Effect` será executado novamente após cada rerender com uma `url` diferente. Isso, por sua vez, chamará o Evento de Effect `onVisit`.

Como resultado, você chamará `logVisit` para cada alteração na `url` e sempre lerá o último `numberOfItems`. No entanto, se `numberOfItems` mudar por conta própria, isso não fará com que nenhum dos códigos seja executado novamente.

<Note>

Você pode estar se perguntando se poderia chamar `onVisit()` sem argumentos e ler a `url` dentro dele:

```js {2,6}
  const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
  });

  useEffect(() => {
    onVisit();
  }, [url]);
```

Isso funcionaria, mas é melhor passar essa `url` para o Evento de `Effect` explicitamente. **Ao passar `url` como um argumento para seu Evento de Effect, você está dizendo que visitar uma página com uma `url` diferente constitui um "evento" separado da perspectiva do usuário.** A `visitedUrl` é uma *parte* do "evento" que aconteceu:

```js {1-2,6}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
```

Como seu Evento de Effect "pergunta" explicitamente pela `visitedUrl`, agora você não pode remover acidentalmente a `url` das dependências do `Effect`. Se você remover a dependência `url` (fazendo com que as visitas distintas à página sejam contadas como uma), o linter o avisará sobre isso. Você quer que `onVisit` seja reativo em relação à `url`, então, em vez de ler a `url` dentro (onde não seria reativo), você a passa *do* seu `Effect`.

Isso se torna especialmente importante se houver alguma lógica assíncrona dentro do `Effect`:

```js {6,8}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    setTimeout(() => {
      onVisit(url);
    }, 5000); // Atrasar o registro de visitas
  }, [url]);
```

Aqui, a `url` dentro de `onVisit` corresponde à *última* `url` (que já pode ter mudado), mas `visitedUrl` corresponde à `url` que originalmente causou a execução desse `Effect` (e essa chamada `onVisit`).

</Note>

<DeepDive>

#### É aceitável suprimir o linter de dependência em vez disso? {/*is-it-okay-to-suppress-the-dependency-linter-instead*/}

Nas bases de código existentes, você pode às vezes ver a regra de lint suprimida desta forma:

```js {7-9}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 Evite suprimir o linter desta forma:
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [url]);
  // ...
}
```

Depois que `useEffectEvent` se tornar uma parte estável do React, recomendamos **nunca suprimir o linter**.

A primeira desvantagem de suprimir a regra é que o React não o avisará mais quando seu `Effect` precisar "reagir" a uma nova dependência reativa que você introduziu em seu código. No exemplo anterior, você adicionou a `url` às dependências *porque* o React lembrou você de fazê-lo. Você não receberá mais esses lembretes para quaisquer edições futuras nesse `Effect` se desativar o linter. Isso leva a erros.

Aqui está um exemplo de um bug confuso causado pela supressão do linter. Neste exemplo, a função `handleMove` deve ler o valor atual da variável de estado `canMove` para decidir se o ponto deve seguir o cursor. No entanto, `canMove` é sempre `true` dentro de `handleMove`.

Consegue ver o porquê?

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        The dot is allowed to move
      </label>
      <hr />
      <div style={{
        position: 'absolute',
        backgroundColor: 'pink',
        borderRadius: '50%',
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: 'none',
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }} />
    </>
  );
}
```

```css
body {
  height: 200px;
}
```

</Sandpack>

O problema com esse código está em suprimir o linter de dependência. Se você remover a supressão, verá que esse `Effect` deve depender da função `handleMove`. Isso faz sentido: `handleMove` é declarado dentro do corpo do componente, o que o torna um valor reativo. Todo valor reativo deve ser especificado como uma dependência ou pode ficar obsoleto com o tempo!

O autor do código original "mentiu" para o React dizendo que o `Effect` não depende (`[]`) de nenhum valor reativo. É por isso que o React não ressincronizou o `Effect` depois que `canMove` foi alterado (e `handleMove` com ele). Como o React não ressincronizou o `Effect`, o `handleMove` anexado como um ouvinte é a função `handleMove` criada durante a renderização inicial. Durante a renderização inicial, `canMove` era `true`, e é por isso que `handleMove` da renderização inicial sempre verá esse valor.

**Se você nunca suprimir o linter, nunca verá problemas com valores obsoletos.**

Com `useEffectEvent`, não há necessidade de "mentir" para o linter e o código funciona como você esperaria:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```
```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  const onMove = useEffectEvent(e => {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  });

  useEffect(() => {
    window.addEventListener('pointermove', onMove);
    return () => window.removeEventListener('pointermove', onMove);
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        The dot is allowed to move
      </label>
      <hr />
      <div style={{
        position: 'absolute',
        backgroundColor: 'pink',
        borderRadius: '50%',
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: 'none',
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }} />
    </>
  );
}
```

```css
body {
  height: 200px;
}
```

</Sandpack>

This doesn't mean that `useEffectEvent` is *always* the correct solution. You should only apply it to the lines of code that you don't want to be reactive. In the above sandbox, you didn't want the Effect's code to be reactive with regards to `canMove`. That's why it made sense to extract an Effect Event.

Read [Removing Effect Dependencies](/learn/removing-effect-dependencies) for other correct alternatives to suppressing the linter.

</DeepDive>

### Limitations of Effect Events {/*limitations-of-effect-events*/}

<Wip>

This section describes an **experimental API that has not yet been released** in a stable version of React.

</Wip>

Effect Events are very limited in how you can use them:

*   **Only call them from inside Effects.**
*   **Never pass them to other components or Hooks.**

For example, don't declare and pass an Effect Event like this:

```js {4-6,8}
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 Avoid: Passing Effect Events

  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback();
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay, callback]); // Need to specify "callback" in dependencies
}
```

Instead, always declare Effect Events directly next to the Effects that use them:

```js {10-12,16,21}
function Timer() {
  const [count, setCount] = useState(0);
  useTimer(() => {
    setCount(count + 1);
  }, 1000);
  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ Good: Only called locally inside an Effect
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // No need to specify "onTick" (an Effect Event) as a dependency
}
```

Effect Events are non-reactive "pieces" of your Effect code. They should be next to the Effect using them.

<Recap>

-   Event handlers run in response to specific interactions.
-   Effects run whenever synchronization is needed.
-   Logic inside event handlers is not reactive.
-   Logic inside Effects is reactive.
-   You can move non-reactive logic from Effects into Effect Events.
-   Only call Effect Events from inside Effects.
-   Don't pass Effect Events to other components or Hooks.

</Recap>

<Challenges>

#### Fix a variable that doesn't update {/*fix-a-variable-that-doesnt-update*/}

This `Timer` component keeps a `count` state variable which increases every second. The value by which it's increasing is stored in the `increment` state variable. You can control the `increment` variable with the plus and minus buttons.

However, no matter how many times you click the plus button, the counter is still incremented by one every second. What's wrong with this code? Why is `increment` always equal to `1` inside the Effect's code? Find the mistake and fix it.

<Hint>

To fix this code, it's enough to follow the rules.

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

As usual, when you're looking for bugs in Effects, start by searching for linter suppressions.

If you remove the suppression comment, React will tell you that this Effect's code depends on `increment`, but you "lied" to React by claiming that this Effect does not depend on any reactive values (`[]`). Add `increment` to the dependency array:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, [increment]);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

Now, when `increment` changes, React will re-synchronize your Effect, which will restart the interval.

</Solution>

#### Fix a freezing counter {/*fix-a-freezing-counter*/}

This `Timer` component keeps a `count` state variable which increases every second. The value by which it's increasing is stored in the `increment` state variable, which you can control it with the plus and minus buttons. For example, try pressing the plus button nine times, and notice that the `count` now increases each second by ten rather than by one.

There is a small issue with this user interface. You might notice that if you keep pressing the plus or minus buttons faster than once per second, the timer itself seems to pause. It only resumes after a second passes since the last time you've pressed either button. Find why this is happening, and fix the issue so that the timer ticks on *every* second without interruptions.

<Hint>

It seems like the Effect which sets up the timer "reacts" to the `increment` value. Does the line that uses the current `increment` value in order to call `setCount` really need to be reactive?

</Hint>

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, [increment]);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

The issue is that the code inside the Effect uses the `increment` state variable. Since it's a dependency of your Effect, every change to `increment` causes the Effect to re-synchronize, which causes the interval to clear. If you keep clearing the interval every time before it has a chance to fire, it will appear as if the timer has stalled.

To solve the issue, extract an `onTick` Effect Event from the Effect:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, []);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

Since `onTick` is an Effect Event, the code inside it isn't reactive. The change to `increment` does not trigger any Effects.

</Solution>

#### Fix a non-adjustable delay {/*fix-a-non-adjustable-delay*/}

In this example, you can customize the interval delay. It's stored in a `delay` state variable which is updated by two buttons. However, even if you press the "plus 100 ms" button until the `delay` is 1000 milliseconds (that is, a second), you'll notice that the timer still increments very fast (every 100 ms). It's as if your changes to the `delay` are ignored. Find and fix the bug.

<Hint>

Code inside Effect Events is not reactive. Are there cases in which you would *want* the `setInterval` call to re-run?

</Hint>

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);
  const [delay, setDelay] = useState(100);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  const onMount = useEffectEvent(() => {
    return setInterval(() => {
      onTick();
    }, delay);
  });

  useEffect(() => {
    const id = onMount();
    return () => {
      clearInterval(id);
    }
  }, []);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        Increment delay:
        <button disabled={delay === 100} onClick={() => {
          setDelay(d => d - 100);
        }}>–100 ms</button>
        <b>{delay} ms</b>
        <button onClick={() => {
          setDelay(d => d + 100);
        }}>+100 ms</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

The problem with the above example is that it extracted an Effect Event called `onMount` without considering what the code should actually be doing. You should only extract Effect Events for a specific reason: when you want to make a part of your code non-reactive. However, the `setInterval` call *should* be reactive with respect to the `delay` state variable. If the `delay` changes, you want to set up the interval from scratch! To fix this code, pull all the reactive code back inside the Effect:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);
  const [delay, setDelay] = useState(100);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, delay);
    return () => {
      clearInterval(id);
    }
  }, [delay]);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        Increment delay:
        <button disabled={delay === 100} onClick={() => {
          setDelay(d => d - 100);
        }}>–100 ms</button>
        <b>{delay} ms</b>
        <button onClick={() => {
          setDelay(d => d + 100);
        }}>+100 ms</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

In general, you should be suspicious of functions like `onMount` that focus on the *timing* rather than the *purpose* of a piece of code. It may feel "more descriptive" at first but it obscures your intent. As a rule of thumb, Effect Events should correspond to something that happens from the *user's* perspective. For example, `onMessage`, `onTick`, `onVisit`, or `onConnected` are good Effect Event names. Code inside them would likely not need to be reactive. On the other hand, `onMount`, `onUpdate`, `onUnmount`, or `onAfterRender` are so generic that it's easy to accidentally put code that *should* be reactive into them. This is why you should name your Effect Events after *what the user thinks has happened,* not when some code happened to run.

</Solution>

#### Fix a delayed notification {/*fix-a-delayed-notification*/}

When you join a chat room, this component shows a notification. However, it doesn't show the notification immediately. Instead, the notification is artificially delayed by two seconds so that the user has a chance to look around the UI.

This almost works, but there is a bug. Try changing the dropdown from "general" to "travel" and then to "music" very quickly. If you do it fast enough, you will see two notifications (as expected!) but they will *both* say "Welcome to music".

Fix it so that when you switch from "general" to "travel" and then to "music" very quickly, you see two notifications, the first one being "Welcome to travel" and the second one being "Welcome to music". (For an additional challenge, assuming you've *already* made the notifications show the correct rooms, change the code so that only the latter notification is displayed.)

<Hint>

Your Effect knows which room it connected to. Is there any information that you might want to pass to your Effect Event?

</Hint>

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Welcome to ' + roomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected();
      }, 2000);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Use dark theme
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution>

Inside your Effect Event, `roomId` is the value *at the time Effect Event was called.*

Your Effect Event is called with a two second delay. If you're quickly switching from the travel to the music room, by the time the travel room's notification shows, `roomId` is already `"music"`. This is why both notifications say "Welcome to music".

To fix the issue, instead of reading the *latest* `roomId` inside the Effect Event, make it a parameter of your Effect Event, like `connectedRoomId` below. Then pass `roomId` from your Effect by calling `onConnected(roomId)`:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(connectedRoomId => {
    showNotification('Welcome to ' + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected(roomId);
      }, 2000);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Use dark theme
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

The Effect that had `roomId` set to `"travel"` (so it connected to the `"travel"` room) will show the notification for `"travel"`. The Effect that had `roomId` set to `"music"` (so it connected to the `"music"` room) will show the notification for `"music"`. In other words, `connectedRoomId` comes from your Effect (which is reactive), while `theme` always uses the latest value.

To solve the additional challenge, save the notification timeout ID and clear it in the cleanup function of your Effect:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(connectedRoomId => {
    showNotification('Welcome to ' + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    let notificationTimeoutId;
    connection.on('connected', () => {
      notificationTimeoutId = setTimeout(() => {
        onConnected(roomId);
      }, 2000);
    });
    connection.connect();
    return () => {
      connection.disconnect();
      if (notificationTimeoutId !== undefined) {
        clearTimeout(notificationTimeoutId);
      }
    };
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Use dark theme
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

This ensures that already scheduled (but not yet displayed) notifications get cancelled when you change rooms.

</Solution>

</Challenges>