---
title: "# Separando Eventos de Efeitos


  Em um aplicativo React, você pode precisar executar algumas ações em resposta
  a eventos. Por exemplo, você pode querer atualizar o estado quando o usuário
  digita em um campo de entrada ou exibir uma mensagem de sucesso após o envio
  de um formulário. **Eventos** são as interações do usuário (como clicar em um
  botão, digitar em um campo de entrada ou passar o mouse sobre um elemento),
  enquanto **efeitos** são as atualizações que você deseja fazer em resposta a
  esses eventos (como atualizar o estado, alterar o DOM ou enviar uma
  solicitação de rede).


  Nesta página, você aprenderá como separar eventos de efeitos para tornar seus
  componentes mais fáceis de entender e depurar.


  ## Eventos e Efeitos


  Vamos começar com um exemplo simples. Digamos que você tenha um componente que
  exibe uma mensagem de saudação e um botão. Quando o usuário clica no botão,
  você deseja atualizar a mensagem de saudação.


  Aqui está uma versão inicial do componente:


  ```js

  import { useState } from 'react';


  function MyComponent() {

  \  const [message, setMessage] = useState('Olá, mundo!');


  \  function handleClick() {

  \    setMessage('Olá, React!');

  \  }


  \  return (

  \    <button onClick={handleClick}>

  \      {message}

  \    </button>

  \  );

  }

  ```


  Neste exemplo, o evento é o clique no botão, e o efeito é a atualização da
  mensagem de saudação. O manipulador de eventos `handleClick` atualiza o estado
  `message` quando o botão é clicado.


  Embora este exemplo funcione, ele não separa claramente o evento do efeito. O
  manipulador de eventos `handleClick` contém tanto o evento (o clique) quanto o
  efeito (a atualização do estado). Isso pode tornar o código mais difícil de
  entender e depurar, especialmente em componentes mais complexos.


  ## Separando Eventos de Efeitos


  Para separar eventos de efeitos, você pode usar uma função separada para lidar
  com o efeito. Isso torna o código mais legível e fácil de manter.


  Aqui está uma versão refatorada do componente:


  ```js

  import { useState } from 'react';


  function MyComponent() {

  \  const [message, setMessage] = useState('Olá, mundo!');


  \  function handleButtonClick() {

  \    // Evento: clique no botão

  \    // Efeito: atualizar a mensagem

  \    setMessage('Olá, React!');

  \  }


  \  return (

  \    <button onClick={handleButtonClick}>

  \      {message}

  \    </button>

  \  );

  }

  ```


  Neste exemplo, o manipulador de eventos `handleButtonClick` agora contém
  apenas o evento (o clique no botão). A função `setMessage` é chamada dentro do
  manipulador de eventos para atualizar o estado, que é o efeito.


  Ao separar eventos de efeitos, você pode tornar seus componentes mais fáceis
  de entender e depurar. Você também pode reutilizar os efeitos em diferentes
  manipuladores de eventos.


  ## Exemplo: Envio de um Formulário


  Vamos dar uma olhada em um exemplo mais complexo. Digamos que você tenha um
  formulário que envia dados para um servidor. Quando o usuário envia o
  formulário, você deseja exibir uma mensagem de sucesso.


  Aqui está uma versão inicial do componente:


  ```js

  import { useState } from 'react';


  function MyForm() {

  \  const [isSubmitting, setIsSubmitting] = useState(false);

  \  const [isSuccess, setIsSuccess] = useState(false);


  \  async function handleSubmit(event) {

  \    event.preventDefault();

  \    setIsSubmitting(true);


  \    try {

  \      // Simular uma solicitação de rede

  \      await new Promise((resolve) => setTimeout(resolve, 2000));

  \      setIsSuccess(true);

  \    } catch (error) {

  \      // Lidar com erros

  \    } finally {

  \      setIsSubmitting(false);

  \    }

  \  }


  \  return (

  \    <form onSubmit={handleSubmit}>

  \      <button type=\"submit\" disabled={isSubmitting}>

  \        {isSubmitting ? 'Enviando...' : 'Enviar'}

  \      </button>

  \      {isSuccess && <p>Enviado com sucesso!</p>}

  \    </form>

  \  );

  }

  ```


  Neste exemplo, o evento é o envio do formulário, e os efeitos são:


  *   Definir `isSubmitting` para `true`

  *   Enviar uma solicitação de rede (simulada)

  *   Definir `isSuccess` para `true`

  *   Definir `isSubmitting` para `false`


  O manipulador de eventos `handleSubmit` contém tanto o evento (o envio do
  formulário) quanto os efeitos (as atualizações de estado).


  Para separar eventos de efeitos, você pode usar funções separadas para lidar
  com os efeitos.


  Aqui está uma versão refatorada do componente:


  ```js

  import { useState } from 'react';


  function MyForm() {

  \  const [isSubmitting, setIsSubmitting] = useState(false);

  \  const [isSuccess, setIsSuccess] = useState(false);


  \  async function handleSubmit(event) {

  \    event.preventDefault();

  \    handleFormSubmit();

  \  }


  \  async function handleFormSubmit() {

  \    // Evento: envio do formulário

  \    // Efeitos:

  \    // 1. Definir isSubmitting para true

  \    setIsSubmitting(true);


  \    try {

  \      // Simular uma solicitação de rede

  \      await new Promise((resolve) => setTimeout(resolve, 2000));

  \      // 2. Definir isSuccess para true

  \      setIsSuccess(true);

  \    } catch (error) {

  \      // Lidar com erros

  \    } finally {

  \      // 3. Definir isSubmitting para false

  \      setIsSubmitting(false);

  \    }

  \  }


  \  return (

  \    <form onSubmit={handleSubmit}>

  \      <button type=\"submit\" disabled={isSubmitting}>

  \        {isSubmitting ? 'Enviando...' : 'Enviar'}

  \      </button>

  \      {isSuccess && <p>Enviado com sucesso!</p>}

  \    </form>

  \  );

  }

  ```


  Neste exemplo, o manipulador de eventos `handleSubmit` agora contém apenas o
  evento (o envio do formulário). A função `handleFormSubmit` contém os efeitos
  (as atualizações de estado).


  ## Benefícios de Separar Eventos de Efeitos


  Separar eventos de efeitos oferece vários benefícios:


  *   **Melhor legibilidade:** O código se torna mais fácil de entender, pois os
  eventos e os efeitos são separados.

  *   **Melhor capacidade de manutenção:** O código se torna mais fácil de
  manter, pois os efeitos podem ser reutilizados em diferentes manipuladores de
  eventos.

  *   **Melhor depuração:** O código se torna mais fácil de depurar, pois você
  pode isolar os efeitos e testá-los separadamente.

  *   **Melhor organização:** O código se torna mais organizado, pois você pode
  agrupar os efeitos em funções separadas.


  ## Conclusão


  Separar eventos de efeitos é uma boa prática que pode tornar seus componentes
  React mais fáceis de entender, depurar e manter. Ao separar eventos de
  efeitos, você pode escrever um código mais limpo e organizado que é mais fácil
  de trabalhar."
---
```markdown
## Guia de Estilo Universal

Este documento descreve as regras que devem ser aplicadas para **todos** os idiomas.
Quando estiver se referindo ao próprio `React`, use `o React`.

## IDs dos Títulos

Todos os títulos possuem IDs explícitos como abaixo:

```md
## Tente React {#try-react}
```

**Não** traduza estes IDs! Eles são usado para navegação e quebrarão se o documento for um link externo, como:

```md
Veja a [seção iniciando](/getting-started#try-react) para mais informações.
```

✅ FAÇA:

```md
## Tente React {#try-react}
```

❌ NÃO FAÇA:

```md
## Tente React {#tente-react}
```

Isto quebraria o link acima.

## Texto em Blocos de Código

Mantenha o texto em blocos de código sem tradução, exceto para os comentários. Você pode optar por traduzir o texto em strings, mas tenha cuidado para não traduzir strings que se refiram ao código!

Exemplo:

```js
// Example
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ FAÇA:

```js
// Exemplo
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ PERMITIDO:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

❌ NÃO FAÇA:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
// "root" se refere a um ID de elemento.
// NÃO TRADUZA
ReactDOM.render(element, document.getElementById('raiz'));
```

❌ DEFINITIVAMENTE NÃO FAÇA:

```js
// Exemplo
const elemento = <h1>Olá mundo</h1>;
ReactDOM.renderizar(elemento, documento.obterElementoPorId('raiz'));
```

## Links Externos

Se um link externo se referir a um artigo no [MDN] or [Wikipedia] e se houver uma versão traduzida em seu idioma em uma qualidade decente, opte por usar a versão traduzida.

[mdn]: https://developer.mozilla.org/pt-BR/
[wikipedia]: https://pt.wikipedia.org/wiki/Wikipédia:Página_principal

Exemplo:

```md
React elements are [immutable](https://en.wikipedia.org/wiki/Immutable_object).
```

✅ OK:

```md
Elementos React são [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável).
```

Para links que não possuem tradução (Stack Overflow, vídeos do YouTube, etc.), simplesmente use o link original.

## Traduções Comuns

Sugestões de palavras e termos:

| Palavra/Termo original | Sugestão                               |
| ---------------------- | -------------------------------------- |
| assertion              | asserção                               |
| at the top level       | na raiz                                |
| browser                | navegador                              |
| bubbling               | propagar                               |
| bug                    | erro                                   |
| caveats                | ressalvas                              |
| class component        | componente de classe                   |
| class                  | classe                                 |
| client                 | cliente                                |
| client-side            | lado do cliente                        |
| container              | contêiner                              |
| context                | contexto                               |
| controlled component   | componente controlado                  |
| debugging              | depuração                              |
| DOM node               | nó do DOM                              |
| event handler          | manipulador de eventos (event handler) |
| function component     | componente de função                   |
| handler                | manipulador                            |
| helper function        | função auxiliar                        |
| high-order components  | componente de alta-ordem               |
| key                    | chave                                  |
| library                | biblioteca                             |
| lowercase              | minúscula(s) / caixa baixa             |
| package                | pacote                                 |
| React element          | Elemento React                         |
| React fragment         | Fragmento React                        |
| render                 | renderizar (verb), renderizado (noun)  |
| server                 | servidor                               |
| server-side            | lado do servidor                       |
| siblings               | irmãos                                 |
| stateful component     | componente com estado                  |
| stateful logic         | lógica com estado                      |
| to assert              | afirmar                                |
| to wrap                | encapsular                             |
| troubleshooting        | solução de problemas                   |
| uncontrolled component | componente não controlado              |
| uppercase              | maiúscula(s) / caixa alta              |

## Conteúdo que não deve ser traduzido

- array
- arrow function
- bind
- bundle
- bundler
- callback
- camelCase
- DOM
- event listener
- framework
- hook
- log
- mock
- portal
- props
- ref
- release
- script
- single-page-apps
- state
- string
- string literal
- subscribe
- subscription
- template literal
- timestamps
- UI
- watcher
- widgets
- wrapper

## Introdução

Manipuladores de eventos só são executados novamente quando você realiza a mesma interação novamente. Diferente dos manipuladores de eventos, os Effects ressincronizam se algum valor que eles leem, como uma prop ou uma variável de estado, for diferente do que era durante a última renderização. Às vezes, você também quer uma mistura de ambos os comportamentos: um Effect que é executado novamente em resposta a alguns valores, mas não a outros. Esta página vai te ensinar como fazer isso.

</Intro>

<YouWillLearn>

- Como escolher entre um manipulador de eventos e um Effect
- Por que os Effects são reativos, e os manipuladores de eventos não são
- O que fazer quando você quer que uma parte do código do seu Effect não seja reativa
- O que são Eventos de Effect, e como extraí-los dos seus Effects
- Como ler as últimas props e estado dos Effects usando Eventos de Effect

</YouWillLearn>

## Escolhendo entre manipuladores de eventos e Effects {/*choosing-between-event-handlers-and-effects*/}

Primeiro, vamos recapitular a diferença entre manipuladores de eventos e Effects.

Imagine que você está implementando um componente de sala de bate-papo. Seus requisitos são estes:

1. Seu componente deve se conectar automaticamente à sala de bate-papo selecionada.
1. Quando você clicar no botão "Enviar", ele deve enviar uma mensagem para o bate-papo.

Digamos que você já implementou o código para eles, mas não tem certeza de onde colocá-lo. Você deve usar manipuladores de eventos ou Effects? Toda vez que você precisar responder a esta pergunta, considere [*por que* o código precisa ser executado.](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

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

Com um manipulador de eventos, você pode ter certeza de que `sendMessage(message)` *somente* será executado se o usuário pressionar o botão.

### Effects são executados sempre que a sincronização é necessária {/*effects-run-whenever-synchronization-is-needed*/}

Lembre-se de que você também precisa manter o componente conectado à sala de bate-papo. Onde esse código vai?

A *razão* para executar este código não é alguma interação específica. Não importa por que ou como o usuário navegou para a tela da sala de bate-papo. Agora que eles estão olhando para ela e podem interagir com ela, o componente precisa permanecer conectado ao servidor de bate-papo selecionado. Mesmo que o componente da sala de bate-papo fosse a tela inicial do seu aplicativo, e o usuário não tivesse realizado nenhuma interação, você *ainda* precisaria se conectar. É por isso que é um Effect:

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

Com este código, você pode ter certeza de que sempre há uma conexão ativa com o servidor de bate-papo atualmente selecionado, *independentemente* das interações específicas realizadas pelo usuário. Se o usuário acabou de abrir seu aplicativo, selecionou uma sala diferente ou navegou para outra tela e voltou, seu Effect garante que o componente *permanecerá sincronizado* com a sala atualmente selecionada e [se reconectará sempre que for necessário.](/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once)

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
      <h1>Bem-vindo(a) à sala {roomId}!</h1>
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

Intuitivamente, você pode dizer que os manipuladores de eventos são sempre acionados "manualmente", por exemplo, clicando em um botão. Os Effects, por outro lado, são "automáticos": eles são executados e reexecutados com a frequência necessária para permanecer sincronizados.

Há uma maneira mais precisa de pensar sobre isso.

Props, estado e variáveis declaradas dentro do corpo do seu componente são chamados de <CodeStep step={2}>valores reativos</CodeStep>. Neste exemplo, `serverUrl` não é um valor reativo, mas `roomId` e `message` são. Eles participam do fluxo de dados de renderização:

```js [[2, 3, "roomId"], [2, 4, "message"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

Valores reativos como esses podem mudar devido a uma nova renderização. Por exemplo, o usuário pode editar a `message` ou escolher um `roomId` diferente em um menu suspenso. Manipuladores de eventos e Effects respondem às mudanças de forma diferente:

- **A lógica dentro dos manipuladores de eventos *não é reativa.*** Ela não será executada novamente, a menos que o usuário realize a mesma interação (por exemplo, um clique) novamente. Manipuladores de eventos podem ler valores reativos sem "reagir" às suas mudanças.
- **A lógica dentro dos Effects é *reativa.*** Se seu Effect lê um valor reativo, [você deve especificá-lo como uma dependência.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Então, se uma nova renderização fizer com que esse valor mude, o React reexecutará a lógica do seu Effect com o novo valor.

Vamos revisitar o exemplo anterior para ilustrar essa diferença.

### A lógica dentro dos manipuladores de eventos não é reativa {/*logic-inside-event-handlers-is-not-reactive*/}

Dê uma olhada nesta linha de código. Essa lógica deve ser reativa ou não?

```js [[2, 2, "message"]]
    // ...
    sendMessage(message);
    // ...
```

Da perspectiva do usuário, **uma mudança na `message` _não_ significa que eles querem enviar uma mensagem.** Significa apenas que o usuário está digitando. Em outras palavras, a lógica que envia uma mensagem não deve ser reativa. Ela não deve ser executada novamente apenas porque o <CodeStep step={2}>valor reativo</CodeStep> mudou. É por isso que ela pertence ao manipulador de eventos:

```js {2}
  function handleSendClick() {
    sendMessage(message);
  }
```

Manipuladores de eventos não são reativos, então `sendMessage(message)` só será executado quando o usuário clicar no botão Enviar.

### A lógica dentro dos Effects é reativa {/*logic-inside-effects-is-reactive*/}

Agora, vamos voltar a estas linhas:

```js [[2, 2, "roomId"]]
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

Da perspectiva do usuário, **uma mudança na `roomId` *significa* que eles querem se conectar a uma sala diferente.** Em outras palavras, a lógica para se conectar à sala deve ser reativa. Você *quer* que essas linhas de código "acompanhem" o <CodeStep step={2}>valor reativo</CodeStep> e sejam executadas novamente se esse valor for diferente. É por isso que ela pertence a um Effect:

```js {2-3}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Effects são reativos, então `createConnection(serverUrl, roomId)` e `connection.connect()` serão executados para cada valor distinto de `roomId`. Seu Effect mantém a conexão do bate-papo sincronizada com a sala atualmente selecionada.
```

## Extraindo lógica não reativa de Effects {/*extracting-non-reactive-logic-out-of-effects*/}

As coisas ficam mais complicadas quando você deseja misturar lógica reativa com lógica não reativa.

Por exemplo, imagine que você deseja mostrar uma notificação quando o usuário se conecta ao chat. Você lê o tema atual (escuro ou claro) das props para poder mostrar a notificação na cor correta:

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

No entanto, `theme` é um valor reativo (ele pode mudar como resultado da renderização) e [todo valor reativo lido por um Effect deve ser declarado como sua dependência.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Agora você precisa especificar `theme` como uma dependência do seu Effect:

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

Brinque com este exemplo e veja se você consegue identificar o problema com esta experiência do usuário:

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

Em outras palavras, você *não* quer que esta linha seja reativa, mesmo que esteja dentro de um Effect (que é reativo):

```js
      // ...
      showNotification('Connected!', theme);
      // ...
```

Você precisa de uma maneira de separar essa lógica não reativa do Effect reativo ao seu redor.

### Declarando um Evento de Effect {/*declaring-an-effect-event*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Use um Hook especial chamado [`useEffectEvent`](/reference/react/experimental_useEffectEvent) para extrair essa lógica não reativa do seu Effect:

```js {1,4-6}
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

Aqui, `onConnected` é chamado de *Evento de Effect.* Ele faz parte da lógica do seu Effect, mas se comporta muito mais como um manipulador de eventos. A lógica dentro dele não é reativa e sempre "vê" os valores mais recentes de suas props e estado.

Agora você pode chamar o Evento de Effect `onConnected` de dentro do seu Effect:

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

Isso resolve o problema. Observe que você teve que *remover* `theme` da lista de dependências do seu Effect, porque ele não é mais usado no Effect. Você também não precisa *adicionar* `onConnected` a ele, porque **Eventos de Effect não são reativos e devem ser omitidos das dependências.**

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

Você pode pensar nos Eventos de Effect como sendo muito semelhantes aos manipuladores de eventos. A principal diferença é que os manipuladores de eventos são executados em resposta às interações do usuário, enquanto os Eventos de Effect são acionados por você a partir de Effects. Os Eventos de Effect permitem que você "quebre a corrente" entre a reatividade dos Effects e o código que não deve ser reativo.


```markdown
### Lendo as últimas props e estado com Eventos de Efeito {/*reading-latest-props-and-state-with-effect-events*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Eventos de Efeito permitem que você corrija muitos padrões em que você pode ser tentado a suprimir o linter de dependência.

Por exemplo, digamos que você tenha um Effect para registrar as visitas à página:

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
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'url'
  // ...
}
```

Pense no que você quer que o código faça. Você *quer* registrar uma visita separada para diferentes URLs, pois cada URL representa uma página diferente. Em outras palavras, esta chamada `logVisit` *deve* ser reativa em relação à `url`. É por isso que, neste caso, faz sentido seguir o linter de dependência e adicionar `url` como uma dependência:

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

Você usou `numberOfItems` dentro do Effect, então o linter pede que você o adicione como uma dependência. No entanto, você *não* quer que a chamada `logVisit` seja reativa em relação a `numberOfItems`. Se o usuário colocar algo no carrinho de compras e o `numberOfItems` mudar, isso *não significa* que o usuário visitou a página novamente. Em outras palavras, *visitar a página* é, em certo sentido, um "evento". Acontece em um momento preciso no tempo.

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

Aqui, `onVisit` é um Evento de Efeito. O código dentro dele não é reativo. É por isso que você pode usar `numberOfItems` (ou qualquer outro valor reativo!) sem se preocupar que isso fará com que o código circundante seja reexecutado em mudanças.

Por outro lado, o próprio Effect permanece reativo. O código dentro do Effect usa a prop `url`, então o Effect será executado novamente após cada re-renderização com uma `url` diferente. Isso, por sua vez, chamará o Evento de Efeito `onVisit`.

Como resultado, você chamará `logVisit` para cada alteração na `url` e sempre lerá o último `numberOfItems`. No entanto, se `numberOfItems` mudar por conta própria, isso não fará com que nenhum dos códigos seja reexecutado.

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

Isso funcionaria, mas é melhor passar esta `url` para o Evento de Efeito explicitamente. **Ao passar `url` como um argumento para seu Evento de Efeito, você está dizendo que visitar uma página com uma `url` diferente constitui um "evento" separado da perspectiva do usuário.** A `visitedUrl` é uma *parte* do "evento" que aconteceu:

```js {1-2,6}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
```

Como seu Evento de Efeito "pede" explicitamente a `visitedUrl`, agora você não pode remover acidentalmente a `url` das dependências do Effect. Se você remover a dependência `url` (fazendo com que visitas distintas à página sejam contadas como uma), o linter o avisará sobre isso. Você quer que `onVisit` seja reativo em relação à `url`, então, em vez de ler a `url` dentro (onde não seria reativo), você a passa *de* seu Effect.

Isso se torna especialmente importante se houver alguma lógica assíncrona dentro do Effect:

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

Aqui, `url` dentro de `onVisit` corresponde à *última* `url` (que já poderia ter mudado), mas `visitedUrl` corresponde à `url` que originalmente causou a execução deste Effect (e desta chamada `onVisit`).

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

A primeira desvantagem de suprimir a regra é que o React não o avisará mais quando seu Effect precisar "reagir" a uma nova dependência reativa que você introduziu em seu código. No exemplo anterior, você adicionou `url` às dependências *porque* o React lembrou você de fazê-lo. Você não receberá mais esses lembretes para quaisquer edições futuras nesse Effect se desativar o linter. Isso leva a erros.

Aqui está um exemplo de um erro confuso causado pela supressão do linter. Neste exemplo, a função `handleMove` deve ler o valor atual da variável de estado `canMove` para decidir se o ponto deve seguir o cursor. No entanto, `canMove` é sempre `true` dentro de `handleMove`.

Você consegue ver o porquê?

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

O problema com este código está em suprimir o linter de dependência. Se você remover a supressão, verá que este Effect deve depender da função `handleMove`. Isso faz sentido: `handleMove` é declarado dentro do corpo do componente, o que o torna um valor reativo. Cada valor reativo deve ser especificado como uma dependência, ou ele pode ficar obsoleto com o tempo!

O autor do código original "mentiu" para o React, dizendo que o Effect não depende (`[]`) de nenhum valor reativo. É por isso que o React não ressincronizou o Effect depois que `canMove` mudou (e `handleMove` com ele). Como o React não ressincronizou o Effect, o `handleMove` anexado como um listener é a função `handleMove` criada durante a renderização inicial. Durante a renderização inicial, `canMove` era `true`, e é por isso que `handleMove` da renderização inicial sempre verá esse valor.

**Se você nunca suprimir o linter, nunca verá problemas com valores obsoletos.**

Com `useEffectEvent`, não há necessidade de "mentir" para o linter, e o código funciona como você esperaria:

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

Isso não significa que `useEffectEvent` é *sempre* a solução correta. Você só deve aplicá-lo às linhas de código que você não quer que sejam reativas. No sandbox acima, você não queria que o código do Effect fosse reativo em relação a `canMove`. É por isso que fez sentido extrair um Evento de Efeito.

Leia [Removendo Dependências de Efeito](/learn/removing-effect-dependencies) para outras alternativas corretas para suprimir o linter.

</DeepDive>
```

```markdown
### Limitações dos Eventos de Efeito {/*limitations-of-effect-events*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Os Eventos de Efeito são muito limitados em como você pode usá-los:

*   **Chame-os apenas de dentro de Effects.**
*   **Nunca os passe para outros componentes ou Hooks.**

Por exemplo, não declare e passe um Evento de Efeito como este:

```js {4-6,8}
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 Evite: Passando Eventos de Efeito

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

Em vez disso, sempre declare Eventos de Efeito diretamente ao lado dos Effects que os usam:

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
  }, [delay]); // Não há necessidade de especificar "onTick" (um Evento de Efeito) como uma dependência
}
```

Eventos de Efeito são "pedaços" não reativos do código do seu Effect. Eles devem estar próximos ao Effect que os usa.

<Recap>

-   Manipuladores de eventos são executados em resposta a interações específicas.
-   Effects são executados sempre que a sincronização é necessária.
-   A lógica dentro dos manipuladores de eventos não é reativa.
-   A lógica dentro dos Effects é reativa.
-   Você pode mover a lógica não reativa dos Effects para os Eventos de Efeito.
-   Chame Eventos de Efeito apenas de dentro de Effects.
-   Não passe Eventos de Efeito para outros componentes ou Hooks.

</Recap>

<Challenges>

#### Corrija uma variável que não atualiza {/*fix-a-variable-that-doesnt-update*/}

Este componente `Timer` mantém uma variável de estado `count` que aumenta a cada segundo. O valor pelo qual ele está aumentando é armazenado na variável de estado `increment`. Você pode controlar a variável `increment` com os botões de mais e menos.

No entanto, não importa quantas vezes você clique no botão de mais, o contador ainda é incrementado em um a cada segundo. O que há de errado com este código? Por que `increment` é sempre igual a `1` dentro do código do Effect? Encontre o erro e corrija-o.

<Hint>

Para corrigir este código, basta seguir as regras.

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

Como de costume, quando você estiver procurando por erros em Effects, comece procurando por supressões de linter.

Se você remover o comentário de supressão, o React informará que o código deste Effect depende de `increment`, mas você "mentiu" para o React ao afirmar que este Effect não depende de nenhum valor reativo (`[]`). Adicione `increment` à matriz de dependência:

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

Agora, quando `increment` mudar, o React ressincronizará seu Effect, o que reiniciará o intervalo.

</Solution>

#### Corrija um contador congelante {/*fix-a-freezing-counter*/}

Este componente `Timer` mantém uma variável de estado `count` que aumenta a cada segundo. O valor pelo qual ele está aumentando é armazenado na variável de estado `increment`, que você pode controlar com os botões de mais e menos. Por exemplo, tente pressionar o botão de mais nove vezes e observe que o `count` agora aumenta a cada segundo em dez, em vez de um.

Há um pequeno problema com esta interface do usuário. Você pode notar que, se continuar pressionando os botões de mais ou menos mais rápido do que uma vez por segundo, o próprio temporizador parece pausar. Ele só é retomado após um segundo ter passado desde a última vez que você pressionou qualquer um dos botões. Descubra por que isso está acontecendo e corrija o problema para que o temporizador marque a cada segundo sem interrupções.

<Hint>

Parece que o Effect que configura o temporizador "reage" ao valor `increment`. A linha que usa o valor `increment` atual para chamar `setCount` realmente precisa ser reativa?

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

O problema é que o código dentro do Effect usa a variável de estado `increment`. Como é uma dependência do seu Effect, cada alteração em `increment` faz com que o Effect seja ressincronizado, o que faz com que o intervalo seja limpo. Se você continuar limpando o intervalo toda vez antes que ele tenha a chance de disparar, parecerá que o temporizador parou.

Para resolver o problema, extraia um Evento de Efeito `onTick` do Effect:

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

Como `onTick` é um Evento de Efeito, o código dentro dele não é reativo. A alteração em `increment` não aciona nenhum Effect.

</Solution>
```

#### Corrigir um atraso não ajustável {/*fix-a-non-adjustable-delay*/}

Neste exemplo, você pode personalizar o atraso do intervalo. Ele é armazenado em uma variável de estado `delay`, que é atualizada por dois botões. No entanto, mesmo que você pressione o botão "mais 100 ms" até que o `delay` seja de 1000 milissegundos (ou seja, um segundo), você notará que o cronômetro ainda incrementa muito rápido (a cada 100 ms). É como se suas alterações no `delay` fossem ignoradas. Encontre e corrija o erro.

<Hint>

O código dentro de Eventos de Efeito não é reativo. Existem casos em que você _deseja_ que a chamada `setInterval` seja executada novamente?

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

O problema com o exemplo acima é que ele extraiu um Evento de Efeito chamado `onMount` sem considerar o que o código realmente deveria estar fazendo. Você só deve extrair Eventos de Efeito por um motivo específico: quando você deseja tornar uma parte do seu código não reativa. No entanto, a chamada `setInterval` *deve* ser reativa em relação à variável de estado `delay`. Se o `delay` mudar, você deseja configurar o intervalo do zero! Para corrigir este código, coloque todo o código reativo de volta dentro do Effect:

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

Em geral, você deve desconfiar de funções como `onMount` que se concentram no *tempo* em vez do *propósito* de um trecho de código. Pode parecer "mais descritivo" a princípio, mas obscurece sua intenção. Como regra geral, os Eventos de Efeito devem corresponder a algo que acontece da perspectiva do *usuário*. Por exemplo, `onMessage`, `onTick`, `onVisit` ou `onConnected` são bons nomes de Eventos de Efeito. O código dentro deles provavelmente não precisaria ser reativo. Por outro lado, `onMount`, `onUpdate`, `onUnmount` ou `onAfterRender` são tão genéricos que é fácil colocar acidentalmente código que *deveria* ser reativo neles. É por isso que você deve nomear seus Eventos de Efeito após *o que o usuário acha que aconteceu*, e não quando algum código aconteceu de ser executado.

</Solution>


```markdown
#### Corrigir uma notificação atrasada {/*fix-a-delayed-notification*/}

Quando você entra em uma sala de bate-papo, este componente mostra uma notificação. No entanto, ele não mostra a notificação imediatamente. Em vez disso, a notificação é artificialmente atrasada em dois segundos para que o usuário tenha a chance de olhar ao redor da interface do usuário.

Isso quase funciona, mas há um erro. Tente alterar o menu suspenso de "geral" para "viagem" e, em seguida, para "música" muito rapidamente. Se você fizer isso rápido o suficiente, verá duas notificações (como esperado!), mas ambas dirão "Bem-vindo à música".

Corrija-o para que, ao mudar de "geral" para "viagem" e, em seguida, para "música" muito rapidamente, você veja duas notificações, a primeira sendo "Bem-vindo à viagem" e a segunda sendo "Bem-vindo à música". (Para um desafio adicional, supondo que você *já* tenha feito as notificações mostrarem as salas corretas, altere o código para que apenas a última notificação seja exibida.)

<Hint>

Seu Effect sabe a qual sala ele se conectou. Há alguma informação que você pode querer passar para seu Effect Event?

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

Dentro do seu Effect Event, `roomId` é o valor *no momento em que o Effect Event foi chamado.*

Seu Effect Event é chamado com um atraso de dois segundos. Se você estiver mudando rapidamente da sala de viagem para a sala de música, quando a notificação da sala de viagem aparecer, `roomId` já será `"music"`. É por isso que ambas as notificações dizem "Bem-vindo à música".

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

Isso garante que as notificações já agendadas (mas ainda não exibidas) sejam canceladas quando você muda de sala.

</Solution>

</Challenges>
```