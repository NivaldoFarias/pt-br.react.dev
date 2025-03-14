---
title: 'Ciclo de Vida de Efeitos Reativos'
---

<Intro>

Effects têm um ciclo de vida diferente dos componentes. Componentes podem montar, atualizar ou desmontar. Um Effect pode fazer apenas duas coisas: começar a sincronizar algo e, mais tarde, parar de sincronizá-lo. Este ciclo pode acontecer várias vezes se o seu Effect depender das props e state que mudam com o tempo. React fornece uma regra de linter para verificar se você especificou as dependências do seu Effect corretamente. Isso mantém seu Effect sincronizado com as props e o state mais recentes.

</Intro>

<YouWillLearn>

- Como o ciclo de vida de um Effect é diferente do ciclo de vida de um componente
- Como pensar em cada Effect individualmente, de forma isolada
- Quando seu Effect precisa ser re-sincronizado e por quê
- Como as dependências do seu Effect são determinadas
- O que significa um valor ser reativo
- O que uma array de dependência vazia significa
- Como o React verifica se suas dependências estão corretas com um linter
- O que fazer quando você discorda do linter

</YouWillLearn>

## O ciclo de vida de um Effect {/*the-lifecycle-of-an-effect*/}

Todo componente React passa pelo mesmo ciclo de vida:

- Um componente _monta_ quando ele é adicionado à tela.
- Um componente _atualiza_ quando recebe novas props ou state, geralmente em resposta a uma interação.
- Um componente _desmonta_ quando é removido da tela.

**É uma boa maneira de pensar sobre componentes, mas _não_ sobre Effects.** Em vez disso, tente pensar em cada Effect independentemente do ciclo de vida do seu componente. Um Effect descreve como [sincronizar um sistema externo](/learn/synchronizing-with-effects) com as props e o state atuais. À medida que seu código muda, a sincronização precisará acontecer com mais ou menos frequência.

Para ilustrar este ponto, considere este Effect conectando seu componente a um servidor de chat:

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
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

O corpo do seu Effect especifica como **começar a sincronizar:**

```js {2-3}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

A função de cleanup retornada pelo seu Effect especifica como **parar de sincronizar:**

```js {5}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

Intuitivamente, você pode pensar que o React **começaria a sincronizar** quando seu componente montar e **pararia de sincronizar** quando seu componente desmontar. No entanto, esta não é a conclusão da história! Às vezes, também pode ser necessário **começar e parar de sincronizar várias vezes** enquanto o componente permanece montado.

Vamos ver _por que_ isso é necessário, _quando_ isso acontece e _como_ você pode controlar esse comportamento.

<Note>

Alguns Effects não retornam uma função de cleanup. [Na maioria das vezes,](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) você vai querer retornar uma — mas se você não fizer isso, React se comportará como se você tivesse retornado uma função de cleanup vazia.

</Note>

### Por que a sincronização pode precisar acontecer mais de uma vez {/*why-synchronization-may-need-to-happen-more-than-once*/}

Imagine que este componente `ChatRoom` recebe uma `roomId` prop que o usuário escolhe em um dropdown. Digamos que, inicialmente, o usuário escolha a sala `"general"` como `roomId`. Seu aplicativo exibe a sala de chat `"general"`:

```js {3}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

Depois que a UI for exibida, o React executará seu Effect para **começar a sincronizar.** Ele se conecta à sala `"general"`:

```js {3,4}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta à sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Desconecta da sala "general"
    };
  }, [roomId]);
  // ...
```

Até agora, tudo bem.

Mais tarde, o usuário escolhe uma sala diferente no dropdown (por exemplo, `"travel"`). Primeiro, o React atualizará a UI:

```js {1}
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

Pense no que deve acontecer a seguir. O usuário vê que `"travel"` é a sala de chat selecionada na UI. No entanto, o Effect que foi executado na última vez ainda está conectado à sala `"general"`. **A prop `roomId` mudou, então o que seu Effect fez naquela época (conectando-se à sala `"general"`) não corresponde mais à UI.**

Neste ponto, você quer que o React faça duas coisas:

1. Parar de sincronizar com o `roomId` antigo (desconectar da sala `"general"`)
2. Começar a sincronizar com o novo `roomId` (conectar-se à sala `"travel"`)

**Felizmente, você já ensinou ao React como fazer as duas coisas!** O corpo do seu Effect especifica como começar a sincronizar e sua função de cleanup especifica como parar de sincronizar. Tudo o que o React precisa fazer agora é chamá-los na ordem correta e com as props e o state corretos. Vamos ver como isso acontece exatamente.

### Como o React re-sincroniza seu Effect {/*how-react-re-synchronizes-your-effect*/}

Lembre-se de que seu componente `ChatRoom` recebeu um novo valor para sua prop `roomId`. Ele costumava ser `"general"` e agora é `"travel"`. O React precisa re-sincronizar seu Effect para reconectá-lo a uma sala diferente.

Para **parar de sincronizar**, o React chamará a função de cleanup que seu Effect retornou após se conectar à sala `"general"`. Como `roomId` era `"general"`, a função de cleanup desconecta da sala `"general"`:

```js {6}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta à sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Desconecta da sala "general"
    };
    // ...
```

Então o React executará o Effect que você forneceu durante esta renderização. Desta vez, `roomId` é `"travel"`, então ele **começará a sincronizar** com a sala de chat `"travel"` (até que sua função de cleanup seja eventualmente chamada também):

```js {3,4}
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta à sala "travel"
    connection.connect();
    // ...
```

Graças a isso, você agora está conectado à mesma sala que o usuário escolheu na UI. Desastre evitado!

Toda vez que seu componente re-renderizar com um `roomId` diferente, seu Effect re-sincronizará. Por exemplo, digamos que o usuário muda `roomId` de `"travel"` para `"music"`. O React irá novamente **parar de sincronizar** seu Effect chamando sua função de cleanup (desconectando você da sala `"travel"`). Então, ele **começará a sincronizar** novamente executando seu corpo com a nova prop `roomId` (conectando você à sala `"music"`).

Finalmente, quando o usuário for para uma tela diferente, `ChatRoom` desmontará. Agora não há necessidade de ficar conectado. O React **parará de sincronizar** seu Effect pela última vez e desconectará você da sala de chat `"music"`.

### Pensando da perspectiva do Effect {/*thinking-from-the-effects-perspective*/}

Vamos recapitular tudo o que aconteceu da perspectiva do componente `ChatRoom`:

1. `ChatRoom` montou com `roomId` definido como `"general"`
1. `ChatRoom` atualizou com `roomId` definido como `"travel"`
1. `ChatRoom` atualizou com `roomId` definido como `"music"`
1. `ChatRoom` desmontou

Durante cada um desses pontos no ciclo de vida do componente, seu Effect fez coisas diferentes:

1. Seu Effect conectou-se à sala `"general"`
1. Seu Effect desconectou-se da sala `"general"` e conectou-se à sala `"travel"`
1. Seu Effect desconectou-se da sala `"travel"` e conectou-se à sala `"music"`
1. Seu Effect desconectou-se da sala `"music"`

Agora, vamos pensar sobre o que aconteceu da perspectiva do próprio Effect:

```js
  useEffect(() => {
    // Seu Effect conectou à sala especificada com roomId...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...até desconectar
      connection.disconnect();
    };
  }, [roomId]);
```

A estrutura deste código pode inspirá-lo a ver o que aconteceu como uma sequência de períodos de tempo não sobrepostos:

1. Seu Effect conectou-se à sala `"general"` (até desconectar)
1. Seu Effect conectou-se à sala `"travel"` (até desconectar)
1. Seu Effect conectou-se à sala `"music"` (até desconectar)

Anteriormente, você estava pensando da perspectiva do componente. Quando você olhava da perspectiva do componente, era tentador pensar em Effects como "callbacks" ou "eventos do ciclo de vida" que disparam em um momento específico, como "após uma renderização" ou "antes de desmontar". Essa maneira de pensar fica complicada muito rápido, por isso é melhor evitar.

**Em vez disso, sempre se concentre em um único ciclo de início/parada por vez. Não deveria importar se um componente está montando, atualizando ou desmontando. Tudo o que você precisa fazer é descrever como começar a sincronização e como pará-la. Se você fizer isso bem, seu Effect será resistente a ser iniciado e parado quantas vezes forem necessárias.**

Isso pode lembrá-lo de como você não pensa se um componente está montando ou atualizando ao escrever a lógica de renderização que cria JSX. Você descreve o que deve estar na tela, e o React [descobre o resto.](/learn/reacting-to-input-with-state)

### Como o React verifica se seu Effect pode re-sincronizar {/*how-react-verifies-that-your-effect-can-re-synchronize*/}

Aqui está um exemplo ao vivo com o qual você pode brincar. Pressione "Open chat" para montar o componente `ChatRoom`:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>Bem-vindo à sala {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
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
        {show ? 'Fechar chat' : 'Abrir chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Uma implementação real realmente conectaria ao servidor
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Observe que, quando o componente monta pela primeira vez, você vê três logs:

1. `✅ Conectando à sala "general" em https://localhost:1234...` *(apenas para desenvolvimento)*
1. `❌ Desconectado da sala "general" em https://localhost:1234.` *(somente para desenvolvimento)*
1. `✅ Conectando à sala "general" em https://localhost:1234...`

Os dois primeiros logs são apenas para desenvolvimento. No desenvolvimento, o React sempre remonta cada componente uma vez.

**O React verifica se seu Effect pode re-sincronizar, forçando-o a fazer isso imediatamente no desenvolvimento.** Isso pode lembrá-lo de abrir uma porta e fechá-la uma vez extra para verificar se a fechadura da porta funciona. O React inicia e interrompe seu Effect uma vez extra no desenvolvimento para verificar [se você implementou bem sua limpeza.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

A principal razão pela qual seu Effect será re-sincronizado na prática é se alguns dados que ele usa mudarem. No sandbox acima, altere a sala de chat selecionada. Observe como, quando o `roomId` muda, seu Effect re-sincroniza.

No entanto, também existem casos mais incomuns em que a re-sincronização é necessária. Por exemplo, tente editar o `serverUrl` no sandbox acima enquanto o chat está aberto. Observe como o Effect re-sincroniza em resposta às suas edições no código. No futuro, o React pode adicionar mais recursos que dependem de re-sincronização.

### Como o React sabe que precisa re-sincronizar o Effect {/*how-react-knows-that-it-needs-to-re-synchronize-the-effect*/}

Você pode estar se perguntando como o React sabia que seu Effect precisava re-sincronizar depois que `roomId` muda. Isso porque *você disse ao React* que seu código depende de `roomId` incluindo-o na [lista de dependências:](/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)

```js {1,3,8}
function ChatRoom({ roomId }) { // A prop roomId pode mudar com o tempo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Este Effect lê roomId 
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Então você diz ao React que este Effect "depende de" roomId
  // ...
```

Veja como isso funciona:

1. Você sabia que `roomId` é uma prop, o que significa que pode mudar com o tempo.
2. Você sabia que seu Effect lê `roomId` (então sua lógica depende de um valor que pode mudar mais tarde).
3. É por isso que você o especificou como a dependência do seu Effect (para que ele re-sincronize quando `roomId` muda).

Toda vez que seu componente re-renderiza, o React examinará a array de dependências que você passou. Se algum dos valores na array for diferente do valor no mesmo local que você passou durante a renderização anterior, o React re-sincronizará seu Effect.

Por exemplo, se você passou `["general"]` durante a renderização inicial e, mais tarde, passou `["travel"]` durante a próxima renderização, o React comparará `"general"` e `"travel"`. Estes são valores diferentes (comparados com [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), então o React re-sincronizará seu Effect. Por outro lado, se seu componente re-renderizar, mas `roomId` não tiver mudado, seu Effect permanecerá conectado à mesma sala.

### Cada Effect representa um processo de sincronização separado {/*each-effect-represents-a-separate-synchronization-process*/}

Resista a adicionar lógica não relacionada ao seu Effect apenas porque essa lógica precisa ser executada ao mesmo tempo que um Effect que você já escreveu. Por exemplo, digamos que você queira enviar um evento de analytics quando o usuário visitar a sala. Você já tem um Effect que depende de `roomId`, então você pode se sentir tentado a adicionar a chamada de analytics lá:

```js {3}
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Mas imagine que você adicione posteriormente outra dependência a este Effect que precisa restabelecer a conexão. Se este Effect re-sincronizar, ele também chamará `logVisit(roomId)` para a mesma sala, o que você não pretendia. Registrar a visita **é um processo separado** da conexão. Escreva-os como dois Effects separados:

```js {2-4}
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

**Cada Effect em seu código deve representar um processo de sincronização separado e independente.**

No exemplo acima, excluir um Effect não interromperia a lógica do outro Effect. Esta é uma boa indicação de que eles sincronizam coisas diferentes e, portanto, fazia sentido dividi-las. Por outro lado, se você dividir uma parte coesa da lógica em Effects separados, o código pode parecer "mais limpo", mas será [mais difícil de manter.](/learn/you-might-not-need-an-effect#chains-of-computations) É por isso que você deve pensar se os processos são os mesmos ou separados, não se o código parece mais limpo.

## Effects "react" para valores reativos {/*effects-react-to-reactive-values*/}

Seu Effect lê duas variáveis (`serverUrl` e `roomId`), mas você só especificou `roomId` como uma dependência:

```js {5,10}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
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

Por que `serverUrl` não precisa ser uma dependência?

Isso ocorre porque o `serverUrl` nunca muda devido a uma re-renderização. É sempre o mesmo, não importa quantas vezes o componente re-renderize e por quê. Como `serverUrl` nunca muda, não faria sentido especificá-lo como uma dependência. Afinal, as dependências só fazem alguma coisa quando mudam com o tempo!

Por outro lado, `roomId` pode ser diferente em uma re-renderização. **Props, state e outros valores declarados dentro do componente são _reativos_ porque são calculados durante a renderização e participam do fluxo de dados do React.**

Se `serverUrl` fosse uma variável de state, ela seria reativa. Valores reativos devem ser incluídos em dependências:
``````js {2,5,10}
function ChatRoom({ roomId }) { // Props mudam com o tempo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // State pode mudar com o tempo

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Seu Effect lê props e state
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // Então você diz ao React que este Effect "depende de" props e state
  // ...
}
```

Ao incluir `serverUrl` como uma dependência, você garante que o Effect re-sincronize após sua alteração.

Tente alterar a sala de bate-papo selecionada ou editar a URL do servidor neste sandbox:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bem-vindo à sala {roomId}!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Sempre que você altera um valor reativo como `roomId` ou `serverUrl`, o Effect se reconecta ao servidor de bate-papo.

### O que significa um Effect com dependências vazias {/*what-an-effect-with-empty-dependencies-means*/}

O que acontece se você mover `serverUrl` e `roomId` para fora do componente?

```js {1,2}
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Todas as dependências declaradas
  // ...
}
```

Agora, o código do seu Effect não usa *nenhum* valor reativo, então suas dependências podem estar vazias (`[]`).

Pensando na perspectiva do componente, a matriz de dependência vazia `[]` significa que este Effect se conecta à sala de bate-papo somente quando o componente monta e se desconecta somente quando o componente desmonta. (Lembre-se de que o React ainda [vai re-sincronizá-lo uma vez](#how-react-verifies-that-your-effect-can-re-synchronize) em desenvolvimento para testar a sua lógica.)


<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar chat' : 'Abrir chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

```js src/chat.js
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

No entanto, se você [pensar na perspectiva do Effect,](#thinking-from-the-effects-perspective) não precisa pensar em montar e desmontar. O que é importante é que você especificou o que seu Effect faz para começar e parar a sincronização. Hoje, ele não tem dependências reativas. Mas se você quiser que o usuário altere `roomId` ou `serverUrl` com o tempo (e eles se tornariam reativos), o código do seu Effect não mudará. Você só precisará adicioná-los às dependências.

### Todas as variáveis declaradas no corpo do componente são reativas {/*all-variables-declared-in-the-component-body-are-reactive*/}

Props e state não são os únicos valores reativos. Valores que você calcula a partir deles também são reativos. Se as props ou state forem alterados, seu componente será renderizado novamente e os valores calculados a partir deles também serão alterados. É por isso que todas as variáveis do corpo do componente usadas pelo Effect devem estar na lista de dependências do Effect.

Digamos que o usuário possa escolher um servidor de bate-papo na lista suspensa, mas também pode configurar um servidor padrão nas configurações. Suponha que você já tenha colocado o estado das configurações em um [contexto](/learn/scaling-up-with-reducer-and-context) para que você leia as `settings` desse contexto. Agora, você calcula a `serverUrl` com base no servidor selecionado nas props e no servidor padrão:

```js {3,5,10}
function ChatRoom({ roomId, selectedServerUrl }) { // roomId é reativo
  const settings = useContext(SettingsContext); // settings é reativo
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl é reativo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Your Effect reads roomId and serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // Então ele precisa re-sincronizar quando qualquer um deles mudar!
  // ...
}
```

Neste exemplo, `serverUrl` não é uma prop ou uma variável de estado. É uma variável normal que você calcula durante a renderização. Mas ela é calculada durante a renderização, então pode mudar devido a uma nova renderização. É por isso que ela é reativa.

**Todos os valores dentro do componente (incluindo props, state e variáveis no corpo do seu componente) são reativos. Qualquer valor reativo pode mudar em uma nova renderização, então você precisa incluir valores reativos como dependências do Effect.**

Em outras palavras, Effects "reagem" a todos os valores do corpo do componente.

<DeepDive>

#### Valores globais ou mutáveis podem ser dependências? {/*can-global-or-mutable-values-be-dependencies*/}

Valores mutáveis (incluindo variáveis globais) não são reativos.

**Um valor mutável como [`location.pathname`](https://developer.mozilla.org/en-US/docs/Web/API/Location/pathname) não pode ser uma dependência.** Ele é mutável, então pode mudar a qualquer momento completamente fora do fluxo de dados de renderização do React. Alterá-lo não acionaria uma nova renderização do seu componente. Portanto, mesmo que você o especificasse nas dependências, o React *não saberia* para re-sincronizar o Effect quando ele mudar. Isso também quebra as regras do React porque ler dados mutáveis durante a renderização (que é quando você calcula as dependências) quebra a [pureza da renderização.](/learn/keeping-components-pure) Em vez disso, você deve ler e se inscrever em um valor mutável externo com [`useSyncExternalStore`.](/learn/you-might-not-need-an-effect#subscribing-to-an-external-store)

**Um valor mutável como [`ref.current`](/reference/react/useRef#reference) ou coisas que você lê dele também não pode ser uma dependência.** O objeto ref retornado por `useRef` em si pode ser uma dependência, mas sua propriedade `current` é intencionalmente mutável. Ele permite que você [acompanhe algo sem acionar uma nova renderização.](/learn/referencing-values-with-refs) Mas como alterá-lo não aciona uma nova renderização, ele não é um valor reativo e o React não saberá para executar seu Effect novamente quando ele mudar.

Como você aprenderá abaixo nesta página, um linter verificará esses problemas automaticamente.

</DeepDive>

### O React verifica se você especificou cada valor reativo como uma dependência {/*react-verifies-that-you-specified-every-reactive-value-as-a-dependency*/}

Se o seu linter estiver [configurado para o React,](/learn/editor-setup#linting) ele verificará se todos os valores reativos usados pelo código do seu Effect são declarados como sua dependência. Por exemplo, este é um erro de lint porque `roomId` e `serverUrl` são reativos:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) { // roomId é reativo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl é reativo

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Algo está errado aqui!

  return (
    <>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bem-vindo(a) à sala {roomId}!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Isso pode parecer um erro do React, mas na verdade o React está apontando um bug no seu código. Tanto `roomId` quanto `serverUrl` podem mudar com o tempo, mas você está esquecendo de re-sincronizar seu Effect quando eles mudam. Você permanecerá conectado ao `roomId` e `serverUrl` iniciais, mesmo depois que o usuário escolher valores diferentes na UI.

Para corrigir o bug, siga a sugestão do linter para especificar `roomId` e `serverUrl` como dependências do seu Effect:

```js {9}
function ChatRoom({ roomId }) { // roomId é reativo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl é reativo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ Todas as dependências declaradas
  // ...
}
```

Tente essa correção no sandbox acima. Verifique se o erro do linter desapareceu e se o chat se reconecta quando necessário.

<Note>

Em alguns casos, o React *sabe* que um valor nunca muda, mesmo que seja declarado dentro do componente. Por exemplo, a função [`set` function](/reference/react/useState#setstate) retornada de `useState` e o objeto ref retornado por [`useRef`](/reference/react/useRef) são *estáveis* -- eles são garantidos que não mudarão em uma nova renderização. Valores estáveis não são reativos, então você pode omiti-los da lista. Incluí-los é permitido: eles não mudarão, então não importa.

</Note>

### O que fazer quando você não quer re-sincronizar {/*what-to-do-when-you-dont-want-to-re-synchronize*/}

No exemplo anterior, você corrigiu o erro de lint listando `roomId` e `serverUrl` como dependências.

**No entanto, você pode, em vez disso, "provar" ao linter que esses valores não são valores reativos,** ou seja, que eles *não podem* ser alterados como resultado de uma nova renderização. Por exemplo, se `serverUrl` e `roomId` não dependem da renderização e sempre têm os mesmos valores, você pode movê-los para fora do componente. Agora, eles não precisam ser dependências:

```js {1,2,11}
const serverUrl = 'https://localhost:1234'; // serverUrl não é reativo
const roomId = 'general'; // roomId não é reativo

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Todas as dependências declaradas
  // ...
}
```

Você também pode movê-los *dentro do Effect.* Eles não são calculados durante a renderização, então não são reativos:

```js {3,4,10}
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl não é reativo
    const roomId = 'general'; // roomId não é reativo
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Todas as dependências declaradas
  // ...
}
```

**Effects são blocos de código reativos.** Eles re-sincronizam quando os valores que você lê dentro deles mudam. Ao contrário dos manipuladores de eventos, que são executados apenas uma vez por interação, os Effects são executados sempre que a sincronização é necessária.

**Você não pode "escolher" suas dependências.** Suas dependências devem incluir todos os [valores reativos](#all-variables-declared-in-the-component-body-are-reactive) que você lê no Effect. O linter impõe isso. Às vezes, isso pode levar a problemas como loops infinitos e o seu Effect re-sincronizando com muita frequência. Não conserte esses problemas suprimindo o linter! Aqui está o que tentar em vez disso:

*   **Verifique se seu Effect representa um processo de sincronização independente.** Se seu Effect não sincronizar nada, [pode ser desnecessário.](/learn/you-might-not-need-an-effect) Se ele sincroniza várias coisas independentes, [divida-o.](#each-effect-represents-a-separate-synchronization-process)

*   **Se você deseja ler o valor mais recente de props ou state sem "reagir" a ele e re-sincronizar o Effect,** você pode dividir seu Effect em uma parte reativa (que você manterá no Effect) e uma parte não reativa (que você extrairá em algo chamado de _Effect Event_). [Leia sobre como separar Events de Effects.](/learn/separating-events-from-effects)

*   **Evite depender de objetos e funções como dependências.** Se você criar objetos e funções durante a renderização e, em seguida, lê-los de um Effect, eles serão diferentes em cada renderização. Isso fará com que seu Effect re-sincronize toda vez. [Leia mais sobre como remover dependências desnecessárias de Effects.](/learn/removing-effect-dependencies)

<Pitfall>

O linter é seu amigo, mas seus poderes são limitados. O linter só sabe quando as dependências estão *erradas*. Ele não sabe *a melhor* forma de resolver cada caso. Se o linter sugere uma dependência, mas adicioná-la causa um loop, isso não significa que o linter deva ser ignorado. Você precisa alterar o código dentro (ou fora) do Effect para que esse valor não seja reativo e não *precise* ser uma dependência.

Se você tiver uma base de código existente, poderá ter alguns Effects que suprimem o linter desta forma:

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 Evite suprimir o linter dessa forma:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

Nas [próximas](/learn/separating-events-from-effects) [páginas](/learn/removing-effect-dependencies), você aprenderá a corrigir esse código sem quebrar as regras. Sempre vale a pena corrigir!

</Pitfall>

<Recap>

-   Componentes podem montar, atualizar e desmontar.
-   Cada Effect tem um ciclo de vida separado do componente circundante.
-   Cada Effect descreve um processo de sincronização separado que pode *iniciar* e *parar*.
-   Ao escrever e ler Effects, pense na perspectiva de cada Effect individual (como iniciar e parar a sincronização) em vez da perspectiva do componente (como ele monta, atualiza ou desmonta).
-   Valores declarados dentro do corpo do componente são "reativos".
-   Valores reativos devem re-sincronizar o Effect porque podem mudar com o tempo.
-   O linter verifica se todos os valores reativos usados dentro do Effect são especificados como dependências.
-   Todos os erros sinalizados pelo linter são legítimos. Sempre há uma maneira de corrigir o código para não quebrar as regras.

</Recap>

<Challenges>

#### Corrigir a reconexão em cada pressionamento de tecla {/*fix-reconnecting-on-every-keystroke*/}

Neste exemplo, o componente `ChatRoom` se conecta à sala de bate-papo quando o componente monta, desconecta quando desmonta e se reconecta quando você seleciona uma sala de bate-papo diferente. Esse comportamento está correto, então você precisa mantê-lo funcionando.

No entanto, há um problema. Sempre que você digita na caixa de mensagem de entrada na parte inferior, `ChatRoom` *também* se reconecta ao chat. (Você pode notar isso limpando o console e digitando na entrada.) Corrija o problema para que isso não aconteça.

<Hint>

Você pode precisar adicionar uma matriz de dependências para este Effect. Quais dependências devem estar lá?

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  });

  return (
    <>
      <h1>Bem-vindo(a) à sala {roomId}!</h1>
      <input
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
    </>
  );
}```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Bem-vindo(a) à sala {roomId}!</h1>
      <input
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```
