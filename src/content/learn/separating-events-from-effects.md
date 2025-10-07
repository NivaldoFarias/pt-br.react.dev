title: 'Separando Eventos de Efeitos'

<Intro>

Manipuladores de eventos são executados apenas quando você realiza a mesma interação novamente. Diferente dos manipuladores de eventos, os Efeitos resincronizam se algum valor que eles leem, como uma prop ou uma variável de estado, for diferente do que era durante a última renderização. Às vezes, você também quer uma mistura de ambos os comportamentos: um Efeito que é reexecutado em resposta a alguns valores, mas não a outros. Esta página ensinará como fazer isso.

</Intro>

<YouWillLearn>

- Como escolher entre um manipulador de eventos e um Efeito
- Por que os Efeitos são reativos e os manipuladores de eventos não são
- O que fazer quando você quer que uma parte do código do seu Efeito não seja reativa
- O que são Eventos de Efeito e como extraí-los dos seus Efeitos
- Como ler as últimas props e estado dos Efeitos usando Eventos de Efeito

</YouWillLearn>

## Escolhendo entre manipuladores de eventos e Efeitos {/*choosing-between-event-handlers-and-effects*/}

Primeiro, vamos recapitular a diferença entre manipuladores de eventos e Efeitos.

Imagine que você está implementando um componente de sala de chat. Seus requisitos são os seguintes:

1. Seu componente deve se conectar automaticamente à sala de chat selecionada.
1. Quando você clicar no botão "Enviar", ele deve enviar uma mensagem para o chat.

Vamos supor que você já implementou o código para eles, mas não tem certeza de onde colocá-lo. Você deve usar manipuladores de eventos ou Efeitos? Toda vez que você precisar responder a essa pergunta, considere [*por que* o código precisa ser executado.](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

### Manipuladores de eventos são executados em resposta a interações específicas {/*event-handlers-run-in-response-to-specific-interactions*/}

Do ponto de vista do usuário, enviar uma mensagem deve acontecer *porque* o botão "Enviar" específico foi clicado. O usuário ficará bastante chateado se você enviar a mensagem em qualquer outro momento ou por qualquer outro motivo. É por isso que enviar uma mensagem deve ser um manipulador de eventos. Manipuladores de eventos permitem que você lide com interações específicas:

```js {4-6}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');
  // ...
  function handleSendClick() {
    sendMessage(message);
  }
  // ...
  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

Com um manipulador de eventos, você pode ter certeza de que `sendMessage(message)` só será executado se o usuário pressionar o botão.

### Efeitos são executados sempre que a sincronização é necessária {/*effects-run-whenever-synchronization-is-needed*/}

Lembre-se de que você também precisa manter o componente conectado à sala de chat. Onde esse código vai?

O *motivo* para executar este código não é alguma interação específica. Não importa por que ou como o usuário navegou para a tela da sala de chat. Agora que eles estão olhando para ela e podem interagir com ela, o componente precisa permanecer conectado ao servidor de chat selecionado. Mesmo que o componente da sala de chat fosse a tela inicial do seu aplicativo, e o usuário não tivesse realizado nenhuma interação, você ainda precisaria se conectar. É por isso que é um Efeito:

```js {3-9}
function ChatRoom({ roomId }) {
  // ...
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Com este código, você pode ter certeza de que sempre há uma conexão ativa com o servidor de chat atualmente selecionado, *independentemente* das interações específicas realizadas pelo usuário. Se o usuário apenas abriu seu aplicativo, selecionou uma sala diferente ou navegou para outra tela e voltou, seu Efeito garantirá que o componente permanecerá *sincronizado* com a sala atualmente selecionada e se [reconectará sempre que for necessário.](/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once)

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
export function sendMessage(message) {
  console.log('🔵 You sent: ' + message);
}

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input, select { margin-right: 20px; }
```

</Sandpack>

## Valores reativos e lógica reativa {/*reactive-values-and-reactive-logic*/}

Intuitivamente, você pode dizer que os manipuladores de eventos são sempre acionados "manualmente", por exemplo, clicando em um botão. Efeitos, por outro lado, são "automáticos": eles são executados e reexecutados quantas vezes for necessário para permanecerem sincronizados.

Existe uma maneira mais precisa de pensar sobre isso.

Props, estado e variáveis declaradas dentro do corpo do seu componente são chamados de <CodeStep step={2}>valores reativos</CodeStep>. Neste exemplo, `serverUrl` não é um valor reativo, mas `roomId` e `message` são. Eles participam do fluxo de dados de renderização:

```js [[2, 3, "roomId"], [2, 4, "message"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

Valores reativos como esses podem mudar devido a uma re-renderização. Por exemplo, o usuário pode editar a `message` ou escolher um `roomId` diferente em um menu suspenso. Manipuladores de eventos e Efeitos respondem às mudanças de maneiras diferentes:

- **A lógica dentro dos manipuladores de eventos *não* é reativa.** Ela não será executada novamente, a menos que o usuário realize a mesma interação (por exemplo, um clique) novamente. Manipuladores de eventos podem ler valores reativos sem "reagir" às suas mudanças.
- **A lógica dentro dos Efeitos é *reativa.*** Se o seu Efeito ler um valor reativo, [você terá que especificá-lo como uma dependência.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Então, se uma re-renderização fizer com que esse valor mude, o React reexecutará a lógica do seu Efeito com o novo valor.

Vamos revisitar o exemplo anterior para ilustrar essa diferença.

### A lógica dentro dos manipuladores de eventos não é reativa {/*logic-inside-event-handlers-is-not-reactive*/}

Dê uma olhada nesta linha de código. Essa lógica deveria ser reativa ou não?

```js [[2, 2, "message"]]
    // ...
    sendMessage(message);
    // ...
```

Do ponto de vista do usuário, **uma mudança na `message` *não* significa que eles querem enviar uma mensagem.** Significa apenas que o usuário está digitando. Em outras palavras, a lógica que envia uma mensagem não deve ser reativa. Ela não deve ser executada novamente apenas porque o <CodeStep step={2}>valor reativo</CodeStep> mudou. É por isso que ela pertence ao manipulador de eventos:

```js {2}
  function handleSendClick() {
    sendMessage(message);
  }
```

Manipuladores de eventos não são reativos, então `sendMessage(message)` só será executado quando o usuário clicar no botão Enviar.

### A lógica dentro dos Efeitos é reativa {/*logic-inside-effects-is-reactive*/}

Agora vamos voltar a estas linhas:

```js [[2, 2, "roomId"]]
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

Do ponto de vista do usuário, **uma mudança no `roomId` *significa* que eles querem se conectar a uma sala diferente.** Em outras palavras, a lógica para se conectar à sala deve ser reativa. Você *quer* que essas linhas de código "acompanhem" o <CodeStep step={2}>valor reativo</CodeStep> e sejam executadas novamente se esse valor for diferente. É por isso que ela pertence a um Efeito:

```js {2-3}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Efeitos são reativos, então `createConnection(serverUrl, roomId)` e `connection.connect()` serão executados para cada valor distinto de `roomId`. Seu Efeito mantém a conexão do chat sincronizada com a sala atualmente selecionada.

## Extraindo lógica não reativa de Efeitos {/*extracting-non-reactive-logic-out-of-effects*/}

As coisas ficam mais complicadas quando você quer misturar lógica reativa com lógica não reativa.

Por exemplo, imagine que você quer mostrar uma notificação quando o usuário se conecta ao chat. Você lê o tema atual (escuro ou claro) das props para poder mostrar a notificação na cor correta:

```js {1,4-6}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

No entanto, `theme` é um valor reativo (ele pode mudar como resultado de uma re-renderização), e [todo valor reativo lido por um Efeito deve ser declarado como sua dependência.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Agora você tem que especificar `theme` como uma dependência do seu Efeito:

```js {5,11}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ Todas as dependências declaradas
  // ...
```

Brinque com este exemplo e veja se consegue identificar o problema com a experiência do usuário:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
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
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

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
label { display: block; margin-top: 10px; }
```

</Sandpack>

Quando o `roomId` muda, o chat se reconecta como você esperaria. Mas como `theme` também é uma dependência, o chat *também* se reconecta toda vez que você alterna entre o tema escuro e o claro. Isso não é bom!

Em outras palavras, você *não* quer que esta linha seja reativa, mesmo que esteja dentro de um Efeito (que é reativo):

```js
      // ...
      showNotification('Connected!', theme);
      // ...
```

Você precisa de uma maneira de separar essa lógica não reativa da lógica reativa do Efeito ao redor dela.

### Declarando um Evento de Efeito {/*declaring-an-effect-event*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Use um Hook especial chamado [`useEffectEvent`](/reference/react/experimental_useEffectEvent) para extrair essa lógica não reativa do seu Efeito:

```js {1,4-6}
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

Aqui, `onConnected` é chamado de *Evento de Efeito*. É uma parte da lógica do seu Efeito, mas se comporta muito mais como um manipulador de eventos. A lógica dentro dele não é reativa e ele sempre "vê" os últimos valores de suas props e estado.

Agora você pode chamar o Evento de Efeito `onConnected` de dentro do seu Efeito:

```js {2-4,9,13}
function ChatRoom({ roomId, theme