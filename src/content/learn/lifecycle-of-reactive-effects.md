---
title: 'Ciclo de Vida de Efeitos Reativos'
---

<Intro>

Efeitos têm um ciclo de vida diferente dos componentes. Componentes podem montar, atualizar ou desmontar. Um Efeito só pode fazer duas coisas: começar a sincronizar algo e, mais tarde, parar de sincronizá-lo. Este ciclo pode acontecer várias vezes se o seu Efeito depender de props e estado que mudam ao longo do tempo. O React fornece uma regra de linter para verificar se você especificou as dependências do seu Efeito corretamente. Isso mantém seu Efeito sincronizado com as últimas props e estado.

</Intro>

<YouWillLearn>

- Como o ciclo de vida de um Efeito é diferente do ciclo de vida de um componente
- Como pensar em cada Efeito individualmente, isoladamente
- Quando seu Efeito precisa ressincronizar e por quê
- Como as dependências do seu Efeito são determinadas
- O que significa um valor ser reativo
- O que significa um array de dependências vazio
- Como o React verifica se suas dependências estão corretas com um linter
- O que fazer quando você discorda do linter

</YouWillLearn>

## O ciclo de vida de um Effect {/*o-ciclo-de-vida-de-um-effect*/}

Todo componente React passa pelo mesmo ciclo de vida:

- Um componente _monta_ quando é adicionado à tela.
- Um componente _atualiza_ quando recebe novas props ou estado, geralmente em resposta a uma interação.
- Um componente _desmonta_ quando é removido da tela.

**É uma boa maneira de pensar sobre componentes, mas _não_ sobre Effects.** Em vez disso, tente pensar em cada Effect independentemente do ciclo de vida do seu componente. Um Effect descreve como [sincronizar um sistema externo](/learn/synchronizing-with-effects) com as props e o estado atuais. À medida que seu código muda, a sincronização precisará acontecer com mais ou menos frequência.

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

O corpo do seu Effect especifica como **iniciar a sincronização:**

```js {2-3}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

A função de limpeza retornada pelo seu Effect especifica como **parar a sincronização:**

```js {5}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

Intuitivamente, você pode pensar que o React **iniciará a sincronização** quando seu componente montar e **parará a sincronização** quando seu componente desmontar. No entanto, este não é o fim da história! Às vezes, também pode ser necessário **iniciar e parar a sincronização várias vezes** enquanto o componente permanece montado.

Vamos ver _por que_ isso é necessário, _quando_ isso acontece e _como_ você pode controlar esse comportamento.

<Note>

Alguns Effects não retornam uma função de limpeza. [Na maioria das vezes,](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) você vai querer retornar uma -- mas se não o fizer, o React se comportará como se você tivesse retornado uma função de limpeza vazia.

</Note>

### Por que a sincronização pode precisar acontecer mais de uma vez {/*por-que-a-sincronizacao-pode-precisar-acontecer-mais-de-uma-vez*/}

Imagine que este componente `ChatRoom` recebe uma prop `roomId` que o usuário escolhe em um dropdown. Vamos dizer que inicialmente o usuário escolhe a sala `"general"` como `roomId`. Seu aplicativo exibe a sala de chat `"general"`:

```js {3}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

Após a exibição da UI, o React executará seu Effect para **iniciar a sincronização.** Ele se conecta à sala `"general"`:

```js {3,4}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "general"
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
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

Pense sobre o que deve acontecer a seguir. O usuário vê que `"travel"` é a sala de chat selecionada na UI. No entanto, o Effect que foi executado da última vez ainda está conectado à sala `"general"`. **A prop `roomId` mudou, então o que seu Effect fez naquela época (conectar-se à sala `"general"`) não corresponde mais à UI.**

Neste ponto, você quer que o React faça duas coisas:

1. Pare de sincronizar com o `roomId` antigo (desconecte-se da sala `"general"`)
2. Comece a sincronizar com o novo `roomId` (conecte-se à sala `"travel"`)

**Felizmente, você já ensinou ao React como fazer ambas as coisas!** O corpo do seu Effect especifica como iniciar a sincronização, e sua função de limpeza especifica como parar a sincronização. Tudo o que o React precisa fazer agora é chamá-los na ordem correta e com as props e o estado corretos. Vamos ver exatamente como isso acontece.

### Como o React resincroniza seu Effect {/*como-o-react-resincroniza-seu-effect*/}

Lembre-se que seu componente `ChatRoom` recebeu um novo valor para sua prop `roomId`. Era `"general"` e agora é `"travel"`. O React precisa resincronizar seu Effect para reconectá-lo a uma sala diferente.

Para **parar a sincronização,** o React chamará a função de limpeza que seu Effect retornou após conectar-se à sala `"general"`. Como `roomId` era `"general"`, a função de limpeza desconecta da sala `"general"`:

```js {6}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Desconecta da sala "general"
    };
    // ...
```

Em seguida, o React executará o Effect que você forneceu durante esta renderização. Desta vez, `roomId` é `"travel"`, então ele **iniciará a sincronização** com a sala de chat `"travel"` (até que sua função de limpeza seja eventualmente chamada também):

```js {3,4}
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "travel"
    connection.connect();
    // ...
```

Graças a isso, você agora está conectado à mesma sala que o usuário escolheu na UI. Desastre evitado!

Toda vez que seu componente renderizar novamente com um `roomId` diferente, seu Effect resincronizará. Por exemplo, digamos que o usuário mude `roomId` de `"travel"` para `"music"`. O React novamente **parará a sincronização** do seu Effect chamando sua função de limpeza (desconectando você da sala `"travel"`). Em seguida, ele **iniciará a sincronização** novamente executando seu corpo com a nova prop `roomId` (conectando você à sala `"music"`).

Finalmente, quando o usuário for para uma tela diferente, `ChatRoom` desmontará. Agora não há necessidade de permanecer conectado. O React **parará a sincronização** do seu Effect uma última vez e desconectará você da sala de chat `"music"`.

### Pensando da perspectiva do Effect {/*pensando-da-perspectiva-do-effect*/}

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

Agora vamos pensar sobre o que aconteceu da perspectiva do próprio Effect:

```js
  useEffect(() => {
    // Seu Effect conectou-se à sala especificada com roomId...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...até que se desconectou
      connection.disconnect();
    };
  }, [roomId]);
```

A estrutura deste código pode inspirá-lo a ver o que aconteceu como uma sequência de períodos de tempo não sobrepostos:

1. Seu Effect conectou-se à sala `"general"` (até que se desconectou)
1. Seu Effect conectou-se à sala `"travel"` (até que se desconectou)
1. Seu Effect conectou-se à sala `"music"` (até que se desconectou)

Anteriormente, você estava pensando da perspectiva do componente. Quando você olhava da perspectiva do componente, era tentador pensar em Effects como "callbacks" ou "eventos de ciclo de vida" que disparam em um momento específico como "após uma renderização" ou "antes de desmontar". Essa forma de pensar fica complicada muito rapidamente, então é melhor evitá-la.

**Em vez disso, concentre-se sempre em um único ciclo de início/parada de cada vez. Não deve importar se um componente está montando, atualizando ou desmontando. Tudo o que você precisa fazer é descrever como iniciar a sincronização e como pará-la. Se você fizer isso bem, seu Effect será resiliente a ser iniciado e parado quantas vezes forem necessárias.**

Isso pode lembrá-lo de como você não pensa se um componente está montando ou atualizando ao escrever a lógica de renderização que cria JSX. Você descreve o que deve estar na tela, e o React [descobre o resto.](/learn/reacting-to-input-with-state)

### Como o React verifica que seu Effect pode resincronizar {/*como-o-react-verifica-que-seu-effect-pode-resincronizar*/}

Aqui está um exemplo prático com o qual você pode interagir. Pressione "Open chat" para montar o componente `ChatRoom`:

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
  return <h1>Welcome to the {roomId} room!</h1>;
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Note que quando o componente monta pela primeira vez, você vê três logs:

1. `✅ Connecting to "general" room at https://localhost:1234...` *(apenas em desenvolvimento)*
1. `❌ Disconnected from "general" room at https://localhost:1234.` *(apenas em desenvolvimento)*
1. `✅ Connecting to "general" room at https://localhost:1234...`

Os dois primeiros logs são apenas para desenvolvimento. Em desenvolvimento, o React sempre remonta cada componente uma vez.

**O React verifica que seu Effect pode resincronizar forçando-o a fazer isso imediatamente em desenvolvimento.** Isso pode lembrá-lo de abrir uma porta e fechá-la uma vez extra para verificar se a trava da porta funciona. O React inicia e para seu Effect uma vez extra em desenvolvimento para verificar se [você implementou bem sua limpeza.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

A principal razão pela qual seu Effect resincronizará na prática é se alguns dados que ele usa mudaram. Na sandbox acima, altere a sala de chat selecionada. Observe como, quando o `roomId` muda, seu Effect resincroniza.

No entanto, também existem casos mais incomuns em que a resincronização é necessária. Por exemplo, tente editar o `serverUrl` na sandbox acima enquanto o chat está aberto. Observe como o Effect resincroniza em resposta às suas edições no código. No futuro, o React pode adicionar mais recursos que dependem da resincronização.

### Como o React sabe que precisa resincronizar o Effect {/*como-o-react-sabe-que-precisa-resincronizar-o-effect*/}

Você pode estar se perguntando como o React soube que seu Effect precisava resincronizar após a mudança de `roomId`. É porque *você disse ao React* que seu código depende de `roomId` incluindo-o na [lista de dependências:](/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)

```js {1,3,8}
function ChatRoom({ roomId }) { // A prop roomId pode mudar ao longo do tempo
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

1. Você sabia que `roomId` é uma prop, o que significa que ela pode mudar ao longo do tempo.
2. Você sabia que seu Effect lê `roomId` (portanto, sua lógica depende de um valor que pode mudar mais tarde).
3. É por isso que você o especificou como a dependência do seu Effect (para que ele resincronize quando `roomId` mudar).

Toda vez após seu componente renderizar novamente, o React olhará para a matriz de dependências que você passou. Se algum dos valores na matriz for diferente do valor no mesmo local que você passou durante a renderização anterior, o React resincronizará seu Effect.

Por exemplo, se você passou `["general"]` durante a renderização inicial e, mais tarde, passou `["travel"]` durante a próxima renderização, o React comparará `"general"` e `"travel"`. Estes são valores diferentes (comparados com [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), então o React resincronizará seu Effect. Por outro lado, se seu componente renderizar novamente, mas `roomId` não mudou, seu Effect permanecerá conectado à mesma sala.

### Cada Effect representa um processo de sincronização separado {/*each-effect-represents-a-separate-synchronization-process*/}

Resista à tentação de adicionar lógica não relacionada ao seu Effect apenas porque essa lógica precisa ser executada ao mesmo tempo que um Effect que você já escreveu. Por exemplo, digamos que você queira enviar um evento de análise quando o usuário visita a sala. Você já tem um Effect que depende de `roomId`, então pode se sentir tentado a adicionar a chamada de análise lá:

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

Mas imagine que você adicione mais tarde outra dependência a este Effect que precise restabelecer a conexão. Se este Effect resincronizar, ele também chamará `logVisit(roomId)` para a mesma sala, o que você não pretendia. Registrar a visita **é um processo separado** da conexão. Escreva-os como dois Effects separados:

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

No exemplo acima, excluir um Effect não quebraria a lógica do outro Effect. Esta é uma boa indicação de que eles sincronizam coisas diferentes, e por isso fez sentido dividi-los. Por outro lado, se você dividir uma peça coesa de lógica em Effects separados, o código pode parecer "mais limpo", mas será [mais difícil de manter.](/learn/you-might-not-need-an-effect#chains-of-computations) É por isso que você deve pensar se os processos são os mesmos ou separados, e não se o código parece mais limpo.

## Efeitos "reagem" a valores reativos {/*effects-react-to-reactive-values*/}

Seu Effect lê duas variáveis (`serverUrl` e `roomId`), mas você especificou apenas `roomId` como dependência:

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

Isso ocorre porque `serverUrl` nunca muda devido a uma re-renderização. É sempre o mesmo, não importa quantas vezes o componente seja re-renderizado e por quê. Como `serverUrl` nunca muda, não faria sentido especificá-lo como uma dependência. Afinal, as dependências só fazem algo quando mudam ao longo do tempo!

Por outro lado, `roomId` pode ser diferente em uma re-renderização. **Props, estado e outros valores declarados dentro do componente são _reativos_ porque são calculados durante a renderização e participam do fluxo de dados do React.**

Se `serverUrl` fosse uma variável de estado, seria reativa. Valores reativos devem ser incluídos nas dependências:

```js {2,5,10}
function ChatRoom({ roomId }) { // Props mudam ao longo do tempo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // O estado pode mudar ao longo do tempo

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Seu Effect lê props e estado
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // Então você diz ao React que este Effect "depende" de props e estado
  // ...
}
```

Ao incluir `serverUrl` como uma dependência, você garante que o Effect se resincronize após a mudança.

Tente alterar a sala de chat selecionada ou editar a URL do servidor neste sandbox:

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
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Sempre que você altera um valor reativo como `roomId` ou `serverUrl`, o Effect se reconecta ao servidor de chat.

### O que um Effect com dependências vazias significa {/*what-an-effect-with-empty-dependencies-means*/}

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
  }, []); // ✅ All dependencies declared
  // ...
}
```

Agora, o código do seu Effect não usa *nenhum* valor reativo, então suas dependências podem estar vazias (`[]`).

Pensando da perspectiva do componente, o array de dependências vazio `[]` significa que este Effect se conecta à sala de chat apenas quando o componente é montado e se desconecta apenas quando o componente é desmontado. (Lembre-se de que o React ainda o [ressincronizaria uma vez extra](#how-react-verifies-that-your-effect-can-re-synchronize) em desenvolvimento para testar sua lógica.)


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
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

```js src/chat.js
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
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

No entanto, se você [pensar da perspectiva do Effect,](#thinking-from-the-effects-perspective) você não precisa pensar em montagem e desmontagem. O importante é que você especificou o que seu Effect faz para iniciar e parar a sincronização. Hoje, ele não tem dependências reativas. Mas se você quiser que o usuário altere `roomId` ou `serverUrl` ao longo do tempo (e eles se tornariam reativos), o código do seu Effect não mudará. Você só precisará adicioná-los às dependências.

### Todas as variáveis declaradas no corpo do componente são reativas {/*all-variables-declared-in-the-component-body-are-reactive*/}

Props e estado não são os únicos valores reativos. Valores que você calcula a partir deles também são reativos. Se as props ou o estado mudarem, seu componente será re-renderizado e os valores calculados a partir deles também mudarão. É por isso que todas as variáveis do corpo do componente usadas pelo Effect devem estar na lista de dependências do Effect.

Digamos que o usuário possa escolher um servidor de chat no dropdown, mas também possa configurar um servidor padrão nas configurações. Suponha que você já colocou o estado de configurações em um [contexto](/learn/scaling-up-with-reducer-and-context) para que você leia as `settings` desse contexto. Agora você calcula `serverUrl` com base no servidor selecionado das props e no servidor padrão:

```js {3,5,10}
function ChatRoom({ roomId, selectedServerUrl }) { // roomId é reativo
  const settings = useContext(SettingsContext); // settings é reativo
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl é reativo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Seu Effect lê roomId e serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // Então ele precisa resincronizar quando qualquer um deles mudar!
  // ...
}
```

Neste exemplo, `serverUrl` não é uma prop ou uma variável de estado. É uma variável comum que você calcula durante a renderização. Mas é calculada durante a renderização, então pode mudar devido a uma re-renderização. É por isso que é reativa.

**Todos os valores dentro do componente (incluindo props, estado e variáveis no corpo do seu componente) são reativos. Qualquer valor reativo pode mudar em uma re-renderização, então você precisa incluir valores reativos como dependências do Effect.**

Em outras palavras, os Effects "reagem" a todos os valores do corpo do componente.

<DeepDive>

#### Valores globais ou mutáveis podem ser dependências? {/*can-global-or-mutable-values-be-dependencies*/}

Valores mutáveis (incluindo variáveis globais) não são reativos.

**Um valor mutável como [`location.pathname`](https://developer.mozilla.org/en-US/docs/Web/API/Location/pathname) não pode ser uma dependência.** É mutável, então pode mudar a qualquer momento completamente fora do fluxo de dados de renderização do React. Mudá-lo não acionaria uma re-renderização do seu componente. Portanto, mesmo que você o especificasse nas dependências, o React *não saberia* resincronizar o Effect quando ele mudasse. Isso também quebra as regras do React porque ler dados mutáveis durante a renderização (que é quando você calcula as dependências) quebra [a pureza da renderização.](/learn/keeping-components-pure) Em vez disso, você deve ler e se inscrever em um valor mutável externo com [`useSyncExternalStore`.](/learn/you-might-not-need-an-effect#subscribing-to-an-external-store)

**Um valor mutável como [`ref.current`](/reference/react/useRef#reference) ou coisas que você lê dele também não pode ser uma dependência.** O objeto ref retornado por `useRef` em si pode ser uma dependência, mas sua propriedade `current` é intencionalmente mutável. Ele permite que você [rastreie algo sem acionar uma re-renderização.](/learn/referencing-values-with-refs) Mas como mudá-lo não aciona uma re-renderização, não é um valor reativo, e o React não saberá executar seu Effect quando ele mudar.

Como você aprenderá abaixo nesta página, um linter verificará esses problemas automaticamente.

</DeepDive>

### O React verifica se você especificou cada valor reativo como uma dependência {/*react-verifies-that-you-specified-every-reactive-value-as-a-dependency*/}

Se o seu linter estiver [configurado para React,](/learn/editor-setup#linting) ele verificará se cada valor reativo usado pelo código do seu Effect está declarado como sua dependência. Por exemplo, este é um erro de lint porque tanto `roomId` quanto `serverUrl` são reativos:

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
  }, []); // <-- Something's wrong here!

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Isso pode parecer um erro do React, mas na verdade o React está apontando um bug no seu código. Tanto `roomId` quanto `serverUrl` podem mudar ao longo do tempo, mas você esqueceu de resincronizar seu Effect quando eles mudam. Você permanecerá conectado ao `roomId` e `serverUrl` iniciais, mesmo depois que o usuário escolher valores diferentes na interface do usuário.

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
  }, [serverUrl, roomId]); // ✅ All dependencies declared
  // ...
}
```

Tente esta correção no sandbox acima. Verifique se o erro do linter desapareceu e se o chat se reconecta quando necessário.

<Note>

Em alguns casos, o React *sabe* que um valor nunca muda, mesmo que seja declarado dentro do componente. Por exemplo, a função `set` retornada por `useState` e o objeto ref retornado por `useRef` são *estáveis* - eles garantem que não mudarão em uma re-renderização. Valores estáveis não são reativos, então você pode omiti-los da lista. Incluí-los é permitido: eles não mudarão, então não importa.

</Note>

### O que fazer quando você não quer ressincronizar {/*what-to-do-when-you-dont-want-to-re-synchronize*/}

No exemplo anterior, você corrigiu o erro do linter listando `roomId` e `serverUrl` como dependências.

**No entanto, você poderia "provar" ao linter que esses valores não são valores reativos,** ou seja, que eles *não podem* mudar como resultado de uma nova renderização. Por exemplo, se `serverUrl` e `roomId` não dependem da renderização e sempre têm os mesmos valores, você pode movê-los para fora do componente. Agora eles não precisam ser dependências:

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

Você também pode movê-los *para dentro do Effect*. Eles não são calculados durante a renderização, então não são reativos:

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

**Effects são blocos de código reativos.** Eles ressincronizam quando os valores que você lê dentro deles mudam. Ao contrário dos manipuladores de eventos, que rodam apenas uma vez por interação, os Effects rodam sempre que a sincronização é necessária.

**Você não pode "escolher" suas dependências.** Suas dependências devem incluir todo [valor reativo](#all-variables-declared-in-the-component-body-are-reactive) que você lê dentro do Effect. O linter impõe isso. Às vezes, isso pode levar a problemas como loops infinitos e a seu Effect ressincronizar com muita frequência. Não corrija esses problemas suprimindo o linter! Veja o que tentar em vez disso:

*   **Verifique se seu Effect representa um processo de sincronização independente.** Se seu Effect não sincroniza nada, [ele pode ser desnecessário.](/learn/you-might-not-need-an-effect) Se ele sincroniza várias coisas independentes, [divida-o.](#each-effect-represents-a-separate-synchronization-process)

*   **Se você quiser ler o valor mais recente de props ou state sem "reagir" a ele e ressincronizar o Effect,** você pode dividir seu Effect em uma parte reativa (que você manterá no Effect) e uma parte não reativa (que você extrairá para algo chamado _Evento de Effect_). [Leia sobre como separar Eventos de Effects.](/learn/separating-events-from-effects)

*   **Evite depender de objetos e funções como dependências.** Se você cria objetos e funções durante a renderização e depois os lê de um Effect, eles serão diferentes a cada renderização. Isso fará com que seu Effect ressincronize a cada vez. [Leia mais sobre como remover dependências desnecessárias de Effects.](/learn/removing-effect-dependencies)

<Pitfall>

O linter é seu amigo, mas seus poderes são limitados. O linter só sabe quando as dependências estão *erradas*. Ele não sabe qual é a *melhor* maneira de resolver cada caso. Se o linter sugere uma dependência, mas adicioná-la causa um loop, isso não significa que o linter deva ser ignorado. Você precisa alterar o código dentro (ou fora) do Effect para que esse valor não seja reativo e não *precise* ser uma dependência.

Se você tem um codebase existente, pode ter alguns Effects que suprimem o linter assim:

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 Evite suprimir o linter assim:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

Nas [próximas](/learn/separating-events-from-effects) [páginas](/learn/removing-effect-dependencies), você aprenderá como corrigir esse código sem quebrar as regras. Vale sempre a pena corrigir!

</Pitfall>

<Recap>

- Componentes podem montar, atualizar e desmontar.
- Cada Effect tem um ciclo de vida separado do componente circundante.
- Cada Effect descreve um processo de sincronização separado que pode *começar* e *parar*.
- Ao escrever e ler Effects, pense a partir da perspectiva de cada Effect individual (como iniciar e parar a sincronização) em vez da perspectiva do componente (como ele monta, atualiza ou desmonta).
- Valores declarados dentro do corpo do componente são "reativos".
- Valores reativos devem ressincronizar o Effect porque eles podem mudar ao longo do tempo.
- O linter verifica se todos os valores reativos usados dentro do Effect são especificados como dependências.
- Todos os erros sinalizados pelo linter são legítimos. Sempre há uma maneira de corrigir o código para não quebrar as regras.

</Recap>

<Challenges>

#### Corrigir reconexão a cada pressionamento de tecla {/*fix-reconnecting-on-every-keystroke*/}

Neste exemplo, o componente `ChatRoom` se conecta à sala de chat quando o componente é montado, desconecta quando é desmontado e reconecta quando você seleciona uma sala de chat diferente. Esse comportamento está correto, então você precisa mantê-lo funcionando.

No entanto, há um problema. Sempre que você digita na caixa de entrada de mensagens na parte inferior, `ChatRoom` *também* se reconecta ao chat. (Você pode notar isso limpando o console e digitando na entrada.) Corrija o problema para que isso não aconteça.

<Hint>

Você pode precisar adicionar um array de dependências para este Effect. Quais dependências deveriam estar lá?

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
      <h1>Welcome to the {roomId} room!</h1>
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
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution>

Este Effect não tinha um array de dependências, então ele ressincronizou após cada re-renderização. Primeiro, adicione um array de dependências. Em seguida, certifique-se de que cada valor reativo usado pelo Effect esteja especificado no array. Por exemplo, `roomId` é reativo (porque é uma prop), então ele deve ser incluído no array. Isso garante que, quando o usuário selecionar uma sala diferente, o chat se reconectará. Por outro lado, `serverUrl` é definido fora do componente. É por isso que ele não precisa estar no array.

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
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
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
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

</Solution>

#### Ligar e desligar a sincronização {/*switch-synchronization-on-and-off*/}

Neste exemplo, um Effect se inscreve no evento [`pointermove`](https://developer.mozilla.org/en-US/docs/Web/API/Element/pointermove_event) da janela para mover um ponto rosa na tela. Tente passar o mouse sobre a área de visualização (ou tocar na tela se estiver em um dispositivo móvel) e veja como o ponto rosa segue seu movimento.

Há também uma caixa de seleção. Marcar a caixa de seleção alterna a variável de estado `canMove`, mas essa variável de estado não é usada em nenhum lugar no código. Sua tarefa é alterar o código para que, quando `canMove` for `false` (a caixa de seleção estiver desmarcada), o ponto pare de se mover. Depois de marcar a caixa de seleção novamente (e definir `canMove` como `true`), o ponto seguirá o movimento novamente. Em outras palavras, se o ponto pode se mover ou não deve permanecer sincronizado com se a caixa de seleção está marcada.

<Hint>

Você não pode declarar um Effect condicionalmente. No entanto, o código dentro do Effect pode usar condições!

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
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

<Solution>

Uma solução é envolver a chamada `setPosition` em uma condição `if (canMove) { ... }`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      if (canMove) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, [canMove]);

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

Alternativamente, você poderia envolver a lógica de *inscrição de eventos* em uma condição `if (canMove) { ... }`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    if (canMove) {
      window.addEventListener('pointermove', handleMove);
      return () => window.removeEventListener('pointermove', handleMove);
    }
  }, [canMove]);

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

Em ambos esses casos, `canMove` é uma variável reativa que você lê dentro do Effect. É por isso que ela deve ser especificada na lista de dependências do Effect. Isso garante que o Effect ressincronize após cada mudança em seu valor.

</Solution>

#### Investigue um erro de valor desatualizado {#investigate-a-stale-value-bug}

Neste exemplo, o ponto rosa deveria se mover quando a caixa de seleção estivesse marcada e parar de se mover quando estivesse desmarcada. A lógica para isso já foi implementada: o manipulador de eventos `handleMove` verifica a variável de estado `canMove`.

No entanto, por algum motivo, a variável de estado `canMove` dentro de `handleMove` parece estar "desatualizada": ela está sempre como `true`, mesmo depois de desmarcar a caixa de seleção. Como isso é possível? Encontre o erro no código e corrija-o.

<Hint>

Se você vir uma regra de linter sendo suprimida, remova a supressão! É aí que os erros geralmente estão.

</Hint>

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

<Solution>

O problema com o código original era a supressão do linter de dependências. Se você remover a supressão, verá que este Effect depende da função `handleMove`. Isso faz sentido: `handleMove` é declarada dentro do corpo do componente, o que a torna um valor reativo. Todo valor reativo deve ser especificado como dependência, ou ele pode potencialmente ficar desatualizado com o tempo!

O autor do código original "mentiu" para o React, dizendo que o Effect não depende (`[]`) de nenhum valor reativo. É por isso que o React não resincronizou o Effect após `canMove` ter mudado (e `handleMove` com ele). Como o React não resincronizou o Effect, o `handleMove` anexado como listener é a função `handleMove` criada durante a renderização inicial. Durante a renderização inicial, `canMove` era `true`, que é por que `handleMove` da renderização inicial sempre verá esse valor.

**Se você nunca suprimir o linter, nunca terá problemas com valores desatualizados.** Existem algumas maneiras diferentes de resolver este erro, mas você deve sempre começar removendo a supressão do linter. Em seguida, altere o código para corrigir o erro do linter.

Você pode alterar as dependências do Effect para `[handleMove]`, mas como será uma função recém-definida para cada renderização, você pode simplesmente remover o array de dependências. Então o Effect *irá* resincronizar após cada re-renderização:

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
  });

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

Esta solução funciona, mas não é ideal. Se você colocar `console.log('Resubscribing')` dentro do Effect, notará que ele ressubcreve após cada re-renderização. Ressubcrever é rápido, mas ainda seria bom evitar fazer isso com tanta frequência.

Uma correção melhor seria mover a função `handleMove` *para dentro* do Effect. Então `handleMove` não será um valor reativo, e seu Effect não dependerá de uma função. Em vez disso, ele precisará depender de `canMove`, que seu código agora lê de dentro do Effect. Isso corresponde ao comportamento que você desejava, já que seu Effect agora permanecerá sincronizado com o valor de `canMove`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      if (canMove) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
    }

    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, [canMove]);

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

Tente adicionar `console.log('Resubscribing')` dentro do corpo do Effect e note que agora ele só ressubcreve quando você alterna a caixa de seleção (`canMove` muda) ou edita o código. Isso o torna melhor do que a abordagem anterior que sempre ressubscrevia.

Você aprenderá uma abordagem mais geral para este tipo de problema em [Separando Eventos de Effects.](/learn/separating-events-from-effects)

</Solution>

#### Corrigir um switch de conexão {/*fix-a-connection-switch*/}

Neste exemplo, o serviço de chat em `chat.js` expõe duas APIs diferentes: `createEncryptedConnection` e `createUnencryptedConnection`. O componente raiz `App` permite que o usuário escolha se deseja usar criptografia ou não, e então passa o método de API correspondente para o componente filho `ChatRoom` como a prop `createConnection`.

Observe que, inicialmente, os logs do console indicam que a conexão não está criptografada. Tente marcar a caixa de seleção: nada acontecerá. No entanto, se você mudar a sala selecionada depois disso, o chat se reconectará *e* ativará a criptografia (como você verá nas mensagens do console). Este é um erro. Corrija o erro para que marcar a caixa de seleção *também* cause a reconexão do chat.

<Hint>

Suprimir o linter é sempre suspeito. Isso poderia ser um erro?

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Ativar criptografia
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        createConnection={isEncrypted ?
          createEncryptedConnection :
          createUnencryptedConnection
        }
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';

export default function ChatRoom({ roomId, createConnection }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [roomId]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ 🔐 Conectando a "' + roomId + '... (criptografado)');
    },
    disconnect() {
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando a "' + roomId + '... (não criptografado)');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" (não criptografado)');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

Se você remover a supressão do linter, verá um erro de lint. O problema é que `createConnection` é uma prop, então é um valor reativo. Ele pode mudar com o tempo! (E, de fato, deveria - quando o usuário marca a caixa de seleção, o componente pai passa um valor diferente da prop `createConnection`.) É por isso que ele deve ser uma dependência. Inclua-o na lista para corrigir o erro:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Ativar criptografia
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        createConnection={isEncrypted ?
          createEncryptedConnection :
          createUnencryptedConnection
        }
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';

export default function ChatRoom({ roomId, createConnection }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, createConnection]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ 🔐 Conectando a "' + roomId + '... (criptografado)');
    },
    disconnect() {
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando a "' + roomId + '... (não criptografado)');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" (não criptografado)');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

Está correto que `createConnection` é uma dependência. No entanto, este código é um pouco frágil porque alguém poderia editar o componente `App` para passar uma função inline como valor desta prop. Nesse caso, seu valor seria diferente toda vez que o componente `App` fosse renderizado novamente, então o Effect poderia ressincronizar com muita frequência. Para evitar isso, você pode passar `isEncrypted` em vez disso:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Ativar criptografia
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted }) {
  useEffect(() => {
    const createConnection = isEncrypted ?
      createEncryptedConnection :
      createUnencryptedConnection;
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ 🔐 Conectando a "' + roomId + '... (criptografado)');
    },
    disconnect() {
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando a "' + roomId + '... (não criptografado)');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" (não criptografado)');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

Nesta versão, o componente `App` passa um booleano em vez de uma função. Dentro do Effect, você decide qual função usar. Como `createEncryptedConnection` e `createUnencryptedConnection` são declaradas fora do componente, elas não são reativas e não precisam ser dependências. Você aprenderá mais sobre isso em [Removendo Dependências de Effect.](/learn/removing-effect-dependencies)

</Solution>

#### Preencher uma cadeia de caixas de seleção {/*populate-a-chain-of-select-boxes*/}

Neste exemplo, existem duas caixas de seleção. Uma caixa de seleção permite ao usuário escolher um planeta. Outra caixa de seleção permite ao usuário escolher um lugar *nesse planeta*. A segunda caixa ainda não funciona. Sua tarefa é fazê-la mostrar os lugares no planeta escolhido.

Veja como a primeira caixa de seleção funciona. Ela preenche o estado `planetList` com o resultado da chamada da API `"/planets"`. O ID do planeta atualmente selecionado é mantido na variável de estado `planetId`. Você precisa encontrar onde adicionar algum código adicional para que a variável de estado `placeList` seja preenchida com o resultado da chamada da API `"/planets/" + planetId + "/places"`.

Se você implementar isso corretamente, selecionar um planeta deve preencher a lista de lugares. Mudar um planeta deve mudar a lista de lugares.

<Hint>

Se você tem dois processos de sincronização independentes, você precisa escrever dois Effects separados.

</Hint>

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export default function Page() {
  const [planetList, setPlanetList] = useState([])
  const [planetId, setPlanetId] = useState('');

  const [placeList, setPlaceList] = useState([]);
  const [placeId, setPlaceId] = useState('');

  useEffect(() => {
    let ignore = false;
    fetchData('/planets').then(result => {
      if (!ignore) {
        console.log('Fetched a list of planets.');
        setPlanetList(result);
        setPlanetId(result[0].id); // Select the first planet
      }
    });
    return () => {
      ignore = true;
    }
  }, []);

  return (
    <>
      <label>
        Pick a planet:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Pick a place:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>You are going to: {placeId || '???'} on {planetId || '???'} </p>
    </>
  );
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('Expected URL like "/planets/earth/places". Received: "' + url + '".');
    }
    return fetchPlaces(match[1]);
  } else throw Error('Expected URL like "/planets" or "/planets/earth/places". Received: "' + url + '".');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'        
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) expects a string argument. ' +
      'Instead received: ' + planetId + '.'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'        
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'        
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('Unknown planet ID: ' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

Existem dois processos de sincronização independentes:

- A primeira caixa de seleção está sincronizada com a lista remota de planetas.
- A segunda caixa de seleção está sincronizada com a lista remota de lugares para o `planetId` atual.

É por isso que faz sentido descrevê-los como dois Effects separados. Aqui está um exemplo de como você poderia fazer isso:

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export default function Page() {
  const [planetList, setPlanetList] = useState([])
  const [planetId, setPlanetId] = useState('');

  const [placeList, setPlaceList] = useState([]);
  const [placeId, setPlaceId] = useState('');

  useEffect(() => {
    let ignore = false;
    fetchData('/planets').then(result => {
      if (!ignore) {
        console.log('Fetched a list of planets.');
        setPlanetList(result);
        setPlanetId(result[0].id); // Select the first planet
      }
    });
    return () => {
      ignore = true;
    }
  }, []);

  useEffect(() => {
    if (planetId === '') {
      // Nothing is selected in the first box yet
      return;
    }

    let ignore = false;
    fetchData('/planets/' + planetId + '/places').then(result => {
      if (!ignore) {
        console.log('Fetched a list of places on "' + planetId + '".');
        setPlaceList(result);
        setPlaceId(result[0].id); // Select the first place
      }
    });
    return () => {
      ignore = true;
    }
  }, [planetId]);

  return (
    <>
      <label>
        Pick a planet:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Pick a place:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>You are going to: {placeId || '???'} on {planetId || '???'} </p>
    </>
  );
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('Expected URL like "/planets/earth/places". Received: "' + url + '".');
    }
    return fetchPlaces(match[1]);
  } else throw Error('Expected URL like "/planets" or "/planets/earth/places". Received: "' + url + '".');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'        
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) expects a string argument. ' +
      'Instead received: ' + planetId + '.'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'        
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'        
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('Unknown planet ID: ' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

Este código é um pouco repetitivo. No entanto, essa não é uma boa razão para combiná-lo em um único Effect! Se você fizesse isso, teria que combinar as dependências de ambos os Effects em uma única lista, e então mudar o planeta faria com que a lista de todos os planetas fosse buscada novamente. Effects não são uma ferramenta para reutilização de código.

Em vez disso, para reduzir a repetição, você pode extrair alguma lógica para um Hook personalizado como `useSelectOptions` abaixo:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useSelectOptions } from './useSelectOptions.js';

export default function Page() {
  const [
    planetList,
    planetId,
    setPlanetId
  ] = useSelectOptions('/planets');

  const [
    placeList,
    placeId,
    setPlaceId
  ] = useSelectOptions(planetId ? `/planets/${planetId}/places` : null);

  return (
    <>
      <label>
        Pick a planet:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList?.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Pick a place:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList?.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>You are going to: {placeId || '...'} on {planetId || '...'} </p>
    </>
  );
}
```

```js src/useSelectOptions.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export function useSelectOptions(url) {
  const [list, setList] = useState(null);
  const [selectedId, setSelectedId] = useState('');
  useEffect(() => {
    if (url === null) {
      return;
    }

    let ignore = false;
    fetchData(url).then(result => {
      if (!ignore) {
        setList(result);
        setSelectedId(result[0].id);
      }
    });
    return () => {
      ignore = true;
    }
  }, [url]);
  return [list, selectedId, setSelectedId];
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('Expected URL like "/planets/earth/places". Received: "' + url + '".');
    }
    return fetchPlaces(match[1]);
  } else throw Error('Expected URL like "/planets" or "/planets/earth/places". Received: "' + url + '".');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'        
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) expects a string argument. ' +
      'Instead received: ' + planetId + '.'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'        
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'        
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('Unknown planet ID: ' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

Verifique a aba `useSelectOptions.js` no sandbox para ver como ela funciona. Idealmente, a maioria dos Effects em sua aplicação acabará sendo substituída por Hooks personalizados, sejam escritos por você ou pela comunidade. Hooks personalizados ocultam a lógica de sincronização, para que o componente que chama não saiba sobre o Effect. À medida que você continua trabalhando em seu aplicativo, desenvolverá uma paleta de Hooks para escolher, e eventualmente você não precisará escrever Effects em seus componentes com muita frequência.

</Solution>

</Challenges>