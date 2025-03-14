```

```js
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ All dependencies declared

  // ...
```

Now, each `useEffect` call is focused on synchronizing a single thing. This makes the code easier to understand and maintain.

### Can you read a value inside the Effect without "reacting" to it? {/*can-you-read-a-value-inside-the-effect-without-reacting-to-it*/}

Sometimes, you might want to read the current value of a reactive variable *without* reacting to it. For example, you might want to log some information every time an Effect runs.

Imagine that you are synchronizing a chat room. When the user opens the chat, you want to log the current `theme` to the console:

```js {6,8}
function ChatRoom({ roomId }) {
  const theme = useContext(ThemeContext);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    console.log('Current theme:', theme); // Log the theme
    return () => connection.disconnect();
  }, [roomId, theme]); // ✅ All dependencies declared
  // ...
}
```

This works, but it is inefficient. Every time that the `theme` changes, the Effect re-runs, and a new connection is re-established. However, you only wanted to log the theme *once*, when the chat room is opened. You don't need the chat to re-connect when the theme changes.

**To read a value from the Effect without "reacting" to it, you have a few options:**

*   **If the value is only used inside an event handler**, read it in the handler instead.
*   **If you want to log some information**, move `console.log` inside the Effect's body, but *outside* of the dependencies:

```js {6}
function ChatRoom({ roomId }) {
  const theme = useContext(ThemeContext);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared

  console.log('Current theme:', theme); // Log the theme
  // ...
}
```

Now, the Effect will only re-run when `roomId` changes, but the `console.log` for `theme` will still execute during every render. If you need to display the value in the UI, it will be updated automatically with no extra work. If you only need to look at it in the console, this is enough!

### Does your dependency change too often (objects and functions) {/*does-your-dependency-change-too-often-objects-and-functions*/}

If your dependency is an object or a function, it might cause an Effect to run more often than you expect.

Imagine you are writing a chat application. When the user sends a message, you want to automatically scroll to the bottom of the chat. You can use a `ref` to a `scrollable` element, and then call `scrollToBottom()` inside an Effect:

```js
function ChatRoom({ messages }) {
  const scrollRef = useRef(null);

  useEffect(() => {
    scrollToBottom(scrollRef.current);
  }, [scrollRef.current]); // 🚩 scrollRef.current is an object
  // ...
}
```

The dependency list is now `[scrollRef.current]`. In the original code, this causes a bug. When you create a new `ref` with `useRef()`, React will set the `current` property to an object: `{ current: null }`. **React only updates the `current` property of the ref *during render*.** When the component re-renders, `scrollRef.current` will be a *different* object.

This causes the Effect to re-run on every render, endlessly. To remove this unnecessary dependency, you can make use of a ref to store a value *between* renders.

In this example, the `scrollRef` value never changes--so you shouldn't include `scrollRef.current` as a dependency. Instead, the Effect should run whenever `messages` change:

```js
function ChatRoom({ messages }) {
  const scrollRef = useRef(null);

  useEffect(() => {
    scrollToBottom(scrollRef.current);
  }, [messages]); // ✅ All dependencies declared
  // ...
}
```

Now, the Effect will only run when the `messages` prop changes. It reads the `scrollRef` value from the DOM. The scroll position will now automatically update when the new messages are added to the chat history.

Objects and functions are *not* equal by reference. If you create a new object or function during every render, it will be different every time. This can cause an Effect to re-run more often than necessary. Avoiding these kinds of "unstable" dependencies is crucial for performance. There are a few common patterns to avoid this:

*   **If you only need to call a function during the initial render,** move the function definition *outside* of your component. This makes it stable.
*   **If you need to pass a function as a prop to a child,** [memoize the function.](/reference/react/useCallback)
*   **If the value is used to calculate something complex,** [memoize the value.](/reference/react/useMemo)

<Recap>

When resolving Effect dependencies, remember this workflow:

1.  Start by declaring all dependencies.
2.  If the Effect runs too often, identify *why.*
3.  Make changes to your component code to *change* the dependencies.
4.  Move logic to event handlers or to different Effects if you can.
5.  If you need to read the value of a dependency *without* "reacting" to it, move the code around to do so, but don't remove the dependency.
6.  Avoid creating objects or functions inside your component, because they count as unstable dependencies.

</Recap>
```
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ Todas as dependências declaradas

  // ...
```

Agora, o primeiro Effect só é executado novamente se o `country` mudar, enquanto o segundo Effect é executado novamente quando a `city` mudar. Você os separou por propósito: duas coisas diferentes são sincronizadas por dois Effects separados. Dois Effects separados possuem duas listas de dependências separadas, portanto, eles não irão disparar uns aos outros involuntariamente.

O código final é maior que o original, mas dividir esses Effects ainda está correto. [Cada Effect deve representar um processo de sincronização independente.](/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process) Neste exemplo, a remoção de um Effect não quebra a lógica do outro Effect. Isso significa que eles *sincronizam coisas diferentes,* e é bom dividi-los. Se você está preocupado com duplicação, você pode melhorar este código [extraindo a lógica repetitiva em um Hook personalizado.](/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks)

### Você está lendo algum state para calcular o próximo state? {/*are-you-reading-some-state-to-calculate-the-next-state*/}

Este Effect atualiza a variável de state `messages` com um array recém-criado toda vez que uma nova mensagem chega:

```js {2,6-8}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

Ele usa a variável `messages` para [criar um novo array](/learn/updating-arrays-in-state) começando com todas as mensagens existentes e adiciona a nova mensagem no final. No entanto, como `messages` é um valor reativo lido por um Effect, ele deve ser uma dependência:

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ Todas as dependências declaradas
  // ...
```

E tornar `messages` uma dependência introduz um problema.

Toda vez que você recebe uma mensagem, `setMessages()` faz com que o componente renderize novamente com um novo array `messages` que inclui a mensagem recebida. No entanto, como este Effect agora depende de `messages`, isso *também* irá re-sincronizar o Effect. Então, cada nova mensagem fará o chat se reconectar. O usuário não gostaria disso!

Para corrigir o problema, não leia `messages` dentro do Effect. Em vez disso, passe uma [função atualizadora](/reference/react/useState#updating-state-based-on-the-previous-state) para `setMessages`:

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

**Observe como seu Effect não lê a variável `messages` em absoluto agora.** Você só precisa passar uma função atualizadora como `msgs => [...msgs, receivedMessage]`. React [coloca sua função atualizadora em uma fila](/learn/queueing-a-series-of-state-updates) e fornecerá o argumento `msgs` a ela durante o próximo render. É por isso que o próprio Effect não precisa depender mais de `messages`. Como resultado dessa correção, receber uma mensagem de chat não fará mais o chat se reconectar.

### Você quer ler um valor sem "reagir" às suas mudanças? {/*do-you-want-to-read-a-value-without-reacting-to-its-changes*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Suponha que você queira tocar um som quando o usuário receber uma nova mensagem, a menos que `isMuted` seja `true`:

```js {3,10-12}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    // ...
```

Como seu Effect agora usa `isMuted` em seu código, você deve adicioná-lo às dependências:

```js {10,15}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ Todas as dependências declaradas
  // ...
```

O problema é que toda vez que `isMuted` muda (por exemplo, quando o usuário pressiona a alternância "Muted"), o Effect irá re-sincronizar e reconectar ao chat. Essa não é a experiência do usuário desejada! (Neste exemplo, mesmo desabilitar o linter não funcionaria -- se você fizer isso, `isMuted` ficaria "preso" com seu valor antigo.)

Para resolver este problema, você precisa extrair a lógica que não deve ser reativa do Effect. Você não quer que este Effect "reaja" às mudanças em `isMuted`. [Mova este pedaço de lógica não reativa para um Effect Event:](/learn/separating-events-from-effects#declaring-an-effect-event)

```js {1,7-12,18,21}
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent(receivedMessage => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Effect Events permitem que você divida um Effect em partes reativas (que devem "reagir" aos valores reativos, como `roomId` e suas mudanças) e partes não reativas (que só leem seus valores mais recentes, como `onMessage` lê `isMuted`). **Agora que você lê `isMuted` dentro de um Effect Event, ele não precisa ser uma dependência do seu Effect.** Como resultado, o chat não se reconectará quando você ativar e desativar a configuração "Muted", resolvendo o problema original!

#### Encapsulando um manipulador de eventos das props {/*wrapping-an-event-handler-from-the-props*/}

Você pode se deparar com um problema semelhante quando seu componente recebe um manipulador de eventos como uma prop:

```js {1,8,11}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ Todas as dependências declaradas
  // ...
```

Suponha que o componente pai passe uma função `onReceiveMessage` *diferente* em cada renderização:

```js {3-5}
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>
```

Como `onReceiveMessage` é uma dependência, faria o Effect re-sincronizar após cada re-renderização do pai. Isso faria ele se reconectar ao chat. Para resolver isso, encapsule a chamada em um Effect Event:

```js {4-6,12,15}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Effect Events não são reativos, então você não precisa especificá-los como dependências. Como resultado, o chat não se reconectará mesmo se o componente pai passar uma função que seja diferente em cada re-renderização.

#### Separando código reativo e não reativo {/*separating-reactive-and-non-reactive-code*/}

Neste exemplo, você quer registrar uma visita toda vez que `roomId` mudar. Você quer incluir o `notificationCount` atual com cada registro, mas você *não* quer que uma mudança em `notificationCount` acione um evento de registro.

A solução é novamente dividir o código não reativo em um Effect Event:

```js {2-4,7}
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent(visitedRoomId => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
}
```

Você quer que sua lógica seja reativa em relação a `roomId`, então você lê `roomId` dentro do seu Effect. No entanto, você não quer que uma mudança em `notificationCount` registre uma visita extra, então você lê `notificationCount` dentro do Effect Event. [Saiba mais sobre como ler as últimas props e state dos Effects usando Effect Events.](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)

### Algum valor reativo muda involuntariamente? {/*does-some-reactive-value-change-unintentionally*/}

Às vezes, você *quer* que seu Effect "reaja" a um determinado valor, mas esse valor muda com mais frequência do que você gostaria -- e pode não refletir nenhuma mudança real da perspectiva do usuário. Por exemplo, digamos que você crie um objeto `options` no corpo do seu componente e, então, leia esse objeto de dentro do seu Effect:

```js {3-6,9}
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

Este objeto é declarado no corpo do componente, então ele é um [valor reativo.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Quando você lê um valor reativo como este dentro de um Effect, você o declara como uma dependência. Isso garante que seu Effect "reaja" às suas mudanças:

```js {3,6}
  // ...
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ Todas as dependências declaradas
  // ...
```

É importante declará-lo como uma dependência! Isso garante, por exemplo, que, se o `roomId` mudar, seu Effect irá se reconectar ao chat com as novas `options`. No entanto, também há um problema com o código acima. Para vê-lo, tente digitar na entrada no sandbox abaixo e observe o que acontece no console:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Desabilite temporariamente o linter para demonstrar o problema
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Bem-vindo ao chat da sala {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando na sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

No sandbox acima, a entrada atualiza apenas a variável de state `message`. Da perspectiva do usuário, isso não deve afetar a conexão do chat. No entanto, toda vez que você atualizar a `message`, seu componente renderiza novamente. Quando seu componente renderiza novamente, o código dentro dele é executado novamente do zero.

Um novo objeto `options` é criado do zero em cada re-renderização do componente `ChatRoom`. React vê que o objeto `options` é um *objeto diferente* do objeto `options` criado durante a última renderização. É por isso que ele re-sincroniza seu Effect (que depende de `options`), e o chat se reconecta enquanto você digita.

**Este problema só afeta objetos e funções. Em JavaScript, cada objeto e função recém-criados são considerados distintos de todos os outros. Não importa que o conteúdo dentro deles possa ser o mesmo!**

```js {7-8}
// Durante a primeira renderização
const options1 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// Durante a próxima renderização
const options2 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// Estes são dois objetos diferentes!
console.log(Object.is(options1, options2)); // false
```

**Dependências de objetos e funções podem fazer seu Effect re-sincronizar mais frequentemente do que você precisa.**

É por isso que, sempre que possível, você deve tentar evitar objetos e funções como dependências do seu Effect. Em vez disso, tente movê-los para fora do componente, dentro do Effect ou extrair valores primitivos deles.

#### Mova objetos e funções estáticos para fora do seu componente {/*move-static-objects-and-functions-outside-your-component*/}

Se o objeto não depender de nenhuma prop e state, você pode mover esse objeto para fora do seu componente:

```js {1-4,13}
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Todas as dependências declaradas
  // ...
```

Dessa forma, você *prova* ao linter que ele não é reativo. Ele não pode mudar como resultado de uma re-renderização, então não precisa ser uma dependência. Agora, re-renderizar `ChatRoom` não fará com que seu Effect se re-sincronize.

Isso funciona para funções também:

```js {1-6,12}
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Todas as dependências declaradas
  // ...
```

Como `createOptions` é declarado fora do seu componente, ele não é um valor reativo. É por isso que ele não precisa ser especificado nas dependências do seu Effect e porque nunca fará com que seu Effect se re-sincronize.

#### Mova objetos e funções dinâmicos para dentro do seu Effect {/*move-dynamic-objects-and-functions-inside-your-effect*/}

Se seu objeto depende de algum valor reativo que pode mudar como resultado de uma re-renderização, como uma prop `roomId`, você não pode puxá-lo para *fora* do seu componente. Você pode, no entanto, mover sua criação *para dentro* do código do seu Effect:

```js {7-10,11,14}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Agora que `options` é declarado dentro do seu Effect, ele não é mais uma dependência do seu Effect. Em vez disso, o único valor reativo usado pelo seu Effect é `roomId`. Como `roomId` não é um objeto ou função, você pode ter certeza de que ele não será *involuntariamente* diferente. Em JavaScript, números e strings são comparados por seu conteúdo:

```js {7-8}
// Durante a primeira renderização
const roomId1 = 'music';

// Durante a próxima renderização
const roomId2 = 'music';

// Estas duas strings são as mesmas!
console.log(Object.is(roomId1, roomId2)); // true
```

Graças a esta correção, o chat não se reconecta mais se você editar a entrada:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Bem-vindo à sala {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando na sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```
```html
## Parte 2 de 2:

```

```html
## Parte 3 de 2:
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Entretanto, ele *reconecta* quando você muda o dropdown `roomId`, como o esperado.

Isto funciona para funções também:

```js {7-12,14}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Você pode escrever suas próprias funções para agrupar trechos de lógica dentro do seu Effect. Desde que você também as declare *dentro* do seu Effect, elas não são valores reativos e, por isso, não precisam ser dependências do seu Effect.

#### Leia valores primitivos de objetos {/*read-primitive-values-from-objects*/}

Às vezes, você pode receber um objeto das props:

```js {1,5,8}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ Todas as dependências declaradas
  // ...
```

O risco aqui é que o componente pai criará o objeto durante o renderização:

```js {3-6}
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

Isto faria com que seu Effect se reconectasse toda vez que o componente pai renderizasse. Para corrigir isso, leia as informações do objeto *fora* do Effect e evite ter objetos e funções como dependências:

```js {4,7-8,12}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Todas as dependências declaradas
  // ...
```

A lógica fica um pouco repetitiva (você lê alguns valores de um objeto fora de um Effect e depois cria um objeto com os mesmos valores dentro do Effect). Mas isto torna muito explícito sobre quais informações seu Effect *realmente* depende. Se um objeto for recriado sem intenção pelo componente pai, o chat não irá se reconectar. No entanto, se `options.roomId` ou `options.serverUrl` forem realmente diferentes, o chat irá se reconectar.

#### Calcule valores primitivos de funções {/*calculate-primitive-values-from-functions*/}

A mesma abordagem pode funcionar para funções. Por exemplo, suponha que o componente pai passe uma função:

```js {3-8}
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }}
/>
```

Para evitar torná-la uma dependência (e causar sua reconexão em re-renders), chame-a fora do Effect. Isto te dá os valores `roomId` e `serverUrl` que não são objetos e que você pode ler de dentro do seu Effect:

```js {1,4}
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Todas as dependências declaradas
  // ...
```

Isto só funciona para funções [puras](/learn/keeping-components-pure) porque elas são seguras para serem chamadas durante a renderização. Se sua função é um manipulador de eventos, mas você não quer que suas mudanças re-sincronizem seu Effect, [encapsule-a em um Effect Event](#do-you-want-to-read-a-value-without-reacting-to-its-changes) ao invés disso.

<Recap>

- As dependências devem sempre corresponder ao código.
- Quando você não está feliz com suas dependências, o que você precisa editar é o código.
- Suprimir o linter leva a erros muito confusos, e você sempre deve evitá-lo.
- Para remover uma dependência, você precisa "provar" para o linter que ela não é necessária.
- Se algum código deve rodar em resposta a uma interação específica, mova aquele código para um manipulador de evento.
- Se diferentes partes do seu Effect devem rodar novamente por motivos diferentes, divida-o em vários Effects.
- Se você quer atualizar algum estado baseado no estado anterior, passe uma função de atualização.
- Se você quer ler o valor mais recente sem "reagir" a ele, extraia um Effect Event do seu Effect.
- Em JavaScript, objetos e funções são considerados diferentes se foram criados em momentos diferentes.
- Tente evitar dependências de objetos e de funções. Mova-os para fora do componente ou para dentro do Effect.

</Recap>

<Challenges>

#### Consertar um intervalo que reinicia {/*fix-a-resetting-interval*/}

Este Effect configura um intervalo que tica a cada segundo. Você notou algo estranho acontecendo: parece que o intervalo é destruído e recriado toda vez que tica. Corrija o código para que o intervalo não seja constantemente recriado.

<Hint>

Parece que o código deste Effect depende de `count`. Existe alguma forma de não precisar desta dependência? Deveria haver uma forma de atualizar o estado `count` baseado no seu valor anterior sem adicionar uma dependência naquele valor.

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ Criando um interval');
    const id = setInterval(() => {
      console.log('⏰ Interval tick');
      setCount(count + 1);
    }, 1000);
    return () => {
      console.log('❌ Limpando um interval');
      clearInterval(id);
    };
  }, [count]);

  return <h1>Contador: {count}</h1>
}
```

</Sandpack>

<Solution>

Você quer atualizar o estado `count` para ser `count + 1` de dentro do Effect. Entretanto, isto faz com que seu Effect dependa de `count`, que muda a cada tick, e é por isso que seu intervalo é recriado a cada tick.

Para resolver isto, use a [função de atualização](/reference/react/useState#updating-state-based-on-the-previous-state) e escreva `setCount(c => c + 1)` em vez de `setCount(count + 1)`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ Criando um interval');
    const id = setInterval(() => {
      console.log('⏰ Interval tick');
      setCount(c => c + 1);
    }, 1000);
    return () => {
      console.log('❌ Limpando um interval');
      clearInterval(id);
    };
  }, []);

  return <h1>Contador: {count}</h1>
}
```

</Sandpack>

Em vez de ler `count` dentro do Effect, você passa uma instrução `c => c + 1` ("incrementar este número!") para o React. React irá aplicá-la no próximo render. E já que você não precisa ler o valor de `count` dentro do seu Effect, você pode manter as dependências do seu Effect vazias (`[]`). Isto previne que seu Effect recrie o intervalo a cada tick.

</Solution>

#### Consertar uma animação que re-dispara {/*fix-a-retriggering-animation*/}

Neste exemplo, quando você aperta "Mostrar", uma mensagem de boas-vindas desaparece. A animação leva um segundo. Quando você aperta "Remover", a mensagem de boas-vindas desaparece imediatamente. A lógica para a animação de desaparecimento é implementada no arquivo `animation.js` como um simples [loop de animação](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) em JavaScript. Você não precisa mudar essa lógica. Você pode tratá-la como uma biblioteca de terceiros. Seu Effect cria uma instância de `FadeInAnimation` para o nó do DOM e, depois, chama `start(duration)` ou `stop()` para controlar a animação. A `duration` é controlada por um deslizar. Ajuste o deslizador e veja como a animação muda.

Este código já funciona, mas tem algo que você quer mudar. Atualmente, quando você move o deslizador que controla a variável de estado `duration`, ele re-dispara a animação. Mude o comportamento para que o Effect não "reaja" à variável `duration`. Quando você apertar "Mostrar," o Effect deve usar o valor atual de `duration` no deslizador. Entretanto, mover o próprio deslizador não deveria, por si só, re-disparar a animação.

<Hint>

Existe uma linha de código dentro do Effect que não deveria ser reativa? Como você pode mover código não reativo para fora do Effect?

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
import { useState, useEffect, useRef } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome({ duration }) {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [duration]);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        Fade in duration: {duration} ms
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
    </>
  );
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // Jump to end immediately
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // Start animating
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

<Solution>

Seu Effect precisa ler o valor mais recente de `duration`, mas você não quer que ele "reaja" a mudanças em `duration`. Você usa `duration` para iniciar a animação, mas iniciar a animação não é reativo. Extraia a linha de código não reativa para um Effect Event, e chame aquela função do seu Effect.

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
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

function Welcome({ duration }) {
  const ref = useRef(null);

  const onAppear = useEffectEvent(animation => {
    animation.start(duration);
  });

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    onAppear(animation);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        Fade in duration: {duration} ms
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
    </>
  );
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    this.onProgress(0);
    this.startTime = performance.now();
    this.frameId = requestAnimationFrame(() => this.onFrame());
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

Eventos de Effect como `onAppear` não são reativos, então você pode ler `duration` dentro sem re-disparar a animação.

</Solution>

#### Consertar uma reconexão no chat {/*fix-a-reconnecting-chat*/}

Neste exemplo, toda vez que você aperta "Alternar tema", o chat se reconecta. Por que isto acontece? Corrija o erro para que o chat se reconecte apenas quando você editar a Server URL ou escolher uma sala de bate-papo diferente.

Trate `chat.js` como uma biblioteca externa de terceiros: você pode consultá-la para checar sua API, mas não a edite.

<Hint>

Existe mais de uma forma de consertar isto, mas, no fim das contas você quer evitar ter um objeto como sua dependência.

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return <h1>Bem-vindo à sala {options.roomId}!</h1>;
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // A real implementation would actually connect to the server
  if (typeof serverUrl !== 'string') {
    throw Error('Expected serverUrl to be a string. Received: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Expected roomId to be a string. Received: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

<Solution>

Seu Effect está rodando novamente porque ele depende do objeto `options`. Objetos podem ser recriados sem querer, você deveria tentar evitá-los como dependências dos seus Effects sempre que possível.

A correção menos invasiva é ler `roomId` e `serverUrl` bem fora do Effect, e depois fazer o Effect depender daqueles valores primitivos (que não podem mudar sem querer). Dentro do Effect, crie um objeto e passe-o para `createConnection`:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Bem-vindo à sala {options.roomId} room!</h1>;
}
``````js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando ao chat "' + roomId + '" no ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado do chat "' + roomId + '" no ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

Seria ainda melhor substituir a propriedade `options` do objeto pelas propriedades `roomId` e `serverUrl` mais específicas:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        serverUrl={serverUrl}
      />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando ao chat "' + roomId + '" no ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado do chat "' + roomId + '" no ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

Manter as props primitivas sempre que possível torna mais fácil otimizar seus componentes mais tarde.

</Solution>

#### Consertar um chat reconectando, novamente {/*fix-a-reconnecting-chat-again*/}

Este exemplo conecta ao chat com ou sem criptografia. Alterne a caixa de seleção e observe as mensagens diferentes no console quando a criptografia está ativada e desativada. Tente mudar a sala. Em seguida, tente alternar o tema. Quando você estiver conectado a uma sala de chat, receberá novas mensagens a cada poucos segundos. Verifique se a cor delas corresponde ao tema escolhido.

Neste exemplo, o chat se reconecta toda vez que você tenta alterar o tema. Corrija isso. Após a correção, alterar o tema não deve reconectar o chat, mas alternar as configurações de criptografia ou alterar a sala deve se reconectar.

Não altere nenhum código em `chat.js`. Fora isso, você pode alterar qualquer código, desde que resulte no mesmo comportamento. Por exemplo, você pode achar útil alterar quais props estão sendo passadas.

<Hint>

Você está passando duas funções: `onMessage` e `createConnection`. Ambas são criadas do zero toda vez que `App` renderiza novamente. Elas são consideradas novos valores toda vez, e é por isso que elas reativam seu Effect.

Uma dessas funções é um manipulador de eventos. Você conhece alguma forma de chamar um manipulador de eventos em um Effect sem "reagir" aos novos valores da função manipuladora de eventos? Isso seria útil!

Outra dessas funções só existe para passar algum `state` para um método API importado. Essa função é realmente necessária? Qual é a informação essencial que está sendo passada? Talvez você precise mover algumas importações de `App.js` para `ChatRoom.js`.

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

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';
import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
      </label>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
        createConnection={() => {
          const options = {
            serverUrl: 'https://localhost:1234',
            roomId: roomId
          };
          if (isEncrypted) {
            return createEncryptedConnection(options);
          } else {
            return createUnencryptedConnection(options);
          }
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function ChatRoom({ roomId, createConnection, onMessage }) {
  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [createConnection, onMessage]);

  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 🔐 Conectando ao chat "' + roomId + '"... (criptografado)');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 🔐 Desconectado do chat "' + roomId + '" (criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando ao chat "' + roomId + '" (não criptografado)...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado do chat "' + roomId + '" (não criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
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
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

<Solution>

Há mais de uma maneira correta de resolver isso, mas aqui está uma possível solução.

No exemplo original, alternar o tema causava a criação e passagem de funções `onMessage` e `createConnection` diferentes. Como o Effect dependia dessas funções, o chat se reconectava toda vez que você alternava o tema.

Para corrigir o problema com `onMessage`, você precisava encapsulá-lo em um Evento de Effect:

```js {1,2,6}
export default function ChatRoom({ roomId, createConnection, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    // ...
```

Diferente da prop `onMessage`, o Evento de Effect `onReceiveMessage` não é reativo. É por isso que ele não precisa ser uma dependência do seu Effect. Como resultado, alterações em `onMessage` não farão com que o chat se reconecte.

Você não pode fazer o mesmo com `createConnection` porque ele *deve* ser reativo. Você *quer* que o Effect seja reativado se o usuário alternar entre uma conexão criptografada e uma não criptografada, ou se o usuário mudar a sala atual. No entanto, como `createConnection` é uma função, você não pode verificar se as informações que ela lê foram *realmente* alteradas ou não. Para resolver isso, em vez de passar `createConnection` do componente `App`, passe os valores brutos `roomId` e `isEncrypted`:

```js {2-3}
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
      />
```

Agora você pode mover a função `createConnection` *dentro* do Effect, em vez de passá-la do `App`:

```js {1-4,6,10-20}
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }
    // ...
```

Após essas duas alterações, seu Effect não depende mais de nenhum valor de função:

```js {1,8,10,21}
export default function ChatRoom({ roomId, isEncrypted, onMessage }) { // Valores reativos
  const onReceiveMessage = useEffectEvent(onMessage); // Não reativo

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId // Lendo um valor reativo
      };
      if (isEncrypted) { // Lendo um valor reativo
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]); // ✅ Todas as dependências declaradas
```

Como resultado, o chat se reconecta somente quando algo significativo (`roomId` ou `isEncrypted`) muda:

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

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
      </label>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]);

  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 🔐 Conectando ao chat "' + roomId + '"... (criptografado)');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 🔐 Desconectado do chat "' + roomId + '" (criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava que roomId fosse uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando ao chat "' + roomId + '" (não criptografado)...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado do chat "' + roomId + '" (não criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
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
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

</Solution>

</Challenges>
```