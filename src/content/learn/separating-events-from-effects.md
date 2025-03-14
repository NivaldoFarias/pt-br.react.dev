---
title: 'Separando Eventos de Efeitos'
---

<Intro>

Manipuladores de eventos só são reexecutados quando você executa a mesma interação novamente. Diferente dos manipuladores de eventos, os Efeitos ressincronizam se algum valor que eles lêem, como uma prop ou uma variável de estado, é diferente do que era durante a última renderização. Às vezes, você também deseja uma mistura dos dois comportamentos: um Efeito que é reexecutado em resposta a alguns valores, mas não a outros. Esta página vai te ensinar como fazer isso.

</Intro>

<YouWillLearn>

- Como escolher entre um manipulador de eventos e um Efeito
- Por que Efeitos são reativos e manipuladores de eventos não
- O que fazer quando você quer que uma parte do código do seu Efeito não seja reativa
- O que são Eventos de Efeito e como extraí-los de seus Efeitos
- Como ler as últimas props e estado de Efeitos usando Eventos de Efeito

</YouWillLearn>

## Escolhendo entre manipuladores de eventos e Efeitos {/*choosing-between-event-handlers-and-effects*/}

Primeiro, vamos recapitular a diferença entre manipuladores de eventos e Efeitos.

Imagine que você está implementando um componente de sala de bate-papo. Seus requisitos são os seguintes:

1. Seu componente deve se conectar automaticamente à sala de bate-papo selecionada.
1. Quando você clica no botão "Enviar", ele deve enviar uma mensagem para o bate-papo.

Digamos que você já implementou o código para eles, mas não tem certeza de onde colocá-lo. Você deve usar manipuladores de eventos ou Efeitos? Toda vez que precisar responder a essa pergunta, considere [*por que* o código precisa ser executado.](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

### Manipuladores de eventos são executados em resposta a interações específicas {/*event-handlers-run-in-response-to-specific-interactions*/}

Da perspectiva do usuário, enviar uma mensagem deve acontecer *porque* o botão "Enviar" específico foi clicado. O usuário ficará bastante chateado se você enviar sua mensagem em qualquer outro momento ou por qualquer outro motivo. É por isso que enviar uma mensagem deve ser um manipulador de eventos. Manipuladores de eventos permitem que você lide com interações específicas:

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
      <button onClick={handleSendClick}>Enviar</button>
    </>
  );
}
```

Com um manipulador de eventos, você pode ter certeza de que `sendMessage(message)` *apenas* será executado se o usuário pressionar o botão.

### Efeitos são executados sempre que a sincronização é necessária {/*effects-run-whenever-synchronization-is-needed*/}

Lembre-se que você também precisa manter o componente conectado à sala de bate-papo. Onde esse código vai?

A *razão* para executar este código não é alguma interação específica. Não importa por que ou como o usuário navegou para a tela da sala de bate-papo. Agora que eles estão olhando para ela e poderiam interagir com ela, o componente precisa ficar conectado ao servidor de bate-papo selecionado. Mesmo se o componente da sala de bate-papo fosse a tela inicial do seu aplicativo, e o usuário não tivesse executado nenhuma interação, você *ainda* precisaria conectar. É por isso que é um Efeito:

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

Com este código, você pode ter certeza de que sempre haverá uma conexão ativa com o servidor de bate-papo selecionado no momento, *independentemente* das interações específicas realizadas pelo usuário. Seja o usuário abrindo seu aplicativo, selecionando uma sala diferente ou navegando para outra tela e voltando, seu Efeito garante que o componente *permanecerá sincronizado* com a sala selecionada no momento e [se reconectará sempre que necessário.](/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once)

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
      <h1>Bem-vindo à sala {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Enviar</button>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar bate-papo' : 'Abrir bate-papo'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
export function sendMessage(message) {
  console.log('🔵 Você enviou: ' + message);
}

export function createConnection(serverUrl, roomId) {
  // Uma implementação real realmente se conectaria ao servidor
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
input, select { margin-right: 20px; }
```

</Sandpack>

## Valores reativos e lógica reativa {/*reactive-values-and-reactive-logic*/}

Intuitivamente, você pode dizer que os manipuladores de eventos são sempre acionados "manualmente", por exemplo, clicando em um botão. Efeitos, por outro lado, são "automáticos": eles são executados e reexecutados com a frequência necessária para permanecerem sincronizados.

Há uma maneira mais precisa de pensar nisso.

Props, estado e variáveis declaradas dentro do corpo do seu componente são chamados de <CodeStep step={2}>valores reativos</CodeStep>. Neste exemplo, `serverUrl` não é um valor reativo, mas `roomId` e `message` são. Eles participam do fluxo de dados de renderização:

```js [[2, 3, "roomId"], [2, 4, "message"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

Valores reativos como esses podem mudar devido a uma rerenderização. Por exemplo, o usuário pode editar a `message` ou escolher um `roomId` diferente em um menu suspenso. Manipuladores de eventos e Efeitos respondem às mudanças de maneira diferente:

- **A lógica dentro dos manipuladores de eventos *não é reativa.*** Ela não será executada novamente a menos que o usuário realize a mesma interação (por exemplo, um clique) novamente. Manipuladores de eventos podem ler valores reativos sem "reagir" às suas alterações.
- **A lógica dentro dos Efeitos é *reativa.*** Se seu Efeito lê um valor reativo, [você precisa especificá-lo como uma dependência.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Então, se uma rerenderização fizer com que esse valor mude, o React reexecutará a lógica do seu Efeito com o novo valor.

Vamos revisitar o exemplo anterior para ilustrar essa diferença.

### A lógica dentro dos manipuladores de eventos não é reativa {/*logic-inside-event-handlers-is-not-reactive*/}

Dê uma olhada nesta linha de código. Essa lógica deve ser reativa ou não?

```js [[2, 2, "message"]]
    // ...
    sendMessage(message);
    // ...
```

Da perspectiva do usuário, **uma alteração na `message` _não_ significa que ele deseja enviar uma mensagem.** Significa apenas que o usuário está digitando. Em outras palavras, a lógica que envia uma mensagem não deve ser reativa. Ela não deve ser executada novamente apenas porque o <CodeStep step={2}>valor reativo</CodeStep> mudou. É por isso que ela pertence ao manipulador de eventos:

```js {2}
  function handleSendClick() {
    sendMessage(message);
  }
```

Manipuladores de eventos não são reativos, então `sendMessage(message)` só será executado quando o usuário clicar no botão Enviar.

### A lógica dentro dos Efeitos é reativa {/*logic-inside-effects-is-reactive*/}

Agora, vamos voltar a estas linhas:

```js [[2, 2, "roomId"]]
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

Da perspectiva do usuário, **uma alteração no `roomId` *significa* que ele deseja se conectar a uma sala diferente.** Em outras palavras, a lógica para conectar à sala deve ser reativa. Você *quer* que essas linhas de código "acompanhem" o <CodeStep step={2}>valor reativo</CodeStep> e sejam executadas novamente se esse valor for diferente. É por isso que ela pertence a um Efeito:

```js {2-3}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Efeitos são reativos, então `createConnection(serverUrl, roomId)` e `connection.connect()` serão executados para cada valor distinto de `roomId`. Seu Efeito mantém a conexão de bate-papo sincronizada com a sala selecionada no momento.

## Extraindo lógica não reativa de Efeitos {/*extracting-non-reactive-logic-out-of-effects*/}

As coisas ficam mais complicadas quando você quer misturar lógica reativa com lógica não reativa.

Por exemplo, imagine que você queira mostrar uma notificação quando o usuário se conectar ao bate-papo. Você lê o tema atual (escuro ou claro) das props para poder mostrar a notificação na cor correta:

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

No entanto, `theme` é um valor reativo (ele pode mudar como resultado da rerenderização), e [todo valor reativo lido por um Efeito deve ser declarado como sua dependência.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Agora você precisa especificar `theme` como uma dependência do seu Efeito:

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

Brinque com este exemplo e veja se você consegue identificar o problema com essa experiência do usuário:

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

  return <h1>Bem-vindo à sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
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
  // Uma implementação real realmente se conectaria ao servidor
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
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'connected') {
        throw Error('Apenas o evento "connected" é suportado.');
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

Quando o `roomId` muda, o bate-papo se reconecta como você esperaria. Mas como `theme` também é uma dependência, o bate-papo *também* se reconecta toda vez que você alterna entre o tema escuro e o tema claro. Isso não é ótimo!

Em outras palavras, você *não* quer que esta linha seja reativa, embora esteja dentro de um Efeito (que é reativo):

```js
      // ...
      showNotification('Connected!', theme);
      // ...
```

Você precisa de uma forma de separar essa lógica não reativa do Efeito reativo ao seu redor.

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

Aqui, `onConnected` é chamado de um *Evento de Efeito*. É uma parte da lógica do seu Efeito, mas se comporta muito mais como um manipulador de eventos. A lógica dentro dele não é reativa, e ela sempre "vê" os últimos valores das suas props e estado.

Agora você pode chamar o Evento de Efeito `onConnected` de dentro do seu Efeito:

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

Isso resolve o problema. Observe que você precisou *remover* `onConnected` da lista de dependências do seu Efeito. **Eventos de Efeito não são reativos e devem ser omitidos das dependências.**

Verifique se o novo comportamento funciona como você esperaria:

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

  return <h1>Bem-vindo à sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
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
  // Uma implementação real realmente se conectaria ao servidor
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
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'connected') {
        throw Error('Apenas o evento "connected" é suportado.');
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

Você pode pensar em Eventos de Efeito como muito semelhantes aos manipuladores de eventos. A principal diferença é que os manipuladores de eventos são executados em resposta a interações do usuário, enquanto os Eventos de Efeito são acionados por você de Efeitos. Os Eventos de Efeito permitem que você "quebre a corrente" entre a reatividade dos Efeitos e o código que não deve ser reativo.

### Lendo as últimas props e estado com Eventos de Efeito {/*reading-latest-props-and-state-with-effect-events*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Eventos de Efeito permitem que você corrija muitos padrões em que você pode ser tentado a suprimir o linter de dependência.

Por exemplo, digamos que você tenha um Efeito para registrar as visitas à página:

```js
function Page() {
  useEffect(() => {
    logVisit();
  }, []);
  // ...
}
```

Mais tarde, você adiciona várias rotas ao seu site. Agora, seu componente `Page` recebe uma prop `url` com o caminho atual. Você quer passar a `url` como parte da sua chamada `logVisit`, mas o linter de dependência reclama:

```js {1,3}
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, []); // 🔴 React Hook useEffect tem uma dependência ausente: 'url'
  // ...
}
```

Pense sobre o que você quer que o código faça. Você *quer* registrar uma visita separada para diferentes URLs, pois cada URL representa uma página diferente. Em outras palavras, essa chamada `logVisit` *deve* ser reativa com relação à `url`. É por isso que, nesse caso, faz sentido seguir o linter de dependência e adicionar `url` como uma dependência:

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
  }, [url]); // 🔴 React Hook useEffect tem uma dependência ausente: 'numberOfItems'
  // ...
}
```

Você usou `numberOfItems` dentro do Efeito, então o linter pede que você adicione isso como uma dependência. No entanto, você *não* quer que a chamada `logVisit` seja reativa em relação a `numberOfItems`. Se o usuário colocar algo no carrinho de compras e o `numberOfItems` mudar, isso *não significa* que o usuário visitou a página novamente. Em outras palavras, *visitar a página* é, de certa forma, um "evento". Acontece em um momento preciso no tempo.

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

Aqui, `onVisit` é um Evento de Efeito. O código dentro dele não é reativo. É por isso que você pode usar `numberOfItems` (ou qualquer outro valor reativo!) sem se preocupar que isso fará com que o código circundante seja reexecutado nas alterações.

Por outro lado, o próprio Efeito permanece reativo. O código dentro do Efeito usa a prop `url`, então o Efeito será reexecutado após cada rerenderização com uma `url` diferente. Isso, por sua vez, chamará o Evento de Efeito `onVisit`.

Como resultado, você chamará `logVisit` para cada alteração na `url` e sempre lerá o último `numberOfItems`. No entanto, se `numberOfItems` mudar por conta própria, isso não fará com que nenhum dos códigos seja reexecutado.

<Note>

Você pode estar se perguntando se você pode chamar `onVisit()` sem argumentos e ler a `url` dentro dele:

```js {2,6}
  const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
  });

  useEffect(() => {
    onVisit();
  }, [url]);
```

Isso funcionaria, mas é melhor passar esse `url` para o Evento de Efeito explicitamente. **Ao passar `url` como um argumento para o seu Evento de Efeito, você está dizendo que visitar uma página com uma `url` diferente constitui um "evento" separado da perspectiva do usuário.** O `visitedUrl` é uma *parte* do "evento" que aconteceu:

```js {1-2,6}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
```

Como seu Evento de Efeito "pergunta" explicitamente pela `visitedUrl`, agora você não pode remover acidentalmente `url` das dependências do Efeito. Se você remover a dependência `url` (fazendo com que visitas a páginas distintas sejam contadas como uma), o linter avisará sobre isso. Você quer que `onVisit` seja reativo em relação à `url`, então, em vez de ler a `url` dentro (onde ela não seria reativa), você a passa *de* seu Efeito.

Isso se torna especialmente importante se houver alguma lógica assíncrona dentro do Efeito:

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

Aqui, a `url` dentro de `onVisit` corresponde à *última* `url` (que pode já ter mudado), mas `visitedUrl` corresponde à `url` que originalmente fez com que esse Efeito (e essa chamada `onVisit`) fosse executado.

</Note>

<DeepDive>

#### É aceitável suprimir o linter de dependência em vez disso? {/*is-it-okay-to-suppress-the-dependency-linter-instead*/}

Nas bases de código existentes, você pode, às vezes, ver a regra de lint suprimida assim:

```js {7-9}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 Evite suprimir o linter assim:
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [url]);
  // ...
}
```

Depois que `useEffectEvent` se tornar uma parte estável do React, recomendamos **nunca suprimir o linter**.

A primeira desvantagem de suprimir a regra é que o React não avisará mais quando seu Efeito precisar "reagir" a uma nova dependência reativa que você introduziu em seu código. No exemplo anterior, você adicionou `url` às dependências *porque* o React lembrou você de fazê-lo. Você não receberá mais lembretes para nenhuma edição futura nesse Efeito se desativar o linter. Isso leva a erros.

Aqui está um exemplo de um bug confuso causado pela supressão do linter. Neste exemplo, a função `handleMove` deve ler o valor atual da variável de estado `canMove` para decidir se o ponto deve seguir o cursor. No entanto, `canMove` é sempre `true` dentro de `handleMove`.

Você consegue ver por quê?

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
        É permitido que o ponto se mova
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

O problema com este código está em suprimir o linter de dependência. Se você remover a supressão, verá que este Efeito deve depender da função `handleMove`. Isso faz sentido: `handleMove` é declarado dentro do corpo do componente, o que o torna um valor reativo. Cada valor reativo deve ser especificado como uma dependência, ou ele pode ficar desatualizado com o tempo!

O autor do código original "mentiu" para o React, dizendo que o Efeito não depende (`[]`) de nenhum valor reativo. É por isso que o React não ressincronizou o Efeito após `canMove` ter mudado (e `handleMove` com ele). Como o React não ressincronizou o Efeito, o `handleMove` anexado como um listener é a função `handleMove` criada durante a renderização inicial. Durante a renderização inicial, `canMove` era `true`, e é por isso que `handleMove` da renderização inicial sempre verá esse valor.

**Se você nunca suprimir o linter, nunca verá problemas com valores desatualizados.**

Com `useEffectEvent`, não há necessidade de "mentir" para o linter, e o código funciona da maneira que você esperaria:

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
        O ponto pode se mover
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

Isso não significa que o `useEffectEvent` é *sempre* a solução correta. Você deve aplicá-lo apenas às linhas de código que não deseja que sejam reativas. No sandbox acima, você não queria que o código do Effect fosse reativo em relação a `canMove`. É por isso que fez sentido extrair um Effect Event.

Leia [Removendo as Dependências do Effect](/learn/removing-effect-dependencies) para outras alternativas corretas para suprimir o linter.

</DeepDive>

### Limitações de Effect Events {/*limitations-of-effect-events*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Effect Events são muito limitados em como você pode usá-los:

*   **Só chame-os de dentro de Effects.**
*   **Nunca os passe para outros componentes ou Hooks.**

Por exemplo, não declare nem passe um Effect Event assim:

```js {4-6,8}
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 Evitar: Passando Effect Events

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
  }, [delay, callback]); // Precisa especificar "callback" nas dependências
}
```

Em vez disso, declare sempre os Effect Events diretamente ao lado dos Effects que os usam:

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
      onTick(); // ✅ Bom: Chamado apenas localmente dentro de um Effect
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // Não precisa especificar "onTick" (um Effect Event) como uma dependência
}
```

Effect Events são "pedaços" não reativos do seu código de Effect. Eles devem estar próximos ao Effect que os usa.

<Recap>

-   Manipuladores de eventos são executados em resposta a interações específicas.
-   Effects são executados sempre que a sincronização é necessária.
-   A lógica dentro dos manipuladores de eventos não é reativa.
-   A lógica dentro dos Effects é reativa.
-   Você pode mover a lógica não reativa dos Effects para Effect Events.
-   Só chame Effect Events de dentro de Effects.
-   Não passe Effect Events para outros componentes ou Hooks.

</Recap>

<Challenges>

#### Corrigir uma variável que não atualiza {/*fix-a-variable-that-doesnt-update*/}

Este componente `Timer` mantém uma variável de estado `count` que aumenta a cada segundo. O valor pelo qual está aumentando é armazenado na variável de estado `increment`. Você pode controlar a variável `increment` com os botões de mais e menos.

No entanto, não importa quantas vezes você clique no botão de mais, o contador ainda é incrementado em um a cada segundo. O que há de errado com este código? Por que `increment` é sempre igual a `1` dentro do código do Effect? Encontre o erro e corrija-o.

<Hint>

Para corrigir este código, basta seguir as regras.

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
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        A cada segundo, incremente por:
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

Como de costume, ao procurar erros em Effects, comece pesquisando por supressões de linter.

Se você remover o comentário de supressão, o React informará que o código deste Effect depende de `increment`, mas você "mentiu" para o React ao afirmar que este Effect não depende de nenhum valor reativo (`[]`). Adicione `increment` à matriz de dependência:

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
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        A cada segundo, incremente por:
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

Agora, quando `increment` muda, o React ressincronizará seu Effect, que reiniciará o intervalo.

</Solution>

#### Corrigir um contador que congela {/*fix-a-freezing-counter*/}

Este componente `Timer` mantém uma variável de estado `count` que aumenta a cada segundo. O valor pelo qual está aumentando é armazenado na variável de estado `increment`, a qual você pode controlar com os botões de mais e menos. Por exemplo, tente pressionar o botão de mais nove vezes e observe que o `count` agora aumenta a cada segundo em dez, em vez de um.

Há um pequeno problema com esta interface do usuário. Você pode notar que se você continuar pressionando os botões de mais ou menos mais rápido do que uma vez por segundo, o próprio temporizador parece pausar. Ele só retoma depois que um segundo se passa desde a última vez em que você pressionou qualquer um dos botões. Descubra por que isso está acontecendo e corrija o problema para que o cronômetro marque a *cada* segundo, sem interrupções.

<Hint>

Parece que o Effect que configura o cronômetro "reage" ao valor `increment`. A linha que usa o valor `increment` atual para chamar `setCount` realmente precisa ser reativa?

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
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        A cada segundo, incremente por:
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

O problema é que o código dentro do Effect usa a variável de estado `increment`. Como é uma dependência do seu Effect, cada alteração em `increment` faz com que o Effect seja ressincronizado, o que faz com que o intervalo seja limpo. Se você continuar limpando o intervalo toda vez antes que ele tenha a chance de disparar, parecerá que o temporizador parou.

Para resolver o problema, extraia um Effect Event `onTick` do Effect:

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
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        A cada segundo, incremente por:
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

Como `onTick` é um Effect Event, o código dentro dele não é reativo. A alteração para `increment` não aciona nenhum efeito.

</Solution>

#### Corrigir um atraso não ajustável {/*fix-a-non-adjustable-delay*/}

Neste exemplo, você pode personalizar o atraso do intervalo. Ele é armazenado na variável de estado `delay`, que é atualizada por dois botões. No entanto, mesmo que você pressione o botão "mais 100 ms" até que o `delay` seja 1000 milissegundos (ou seja, um segundo), você notará que o temporizador ainda aumenta muito rápido (a cada 100 ms). É como se suas alterações no `delay` fossem ignoradas. Encontre e corrija o erro.

<Hint>

O código dentro dos Effect Events não é reativo. Há casos em que você _desejaria_ que a chamada `setInterval` fosse executada novamente?

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
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        Incrementar por:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        Atraso do incremento:
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

O problema com o exemplo acima é que ele extraiu um Effect Event chamado `onMount` sem considerar o que o código deveria realmente estar fazendo. Você só deve extrair Effect Events por uma razão específica: quando deseja tornar uma parte do seu código não reativa. No entanto, a chamada `setInterval` *deve* ser reativa em relação à variável de estado `delay`. Se o `delay` mudar, você deseja configurar o intervalo do zero! Para consertar este código, puxe todo o código reativo de volta para dentro do Effect:

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
        Contador: {count}
        <button onClick={() => setCount(0)}>Redefinir</button>
      </h1>
      <hr />
      <p>
        Incrementar por:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        Atraso do incremento:
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

Em geral, você deve desconfiar de funções como `onMount` que se concentram no *tempo* em vez do *propósito* de um pedaço de código. Pode parecer "mais descritivo" a princípio, mas oculta sua intenção. Como regra geral, Effect Events devem corresponder a algo que acontece da perspectiva do *usuário*. Por exemplo, `onMessage`, `onTick`, `onVisit` ou `onConnected` são bons nomes de Effect Event. O código dentro deles provavelmente não precisaria ser reativo. Por outro lado, `onMount`, `onUpdate`, `onUnmount` ou `onAfterRender` são tão genéricos que é fácil colocar acidentalmente neles código que *deveria* ser reativo. É por isso que você deve nomear seus Effect Events de acordo com *o que o usuário acha que aconteceu*, não quando algum código aconteceu para ser executado.

</Solution>

#### Corrigir uma notificação atrasada {/*fix-a-delayed-notification*/}

Quando você entra em uma sala de bate-papo, este componente mostra uma notificação. No entanto, ele não mostra a notificação imediatamente. Em vez disso, a notificação é artificialmente atrasada por dois segundos para que o usuário tenha a chance de dar uma olhada na IU.

Isso quase funciona, mas há um erro. Tente mudar o menu suspenso de "geral" para "viagem" e depois para "música" muito rapidamente. Se você fizer isso rápido o suficiente, verá duas notificações (como esperado!), mas elas *ambas* dirão "Bem-vindo à música".

Corrija-o para que, ao mudar de "geral" para "viagem" e depois para "música" muito rapidamente, você veja duas notificações; a primeira sendo "Bem-vindo à viagem" e a segunda sendo "Bem-vindo à música". (Para um desafio adicional, supondo que você já tenha feito as notificações mostrarem os quartos corretos, altere o código para que apenas a última notificação seja exibida.)

<Hint>

Seu Effect sabe para qual sala ele se conectou. Há alguma informação que você pode querer passar para o seu Effect Event?

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
    showNotification('Bem-vindo a ' + roomId, theme);
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

  return <h1>Bem-vindo à sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
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
  // Uma implementação real realmente se conectaria ao servidor
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
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'connected') {
        throw Error('Apenas o evento "conectado" é suportado.');
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

Dentro do seu Effect Event, `roomId` é o valor *no momento em que o Effect Event foi chamado.*

Seu Effect Event é chamado com um atraso de dois segundos. Se você estiver mudando rapidamente da sala de viagem para a sala de música, quando a notificação da sala de viagens aparecer, `roomId` já será `"music"`. É por isso que ambas as notificações dizem "Bem-vindo à música".

Para corrigir o problema, em vez de ler o `roomId` *mais recente* dentro do Effect Event, torne-o um parâmetro do seu Effect Event, como `connectedRoomId` abaixo. Em seguida, passe `roomId` do seu Effect chamando `onConnected(roomId)`:

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
    showNotification('Bem-vindo a ' + connectedRoomId, theme);
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

  return <h1>Bem-vindo à sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
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
  // Uma implementação real realmente se conectaria ao servidor
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
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'connected') {
        throw Error('Apenas o evento "conectado" é suportado.');
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

O Effect que tinha `roomId` definido como `"travel"` (então ele se conectou à sala `"travel"`) mostrará a notificação para `"travel"`. O Effect que tinha `roomId` definido como `"music"` (então ele se conectou à sala `"music"`) mostrará a notificação para `"music"`. Em outras palavras, `connectedRoomId` vem do seu Effect (que é reativo), enquanto `theme` sempre usa o valor mais recente.

Para resolver o desafio adicional, salve o ID do tempo limite da notificação e limpe-o na função de limpeza do seu Effect:

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
    showNotification('Bem-vindo a ' + connectedRoomId, theme);
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

  return <h1>Bem-vindo à sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
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
  // Uma implementação real realmente se conectaria ao servidor
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
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'connected') {
        throw Error('Apenas o evento "conectado" é suportado.');
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

Isso garante que as notificações já agendadas (mas ainda não exibidas) sejam canceladas quando você muda de sala.

</Solution>

</Challenges>
```