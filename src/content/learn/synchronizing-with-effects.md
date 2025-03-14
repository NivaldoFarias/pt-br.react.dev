js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

In other words, the Effect now only re-runs when `isPlaying` changes. Specifying dependencies is like telling React: "Hey, I only need to re-run this Effect if these particular values changed since the last time. If they didn't, you can skip it."

Here are some rules to follow when declaring dependencies:

*   **If an Effect uses any value from the component's scope (props, state, and all variables declared inside the component), you need to list it in the dependency array.** The rule isn't just that you *read* a dependency. The rule is that if some *code* inside your Effect *uses* a prop, state or a variable defined inside your component, it must be in the dependency array.
*   **If you provide a dependency array, you must include *everything* the Effect uses, unless the value is:
    *   A primitive value (like a string or a number) that doesn't change over time.
    *   Declared *inside* the Effect itself.
    *   A function that's defined *outside* the component.
    *   A prop of the component that never changes.

If you forget to list a dependency correctly, your component might:

*   **Be buggy.** Your Effect might refer to stale props and state, resulting in a bug.
*   **Be inefficient.** Your Effect might run more often than necessary.

<DeepDive>

#### Dependency arrays and the "stale values" problem

Sometimes, the most confusing thing about Effects is the concept of "stale values".

```js
function MyComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // The problem: this code "sees" the count from the initial render
    function handleTimeout() {
      alert('You clicked ' + count + ' times!');
    }
    setTimeout(handleTimeout, 3000);
  }, []);

  return (
    <button onClick={() => setCount(count + 1)}>
      Click me
    </button>
  );
}
```

In this example, the [`handleTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) function refers to the `count` state. However, because the Effect has an empty dependency array (`[]`), the `handleTimeout` function is only created once, during the initial render. Consequently, even if you click the button, the value inside the `handleTimeout` does not update. It will always output the initial value of `count`. This is sometimes referred to as the "stale values" problem.

To fix the example above, add `count` as a dependency of the Effect:

```js
useEffect(() => {
  // This fixes the problem: a new function is created after every render
  function handleTimeout() {
    alert('You clicked ' + count + ' times!');
  }
  setTimeout(handleTimeout, 3000);
}, [count]);
```

Now the callback can access the `count` for the current render.

</DeepDive>

### Step 3: Add cleanup if needed {/*step-3-add-cleanup-if-needed*/}

Some Effects need to specify "how to clean up" after themselves.

For example, if your component displays a chat room, you'll likely start a connection to the chat server when the component appears. However, when your component no longer needs to display the chat room, you'll want to stop the connection.

To add cleanup logic to an Effect, **return a function from your Effect:**

```js
useEffect(() => {
  // 1. Set up the Effect
  const connection = createConnection();
  connection.connect();

  // 2. Specify how to clean up
  return () => {
    connection.disconnect();
  };
}, []);
```

The function you return from the Effect runs **every time the component is removed from the screen**, as well as before the next time the Effect runs. In other words, it runs **whenever the component is unmounted or before re-running the Effect due to a dependency change.**

Let's see how it works with a simplified `ChatRoom` component:

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

function createConnection(roomId) {
  // A real implementation would connect to the server
  return {
    connect() {
      console.log('✅ Connecting to chat room ' + roomId + '...');
    },
    disconnect() {
      console.log('❌ Disconnecting from chat room ' + roomId);
    },
  };
}
```

There are a few interesting things happening in this component:

1.  When the `ChatRoom` component appears on the screen (because you rendered it), the Effect runs. It calls `createConnection(roomId)`, which returns a connection object, and then calls `connection.connect()`.
2.  React remembers the cleanup function you've passed.
3.  If the `ChatRoom` component is removed from the screen (because you stopped rendering it), React will call the cleanup function. This will call `connection.disconnect()`.
4.  If the `ChatRoom` component re-renders (because `roomId` has changed), React will first call the cleanup function with the old `roomId`, and then run the Effect again with the new `roomId`.

Note that the dependency array `[roomId]` contains `roomId`. This tells React to re-synchronize the chat connection when the `roomId` prop changes.

```js
<ChatRoom roomId="general" />
```

When you change the `roomId` prop, React calls the `disconnect()` function first and then sets up the new connection. This ensures that you never connect to the wrong chat room.

Here is a full example with a `ChatRoom` and a component that switches the `roomId`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

function createConnection(roomId) {
  // Uma implementação real se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando à sala de chat ' + roomId + '...');
    },
    disconnect() {
      console.log('❌ Desconectando da sala de chat ' + roomId);
    },
  };
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={(e) => setRoomId(e.target.value)}
        >
          <option value="general">Geral</option>
          <option value="travel">Viagens</option>
          <option value="music">Música</option>
        </select>
      </label>
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

</Sandpack>

This pattern of "setting up" and "tearing down" is common in React. If you think about the example above, connecting and disconnecting is very similar to subscribing and unsubscribing to an event listener.

Remember that **cleanup functions run only when the component unmounts, or when its dependencies change.**

<Recipes title="Common cleanup patterns" parentId="synchronizing-with-effects">

#### Subscribing to an external source {/*subscribing-to-an-external-source*/}

Here is an example of a `FriendStatus` component which subscribes to the friend's online status:

```js
function FriendStatus({ friendId }) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    chatAPI.subscribeToFriendStatus(friendId, handleStatusChange);
    return () => {
      chatAPI.unsubscribeFromFriendStatus(friendId, handleStatusChange);
    };
  }, [friendId]);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

// Fake chat API
const chatAPI = {
  subscribeToFriendStatus: (friendId, handleStatusChange) => {
    // Mock implementation
     console.log('✅ [API] Subscribed to ' + friendId);
    // Simulate an update every second
    setInterval(() => {
      const isOnline = Math.random() > 0.5;
      handleStatusChange({ isOnline });
    }, 1000);
  },
  unsubscribeFromFriendStatus: (friendId, handleStatusChange) => {
     console.log('❌ [API] Unsubscribed from ' + friendId);
  }
};
```

In this example, the Effect sets up a subscription using `chatAPI.subscribeToFriendStatus`. It also returns a cleanup function that unsubscribes using `chatAPI.unsubscribeFromFriendStatus`. The dependency array is `[friendId]`, so the Effect updates if `friendId` changes, and unsubscribes from the previous friend.

<Sandpack>

```js
import { useState, useEffect } from 'react';

function FriendStatus({ friendId }) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    chatAPI.subscribeToFriendStatus(friendId, handleStatusChange);
    return () => {
      chatAPI.unsubscribeFromFriendStatus(friendId, handleStatusChange);
    };
  }, [friendId]);

  if (isOnline === null) {
    return 'Carregando...';
  }
  return isOnline ? 'Online' : 'Offline';
}

// API de chat falsa
const chatAPI = {
  subscribeToFriendStatus: (friendId, handleStatusChange) => {
    // Implementação mock
     console.log('✅ [API] Assinado a ' + friendId);
    // Simula uma atualização a cada segundo
    setInterval(() => {
      const isOnline = Math.random() > 0.5;
      handleStatusChange({ isOnline });
    }, 1000);
  },
  unsubscribeFromFriendStatus: (friendId, handleStatusChange) => {
     console.log('❌ [API] Cancelada a assinatura de ' + friendId);
  }
};

export default function App() {
  const [friend, setFriend] = useState('1');
  return (
    <>
      <label>
        Escolha um amigo:{' '}
        <select value={friend} onChange={e => setFriend(e.target.value)}>
          <option value="1">Amigo 1</option>
          <option value="2">Amigo 2</option>
          <option value="3">Amigo 3</option>
        </select>
      </label>
      <FriendStatus friendId={friend} />
    </>
  );
}
```

</Sandpack>

#### Fetching data {/*fetching-data*/}

Here is an example of how to fetch data from a network endpoint using the browser [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) API:

```js
function MyComponent({ userId }) {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      const response = await fetch('https://.../users/' + userId);
      const json = await response.json();
      setUserData(json);
    }

    fetchData();
  }, [userId]);

  if (userData === null) {
    return 'Loading...';
  }
  return (
    <h1>{userData.name}</h1>
  );
}
```

In this example, the Effect calls the `fetchData()` function, which uses [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) to get data from a network endpoint. The dependency array is `[userId]`, so the code will refetch the data when `userId` changes.

<Sandpack>

```js
import { useState, useEffect } from 'react';

function MyComponent({ userId }) {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      const response = await fetch('https://api.github.com/users/' + userId);
      const json = await response.json();
      setUserData(json);
    }

    fetchData();
  }, [userId]);

  if (userData === null) {
    return 'Carregando...';
  }
  return (
    <>
      <img src={userData.avatar_url} alt="Avatar" />
      <h1>{userData.name}</h1>
    </>
  );
}

export default function App() {
  const [userId, setUserId] = useState('octocat');
  return (
    <>
      <label>
        Escolha um usuário do GitHub:{' '}
        <select value={userId} onChange={e => setUserId(e.target.value)}>
          <option value="octocat">octocat</option>
          <option value="gaearon">gaearon</option>
          <option value="sophiebits">sophiebits</option>
        </select>
      </label>
      <MyComponent userId={userId} />
    </>
  );
}
```

</Sandpack>

#### Using a timeout or interval {/*using-a-timeout-or-interval*/}

Here is an example component that uses `setTimeout` to display a message after a delay:

```js
function MyComponent() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setMessage('Hello!');
    }, 1000);

    return () => {
      clearTimeout(timeoutId);
    };
  }, []);

  return (
    <h1>{message}</h1>
  );
}
```

The code uses [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) to set a timeout. It then returns a cleanup function which uses [`clearTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/clearTimeout) to clear the timeout if the component unmounts before the timeout triggers. The dependency array is `[]`, so the timeout is only set up once when the component mounts.

<Sandpack>

```js
import { useState, useEffect } from 'react';

function MyComponent() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setMessage('Olá!');
    }, 1000);

    return () => {
      clearTimeout(timeoutId);
    };
  }, []);

  return (
    <h1>{message}</h1>
  );
}

export default function App() {
  return <MyComponent />;
}
```

</Sandpack>

A similar approach can be used to manage intervals. In this case, the cleanup function will use [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) to clear the interval.

</Recipes>

<Pitfall>

It's tempting to put `async` directly on the `useEffect` function, but this doesn't work. Instead, you should define an inner `async` function inside the Effect.

```js
useEffect(async () => { // 🔴 Não funciona!
  const response = await someAPI.getData();
  // ...
}, []);

useEffect(() => { // ✅ Funciona!
  async function fetchData() {
    const response = await someAPI.getData();
    // ...
  }
  fetchData();
}, []);
```

</Pitfall>

## Effects run twice in development {/*effects-run-twice-in-development*/}

To help you find bugs, **React calls the code of your component *twice* in development.** This helps you ensure that your Effects are idempotent, meaning they don't cause the same effect more than once.

For example, consider this `ChatRoom` component:

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Imagine you're working on this component, and then you see in the console that `connect` is called twice. This is because React re-runs all Effects to simulate the component re-mounting as if it was a completely new component. This check helps you catch mistakes like, for example, if your `connect` call makes a duplicate subscription. If you implemented this Effect correctly, calling `connect()` twice in rapid succession shouldn't matter. It should be idempotent. To make your Effects idempotent, consider these suggestions:

*   **If your Effect only *sets* a value,** it is often already idempotent. For example, `document.title = 'Hello';` can be called many times with no harm.
*   **If your Effect *mutates* something,** consider how to write the effect so that re-running it again with the same values does not cause problems. For example, suppose you need to send an analytics event. Sending one analytic even is not a problem, but sending it multiple times for the same navigation event is a potential bug. If you call `sendEvent('navigation', { page: '...' });` multiple times, you'll want to make sure it only tracks each navigation once.
*   **If your Effect connects to an external system,** ensure it can handle being called multiple times. For example, the `connect()` method in the chat room example should only allow you to connect once, and calling it repeatedly should not result in errors. Your cleanup function should also work as expected.

In production, Effects run only once. If this is causing a problem, you can disable the double-rendering behavior by [wrapping the component in `<React.StrictMode>`](/reference/react/StrictMode). However, it's recommended to fix your Effects rather than disable this behavior.

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

O array de dependências pode conter diversas dependências. React irá pular a re-execução do Effect somente se *todas* as dependências especificadas tiverem exatamente os mesmos valores que tinham na renderização anterior. React compara os valores da dependência usando a comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Veja a [`referência do useEffect`](/reference/react/useEffect#reference) para detalhes.

**Observe que você não pode “escolher” suas dependências.** Você receberá um erro de lint se as dependências que você especificou não corresponderem ao que o React espera com base no código dentro do seu Effect. Isso ajuda a detectar muitos erros no seu código. Se você não deseja que algum código seja reexecutado, [*edite o próprio código do Effect* para que ele não "precise" dessa dependência.](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

<Pitfall>

Os comportamentos sem o array de dependências e com um array de dependências `[]` *vazio* são diferentes:

```js {3,7,11}
useEffect(() => {
  // This runs after every render
});

useEffect(() => {
  // This runs only on mount (when the component appears)
}, []);

useEffect(() => {
  // This runs on mount *and also* if either a or b have changed since the last render
}, [a, b]);
```

Nós daremos uma olhada mais de perto no que "montar/montagem" significa no próximo passo.

</Pitfall>

<DeepDive>

#### Por que a ref foi omitida do array de dependências? {/*why-was-the-ref-omitted-from-the-dependency-array*/}

Esse Effect usa _ambos_ `ref` e `isPlaying`, mas somente `isPlaying` é declarada como uma dependência:

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);
```

Isso ocorre porque o objeto `ref` possui uma *identidade estável:* React garante que [você sempre obterá o mesmo objeto](/reference/react/useRef#returns) da mesma chamada `useRef` em cada renderização. Ele nunca muda, então, por si só, ele nunca fará com que o Effect seja re-executado. Portanto, não importa se você o inclui ou não. Incluí-lo também é bom:

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying, ref]);
```

As funções [`set`](/reference/react/useState#setstate) retornadas por `useState` também possuem identidade estável, então você frequentemente as verá omitidas das dependências também. Se o linter permitir que você omita uma dependência sem erros, é seguro fazê-lo.

A omissão de dependências sempre estáveis só funciona quando o linter pode "ver" que o objeto é estável. Por exemplo, se `ref` foi passado de um componente pai, você teria que especificá-lo no array de dependências. No entanto, isso é bom, pois você não pode saber se o componente pai sempre passa a mesma ref ou passa uma de várias refs condicionalmente. Portanto, seu Effect _dependeria_ de qual ref é passada.

</DeepDive>

### Passo 3: Adicione a limpeza (cleanup) se necessário {/*step-3-add-cleanup-if-needed*/}

Considere um exemplo diferente. Você está escrevendo um componente `ChatRoom` que precisa se conectar ao servidor de chat quando ele aparece. Você recebeu uma API `createConnection()` que retorna um objeto com os métodos `connect()` e `disconnect()`. Como você mantém o componente conectado enquanto ele é exibido para o usuário?

Comece escrevendo a lógica do Effect:

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Seria lento se conectar ao chat após cada re-renderização, então adicione o array de dependência:

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**O código dentro do Effect não usa nenhuma prop ou state, portanto, seu array de dependências é `[]` (vazio). Isso diz ao React para executar este código somente quando o componente "montar/montagem", ou seja, aparecer na tela pela primeira vez.**

Vamos tentar executar este código:

<Sandpack>

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>Bem-vindo ao chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando...');
    },
    disconnect() {
      console.log('❌ Desconectado.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

Este Effect só é executado na montagem/montagem, então você pode esperar que `“✅ Conectando...”` seja impresso uma vez no console. **No entanto, se você verificar o console, `“✅ Conectando...”` é impresso duas vezes. Por que isso acontece?**

Imagine que o componente `ChatRoom` faz parte de um aplicativo maior com muitas telas diferentes. O usuário inicia sua jornada na página `ChatRoom`. O componente monta e chama `connection.connect()`. Em seguida, imagine que o usuário navega para outra tela -- por exemplo, a página de Configurações. O componente `ChatRoom` desmonta. Finalmente, o usuário clica em Voltar e `ChatRoom` monta novamente. Isso configuraria uma segunda conexão -- mas a primeira conexão nunca foi destruída! À medida que o usuário navega pelo aplicativo, as conexões continuariam a se acumular.

Erros como esse são fáceis de perder sem extensos testes manuais. Para ajudá-lo a detectá-los rapidamente, no desenvolvimento o React remonta/remonta cada componente uma vez imediatamente após sua montagem/montagem inicial.

Ver o log `"✅ Conectando..."` duas vezes ajuda você a notar o problema real: seu código não fecha a conexão quando o componente desmonta/desmontagem.

Para corrigir o problema, retorne uma *função de limpeza* do seu Effect:

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

React chamará sua função de limpeza cada vez antes que o Effect seja executado novamente e, por fim, quando o componente desmontar/desmontagem (for removido). Vamos ver o que acontece quando a função de limpeza é implementada:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Bem-vindo ao chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando...');
    },
    disconnect() {
      console.log('❌ Desconectado.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

Agora você obtém três logs de console no desenvolvimento:

1. `"✅ Conectando..."`
2. `"❌ Desconectado."`
3. `"✅ Conectando..."`

**Este é o comportamento correto no desenvolvimento.** Ao remontar/remontar seu componente, React verifica se navegar para fora e para trás não irá quebrar seu código. Desconectar e, em seguida, conectar novamente é exatamente o que deveria acontecer! Quando você implementa a limpeza bem, não deve haver diferença visível pelo usuário entre executar o Effect uma vez e executá-lo, limpá-lo e executá-lo novamente. Há um par de chamadas de conexão/desconexão extra porque o React está testando seu código para erros no desenvolvimento. Isso é normal -- não tente fazer com que desapareça!

**Na produção, você veria apenas `"✅ Conectando..."` sendo impresso uma vez.** Remontar/remontar componentes só acontece no desenvolvimento para ajudá-lo a encontrar Effects que precisam de limpeza. Você pode desativar o [Modo Estrito](/reference/react/StrictMode) para desativar o comportamento de desenvolvimento, mas recomendamos mantê-lo ativado. Isso permite que você encontre muitos erros, como o acima.

## Como lidar com o Effect sendo disparado duas vezes no desenvolvimento? {/*how-to-handle-the-effect-firing-twice-in-development*/}

React remonta/remonta intencionalmente seus componentes no desenvolvimento para encontrar erros, como no exemplo anterior. **A pergunta certa não é "como executar um Effect uma vez", mas "como consertar meu Effect para que ele funcione após a remontagem/remontagem".**

Geralmente, a resposta é implementar a função de limpeza. A função de limpeza deve parar ou desfazer o que o Effect estava fazendo. A regra geral é que o usuário não deve ser capaz de distinguir entre o Effect sendo executado uma vez (como na produção) e uma sequência de _configurações → limpeza → configuração_ (como você veria no desenvolvimento).

A maioria dos Effects que você escreverá se encaixará em um dos padrões comuns abaixo.

<Pitfall>

#### Não use refs para evitar que os Effects sejam disparados {/*dont-use-refs-to-prevent-effects-from-firing*/}

Uma armadilha comum para impedir que os Effects disparem duas vezes no desenvolvimento é usar uma `ref` para impedir que o Effect seja executado mais de uma vez. Por exemplo, você poderia "consertar" o erro acima com um `useRef`:

```js {1,3-4}
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 Isso não corrigirá o erro!!!
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

Isso faz com que você veja apenas `"✅ Conectando..."` uma vez no desenvolvimento, mas não corrige o erro.

Quando o usuário navega para fora, a conexão ainda não é fechada e, quando ele navega de volta, uma nova conexão é criada. À medida que o usuário navega pelo aplicativo, as conexões continuariam a se acumular, da mesma forma que antes da "correção".

Para corrigir o erro, não basta apenas fazer com que o Effect seja executado uma vez. O effect precisa funcionar após a remontagem/remontagem, o que significa que a conexão precisa ser limpa como na solução acima.

Veja os exemplos abaixo para saber como lidar com padrões comuns.

</Pitfall>

### Controlando widgets não-React {/*controlling-non-react-widgets*/}

Às vezes, você precisa adicionar widgets de UI que não foram escritos em React. Por exemplo, digamos que você esteja adicionando um componente de mapa à sua página. Ele tem um método `setZoomLevel()`, e você gostaria de manter o nível de zoom sincronizado com uma variável de state `zoomLevel` no seu código React. Seu Effect seria semelhante a este:

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Observe que nenhuma limpeza é necessária neste caso. No desenvolvimento, o React chamará o Effect duas vezes, mas isso não é um problema porque chamar `setZoomLevel` duas vezes com o mesmo valor não faz nada. Pode ser um pouco mais lento, mas isso não importa porque ele não remontará/remontará desnecessariamente na produção.

Algumas APIs podem não permitir que você as chame duas vezes seguidas. Por exemplo, o método [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) do elemento [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) integrado lança um erro se você o chamar duas vezes. Implemente a função de limpeza e faça com que ela feche a caixa de diálogo:

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

No desenvolvimento, seu Effect chamará `showModal()`, depois `close()` imediatamente, e depois `showModal()` novamente. Isso possui o mesmo comportamento visível pelo usuário de chamar `showModal()` uma vez, como você veria na produção.

### Assinando eventos {/*subscribing-to-events*/}

Se seu Effect assinar algo, a função de limpeza deve cancelar a inscrição:

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

No desenvolvimento, seu Effect chamará `addEventListener()`, então `removeEventListener()` imediatamente e, em seguida, `addEventListener()` novamente com o mesmo manipulador. Portanto, haveria apenas uma assinatura ativa por vez. Isso possui o mesmo comportamento visível pelo usuário de chamar `addEventListener()` uma vez, como na produção.

### Acionando animações {/*triggering-animations*/}

Se o Effect animar algo, a função de limpeza deve redefinir a animação para os valores iniciais:

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

No desenvolvimento, a opacidade será definida como `1`, depois como `0` ​​e, em seguida, como `1` novamente. Isso deve ter o mesmo comportamento visível pelo usuário de defini-lo como `1` diretamente, que é o que aconteceria na produção. Se você usar uma biblioteca de animação de terceiros com suporte a tweening, sua função de limpeza deverá redefinir a linha do tempo para seu estado inicial.

### Buscando dados {/*fetching-data*/}

Se seu Effect buscar algo, a função de limpeza deve [abortar a busca](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) ou ignorar seu resultado:

```js {2,6,13-15}
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

Você não pode "desfazer" uma solicitação de rede que já aconteceu, mas sua função de limpeza deve garantir que a busca que _não é mais relevante_ não continue afetando seu aplicativo. Se `userId` mudar de `'Alice'` para `'Bob'`, a limpeza garante que a resposta `'Alice'` seja ignorada, mesmo que ela chegue depois de `'Bob'`.

**No desenvolvimento, você verá duas buscas na guia Rede.** Não há nada de errado com isso. Com a abordagem acima, o primeiro Effect será imediatamente limpo para que sua cópia da variável `ignore` seja definida como `true`. Portanto, mesmo que haja uma solicitação extra, isso não afetará o state graças à verificação `if (!ignore)`.

**Na produção, haverá apenas uma solicitação.** Se a segunda solicitação no desenvolvimento estiver incomodando você, a melhor abordagem é usar uma solução que deduplica as solicitações e armazena em cache suas respostas entre os componentes:

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

Isso não apenas melhorará a experiência de desenvolvimento, mas também fará com que seu aplicativo pareça mais rápido. Por exemplo, o usuário pressionar o botão Voltar não precisará esperar que alguns dados sejam carregados novamente, porque eles serão armazenados em cache. Você pode criar esse cache sozinho ou usar uma das muitas alternativas para buscar dados manualmente nos Effects.

<DeepDive>

#### Quais são as boas alternativas para buscar dados nos Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Escrever chamadas `fetch` dentro dos Effects é uma [maneira popular de buscar dados](https://www.robinwieruch.de/react-hooks-fetch-data/), especialmente em aplicativos totalmente no lado do cliente. Esta é, no entanto, uma abordagem muito manual e tem desvantagens significativas:
- **Effects não executam no servidor.** Isso significa que o HTML inicial renderizado no servidor incluirá apenas um estado de carregamento sem dados. O computador cliente precisará baixar todo o JavaScript e renderizar seu app apenas para descobrir que agora ele precisa carregar os dados. Isso não é muito eficiente.
- **Buscar diretamente nos Effects facilita a criação de "cascatas de rede".** Você renderiza o componente pai, ele busca alguns dados, renderiza os componentes filhos e, em seguida, eles começam a buscar seus dados. Se a rede não for muito rápida, isso é significativamente mais lento do que buscar todos os dados em paralelo.
- **Buscar diretamente nos Effects geralmente significa que você não pré-carrega ou armazena dados em cache.** Por exemplo, se o componente for desmontado e depois montado novamente, ele terá que buscar os dados novamente.
- **Não é muito ergonômico.** Há bastante código boilerplate envolvido ao escrever chamadas `fetch` de forma que não sofram com bugs como [condições de corrida.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

Essa lista de desvantagens não é específica do React. Ela se aplica a busca de dados na montagem com qualquer biblioteca. Assim como no roteamento, a busca de dados não é trivial de ser feita corretamente, por isso, recomendamos as seguintes abordagens:

- **Se você usar um [framework](/learn/start-a-new-react-project#production-grade-react-frameworks), use seu mecanismo de busca de dados integrado.** Frameworks modernos do React têm mecanismos de busca de dados integrados que são eficientes e não sofrem com as armadilhas acima.
- **Caso contrário, considere usar ou construir um cache do lado do cliente.** Soluções de código aberto populares incluem [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/) e [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) Você também pode construir sua própria solução, caso em que usaria Effects nos bastidores, mas adicionaria lógica para remoção de duplicações de requisições, armazenamento em cache de respostas e para evitar cascatas de rede (pré-carregando dados ou elevando os requisitos de dados para rotas).

Você pode continuar buscando dados diretamente nos Effects se nenhuma dessas abordagens for adequada para você.

</DeepDive>

### Enviando analytics {/*sending-analytics*/}

Considere este código que envia um evento de analytics na visita à página:

```js
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);
```

Em desenvolvimento, `logVisit` será chamado duas vezes para cada URL, então você pode ser tentado a tentar corrigir isso. **Recomendamos manter este código como está.** Assim como nos exemplos anteriores, não há diferença de comportamento *visível para o usuário* entre executá-lo uma ou duas vezes. Do ponto de vista prático, `logVisit` não deve fazer nada em desenvolvimento, pois você não quer que os logs das máquinas de desenvolvimento distorçam as métricas de produção. Seu componente é remontado toda vez que você salva o arquivo, então ele registra visitas extras em desenvolvimento de qualquer maneira.

**Em produção, não haverá logs de visitas duplicados.**

Para depurar os eventos de analytics que você está enviando, você pode implantar seu app em um ambiente de staging (que é executado no modo de produção) ou desativar temporariamente o [Strict Mode](/reference/react/StrictMode) e suas verificações de remontagem apenas para desenvolvimento. Você também pode enviar análises dos manipuladores de eventos de mudança de rota em vez de Effects. Para análises mais precisas, [observadores de interseção](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) podem ajudar a rastrear quais componentes estão na viewport e por quanto tempo eles permanecem visíveis.

### Não é um Effect: Inicializando o aplicativo {/*not-an-effect-initializing-the-application*/}

Alguma lógica deve ser executada apenas uma vez quando o aplicativo for iniciado. Você pode colocá-la fora de seus componentes:

```js {2-3}
if (typeof window !== 'undefined') { // Verifique se estamos executando no navegador.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

Isso garante que essa lógica seja executada apenas uma vez após o navegador carregar a página.

### Não é um Effect: Comprando um produto {/*not-an-effect-buying-a-product*/}

Às vezes, mesmo que você escreva uma função de limpeza, não há como evitar as consequências visíveis para o usuário de executar o Effect duas vezes. Por exemplo, talvez seu Effect envie uma requisição POST como comprar um produto:

```js {2-3}
useEffect(() => {
  // 🔴 Errado: Este Effect dispara duas vezes em desenvolvimento, expondo um problema no código.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

Você não gostaria de comprar o produto duas vezes. No entanto, é também por isso que você não deve colocar essa lógica em um Effect. E se o usuário for para outra página e depois pressionar Voltar? Seu Effect seria executado novamente. Você não quer comprar o produto quando o usuário *visita* uma página; você quer comprá-lo quando o usuário *clica* no botão Comprar.

Comprar não é causado pela renderização; é causado por uma interação específica. Ele deve ser executado apenas quando o usuário pressiona o botão. **Exclua o Effect e mova sua requisição `/api/buy` para o manipulador de eventos do botão Comprar:**

```js {2-3}
  function handleClick() {
    // ✅ Comprar é um evento porque é causado por uma interação específica.
    fetch('/api/buy', { method: 'POST' });
  }
```

**Isso ilustra que, se a remontagem quebrar a lógica do seu aplicativo, isso geralmente revela bugs existentes.** Da perspectiva de um usuário, visitar uma página não deve ser diferente de visitá-la, clicar em um link e, em seguida, pressionar Voltar para visualizar a página novamente. O React verifica se seus componentes cumprem esse princípio, remontando-os uma vez em desenvolvimento.

## Juntando tudo {/*putting-it-all-together*/}

Este playground pode ajudar você a "sentir" como os Effects funcionam na prática.

Este exemplo usa [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) para agendar um log no console com o texto de entrada a aparecer três segundos após a execução do Effect. A função de limpeza cancela o tempo limite pendente. Comece pressionando "Montar o componente":

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Agendar log "' + text + '"');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancelar log "' + text + '"');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        O que registrar:{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Desmontar' : 'Montar'} o componente
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

</Sandpack>

Você verá três logs no início: `Agendar log "a"`, `Cancelar log "a"` e `Agendar log "a"` novamente. Três segundos depois, também haverá um log dizendo `a`. Como você aprendeu anteriormente, o par extra de agendar/cancelar ocorre porque o React remonta o componente uma vez em desenvolvimento para verificar se você implementou a limpeza corretamente.

Agora edite a entrada para dizer `abc`. Se você fizer isso rápido o suficiente, verá `Agendar log "ab"` imediatamente seguido por `Cancelar log "ab"` e `Agendar log "abc"`. **O React sempre limpa o Effect da renderização anterior antes do Effect da próxima renderização.** É por isso que, mesmo se você digitar na entrada rapidamente, haverá, no máximo, um tempo limite agendado de cada vez. Edite a entrada algumas vezes e observe o console para ter uma ideia de como os Effects são limpos.

Digite alguma coisa na entrada e, em seguida, pressione imediatamente "Desmontar o componente". Observe como a desmontagem limpa o Effect da última renderização. Aqui, ele limpa o último tempo limite antes que ele tenha a chance de disparar.

Finalmente, edite o componente acima e comente a função de limpeza para que os tempos limites não sejam cancelados. Tente digitar `abcde` rápido. O que você espera que aconteça em três segundos? O `console.log(text)` dentro do tempo limite imprimirá o *último* `text` e produzirá cinco logs `abcde`? Experimente para verificar sua intuição!

Três segundos depois, você deve ver uma sequência de logs (`a`, `ab`, `abc`, `abcd` e `abcde`) em vez de cinco logs `abcde`. **Cada Effect "captura" o valor `text` de sua renderização correspondente.** Não importa que o estado `text` tenha mudado: um Effect da renderização com `text = 'ab'` sempre verá `'ab'`. Em outras palavras, os Effects de cada renderização são isolados uns dos outros. Se você estiver curioso para saber como isso funciona, pode ler sobre [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures).

<DeepDive>

#### Cada renderização possui seus próprios Effects {/*each-render-has-its-own-effects*/}

Você pode pensar em `useEffect` como "anexar" um comportamento à saída de renderização. Considere este Effect:

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Bem-vindo(a) a {roomId}!</h1>;
}
```

Vamos ver o que exatamente acontece à medida que o usuário navega pelo app.

#### Renderização inicial {/*initial-render*/}

O usuário visita `<ChatRoom roomId="general" />`. Vamos [substituir mentalmente](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` por `'general'`:

```js
  // JSX para a primeira renderização (roomId = "general")
  return <h1>Bem-vindo(a) a general!</h1>;
```

**O Effect é *também* uma parte da saída de renderização.** O Effect da primeira renderização se torna:

```js
  // Effect para a primeira renderização (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para a primeira renderização (roomId = "general")
  ['general']
```

O React executa este Effect, que se conecta à sala de bate-papo `'general'`.

#### Re-renderização com as mesmas dependências {/*re-render-with-same-dependencies*/}

Digamos que `<ChatRoom roomId="general" />` re-renderize. A saída JSX é a mesma:

```js
  // JSX para a segunda renderização (roomId = "general")
  return <h1>Bem-vindo(a) a general!</h1>;
```

O React vê que a saída de renderização não mudou, então não atualiza o DOM.

O Effect da segunda renderização se parece com isto:

```js
  // Effect para a segunda renderização (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para a segunda renderização (roomId = "general")
  ['general']
```

O React compara `['general']` da segunda renderização com `['general']` da primeira renderização. **Como todas as dependências são as mesmas, o React *ignora* o Effect da segunda renderização.** Ele nunca é chamado.

#### Re-renderização com dependências diferentes {/*re-render-with-different-dependencies*/}

Então, o usuário visita `<ChatRoom roomId="travel" />`. Desta vez, o componente retorna um JSX diferente:

```js
  // JSX para a terceira renderização (roomId = "travel")
  return <h1>Bem-vindo(a) a travel!</h1>;
```

O React atualiza o DOM para mudar `"Bem-vindo(a) a general"` para `"Bem-vindo(a) a travel"`.

O Effect da terceira renderização se parece com isto:

```js
  // Effect para a terceira renderização (roomId = "travel")
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para a terceira renderização (roomId = "travel")
  ['travel']
```

O React compara `['travel']` da terceira renderização com `['general']` da segunda renderização. Uma dependência é diferente: `Object.is('travel', 'general')` é `false`. O Effect não pode ser ignorado.

**Antes que o React possa aplicar o Effect da terceira renderização, ele precisa limpar o último Effect que _foi_ executado.** O Effect da segunda renderização foi ignorado, então o React precisa limpar o Effect da primeira renderização. Se você rolar para cima até a primeira renderização, verá que sua limpeza chama `disconnect()` na conexão que foi criada com `createConnection('general')`. Isso desconecta o aplicativo da sala de bate-papo `'general'`.

Depois disso, o React executa o Effect da terceira renderização. Ele se conecta à sala de bate-papo `'travel'`.

#### Desmontar {/*unmount*/}

Finalmente, digamos que o usuário navegue para longe e o componente `ChatRoom` seja desmontado. O React executa a função de limpeza do último Effect. O último Effect foi da terceira renderização. A limpeza da terceira renderização destrói a conexão `createConnection('travel')`. Então, o aplicativo desconecta da sala `'travel'`.

#### Comportamentos apenas para desenvolvimento {/*development-only-behaviors*/}

Quando o [Strict Mode](/reference/react/StrictMode) está ativado, o React remonta cada componente uma vez após a montagem (estado e DOM são preservados). Isso [ajuda você a encontrar Effects que precisam de limpeza](#step-3-add-cleanup-if-needed) e expõe bugs como condições de corrida no início. Além disso, o React remontará os Effects sempre que você salvar um arquivo em desenvolvimento. Ambos os comportamentos são apenas para desenvolvimento.

</DeepDive>

<Recap>

- Ao contrário dos eventos, os Effects são causados pela própria renderização, em vez de uma interação específica.
- Os Effects permitem sincronizar um componente com algum sistema externo (API de terceiros, rede, etc.).
- Por padrão, os Effects são executados após cada renderização (incluindo a inicial).
- O React ignorará o Effect se todas as suas dependências tiverem os mesmos valores da última renderização.
- Você não pode "escolher" suas dependências. Elas são determinadas pelo código dentro do Effect.
- Uma array de dependências vazia (`[]`) corresponde à "montagem" do componente, ou seja, à sua adição à tela.
- No Strict Mode, o React monta componentes duas vezes (somente em desenvolvimento!) para testar seus Effects.
- Se seu Effect quebrar por causa da remontagem, você precisará implementar uma função de limpeza.
- O React chamará sua função de limpeza antes que o Effect seja executado na próxima vez e durante a desmontagem.

</Recap>

<Challenges>

#### Focar um campo na montagem {/*focus-a-field-on-mount*/}

Neste exemplo, o formulário renderiza um componente `<MyInput />`.

Use o método [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) da entrada para fazer com que `MyInput` foque automaticamente quando aparecer na tela. Já existe uma implementação comentada, mas ela não funciona direito. Descubra por que ela não funciona e corrija-a. (Se você estiver familiarizado com o atributo `autoFocus`, finja que ele não existe: estamos reimplementando a mesma funcionalidade do zero.)

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO: Isso não funciona direito. Corrija-o.
  // ref.current.focus()    

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Esconder' : 'Mostrar'} formulário</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Insira seu nome:
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            Torná-lo em maiúsculas
          </label>
          <p>Olá, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
``````css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

Para verificar se sua solução funciona, pressione "Mostrar formulário" e verifique se a entrada recebe o foco (torna-se destacada e o cursor é colocado dentro). Pressione "Ocultar formulário" e "Mostrar formulário" novamente. Verifique se a entrada está destacada novamente.

`MyInput` deve focar apenas _no mount_ em vez de após cada renderização. Para verificar se o comportamento está correto, pressione "Mostrar formulário" e, em seguida, pressione repetidamente a caixa de seleção "Deixar em maiúsculas". Clicar na caixa de seleção _não_ deve focar a entrada acima dela.

<Solution>

Chamar`ref.current.focus()` durante a renderização está errado porque é um *efeito colateral*. Efeitos colaterais devem ser colocados dentro de um manipulador de eventos (event handler) ou ser declarado com `useEffect`. Nesse caso, o efeito colateral é _causado_ pelo aparecimento do componente, em vez de por qualquer interação específica, então faz sentido colocá-lo em um Effect.

Para corrigir o erro, encapsule a chamada `ref.current.focus()` em uma declaração de Effect. Em seguida, para garantir que este Effect seja executado somente no mount em vez de após cada renderização, adicione as dependências `[]` vazias a ele.

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Ocultar' : 'Mostrar'} formulário</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Digite seu nome:
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            Deixar em maiúsculas
          </label>
          <p>Olá, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### Focar um campo condicionalmente {/*focus-a-field-conditionally*/}

Este formulário renderiza dois componentes `MyInput`.

Pressione "Mostrar formulário" e observe que o segundo campo recebe foco automaticamente. Isso ocorre porque ambos os componentes `<MyInput />` tentam focar o campo interno. Quando você chama `focus()` para dois campos de entrada em sequência, o último sempre "vence".

Digamos que você deseja focar o primeiro campo. O primeiro componente `MyInput` agora recebe uma prop `shouldFocus` booleana definida como `true`. Altere a lógica para que `focus()` seja chamado somente se a prop `shouldFocus` recebida por `MyInput` for `true`.

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  // TODO: call focus() only if shouldFocus is true.
  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Ocultar' : 'Mostrar'} formulário</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Digite seu primeiro nome:
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            Digite seu sobrenome:
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>Olá, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

Para verificar sua solução, pressione "Mostrar formulário" e "Ocultar formulário" repetidamente. Quando o formulário aparecer, apenas a *primeira* entrada deverá ser focada. Isso ocorre porque o componente pai renderiza a primeira entrada com `shouldFocus={true}` e a segunda entrada com `shouldFocus={false}`. Verifique também se ambas as entradas ainda funcionam e você pode digitar nas duas.

<Hint>

Você não pode declarar um Effect condicionalmente, mas seu Effect pode incluir lógica condicional.

</Hint>

<Solution>

Coloque a lógica condicional dentro do Effect. Você precisará especificar `shouldFocus` como uma dependência porque está usando dentro do Effect. (Isso significa que, se o `shouldFocus` de alguma entrada mudar de `false` para `true`, ele focará após a montagem.)

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    if (shouldFocus) {
      ref.current.focus();
    }
  }, [shouldFocus]);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Ocultar' : 'Mostrar'} formulário</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Digite seu primeiro nome:
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            Digite seu sobrenome:
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>Olá, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### Corrigir um intervalo que dispara duas vezes {/*fix-an-interval-that-fires-twice*/}

Este componente `Counter` exibe um contador que deve ser incrementado a cada segundo. No mount, ele chama [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) Isso faz com que `onTick` seja executado a cada segundo. A função `onTick` incrementa o contador.

No entanto, em vez de incrementar uma vez por segundo, ele incrementa duas vezes. Por que isso acontece? Encontre a causa do bug e corrija-o.

<Hint>

Tenha em mente que `setInterval` retorna um ID de intervalo, que você pode passar para [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) para interromper o intervalo.

</Hint>

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    setInterval(onTick, 1000);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function Form() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Ocultar' : 'Mostrar'} contador</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

<Solution>

Quando o [Strict Mode](/reference/react/StrictMode) está ativado (como nos sandboxes neste site), o React remonta cada componente uma vez no desenvolvimento. Isso faz com que o intervalo seja configurado duas vezes, e é por isso que a cada segundo o contador incrementa duas vezes.

No entanto, o comportamento do React não é a *causa* do bug: o bug já existe no código. O comportamento do React torna o bug mais perceptível. A verdadeira causa é que esse Effect inicia um processo, mas não oferece uma maneira de limpá-lo.

Para corrigir este código, salve o ID do intervalo retornado por `setInterval` e implemente uma função de limpeza com [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval):

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    const intervalId = setInterval(onTick, 1000);
    return () => clearInterval(intervalId);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Ocultar' : 'Mostrar'} contador</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

No desenvolvimento, o React ainda remontará seu componente uma vez para verificar se você implementou a limpeza bem. Portanto, haverá uma chamada `setInterval`, imediatamente seguida por` clearInterval`, e` setInterval` novamente. Na produção, haverá apenas uma chamada `setInterval`. O comportamento visível para o usuário em ambos os casos é o mesmo: o contador incrementa uma vez por segundo.

</Solution>

#### Corrigir a busca dentro de um Effect {/*fix-fetching-inside-an-effect*/}

Este componente mostra a biografia da pessoa selecionada. Ele carrega a biografia chamando uma função assíncrona `fetchBio(person)` no mount e sempre que `person` muda. Essa função assíncrona retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que eventualmente se resolve em uma string. Quando a busca é concluída, ele chama `setBio` para exibir essa string abaixo da caixa de seleção.

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    setBio(null);
    fetchBio(person).then(result => {
      setBio(result);
    });
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

Há um erro neste código. Comece selecionando "Alice". Em seguida, selecione "Bob" e, imediatamente após, selecione "Taylor". Se você fizer isso rápido o suficiente, notará aquele bug: Taylor está selecionado, mas o parágrafo abaixo diz "This is Bob's bio."

Por que isso acontece? Corrija o bug dentro deste Effect.

<Hint>

Se um Effect busca algo assincronamente, geralmente precisa de limpeza.

</Hint>

<Solution>

Para acionar o bug, as coisas precisam acontecer nesta ordem:

- Selecionar `'Bob'` aciona `fetchBio ('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio ('Taylor')`
- **Buscar `'Taylor'` é concluída *antes* de buscar `'Bob'`**
- O Effect do render de `'Taylor'` chama `setBio('This is Taylor’s bio')`
- Buscar `'Bob'` é concluída
- O Effect do render de `'Bob'` chama `setBio('This is Bob’s bio')`

É por isso que você vê a biografia de Bob, embora Taylor esteja selecionado. Bugs como este são chamados de [condições de corrida](https://pt.wikipedia.org/wiki/Condi%C3%A7%C3%A3o_de_corrida) porque duas operações assíncronas estão "competindo" entre si e podem chegar em uma ordem inesperada.

Para corrigir esta condição de corrida, adicione uma função de limpeza:

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

O Effect de cada renderização tem sua própria variável `ignore`. Inicialmente, a variável `ignore` é definida como `false`. No entanto, se um Effect for limpo (como ao selecionar uma pessoa diferente), sua variável `ignore` se torna `true`. Portanto, agora não importa em que ordem as solicitações são concluídas. Somente o Effect da última pessoa terá `ignore` definido como `false`, então ele chamará `setBio(result)`. Os Effects anteriores foram limpos, então a verificação `if (!ignore)` o impedirá de chamar `setBio`:

- Selecionar `'Bob'` aciona `fetchBio ('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio ('Taylor')` **e limpa o Effect anterior (de Bob)**
- Buscar `'Taylor'` é concluída *antes* de buscar `'Bob'`
- O Effect do render de `'Taylor'` chama `setBio('This is Taylor’s bio')`
- Buscar `'Bob'` é concluída
- O Effect do render de `'Bob'` **não faz nada porque sua flag `ignore` foi definida como `true`**

Além de ignorar o resultado de uma chamada de API desatualizada, você também pode usar [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) para cancelar as solicitações que não são mais necessárias. No entanto, por si só, isso não é suficiente para se proteger contra condições de corrida. Mais etapas assíncronas podem ser encadeadas após a busca, portanto, usar uma flag explícita como `ignore` é a maneira mais confiável de corrigir esse tipo de problema.

</Solution>

</Challenges>
