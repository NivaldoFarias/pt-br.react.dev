---
title: 'Ciclo de Vida de Efeitos Reativos'
---

<Intro>

Efeitos têm um ciclo de vida diferente dos componentes. Os componentes podem montar, atualizar ou desmontar. Um Effect só pode fazer duas coisas: iniciar a sincronização de algo e, posteriormente, parar de sincronizá-lo. Este ciclo pode acontecer várias vezes se o seu Effect depender de props e state que mudam ao longo do tempo. React fornece uma regra de linter para verificar se você especificou as dependências do seu Effect corretamente. Isso mantém seu Effect sincronizado com as props e o state mais recentes.

</Intro>

<YouWillLearn>

- Como o ciclo de vida de um Effect é diferente do ciclo de vida de um componente
- Como pensar em cada Effect individualmente, de forma isolada
- Quando seu Effect precisa se re-sincronizar e por quê
- Como as dependências do seu Effect são determinadas
- O que significa para um valor ser reativo
- O que uma matriz de dependência vazia significa
- Como React verifica se suas dependências estão corretas com um linter
- O que fazer quando você não concorda com o linter

</YouWillLearn>

## O ciclo de vida de um Effect {/*the-lifecycle-of-an-effect*/}

Cada componente React passa pelo mesmo ciclo de vida:

- Um componente _monta_ quando é adicionado à tela.
- Um componente _atualiza_ quando recebe novas props ou state, geralmente em resposta a uma interação.
- Um componente _desmonta_ quando é removido da tela.

**É uma boa maneira de pensar sobre componentes, mas _não_ sobre Effects.** Em vez disso, tente pensar em cada Effect independentemente do ciclo de vida do seu componente. Um Effect descreve como [sincronizar um sistema externo](/learn/synchronizing-with-effects) com as props e o state atuais. À medida que seu código muda, a sincronização precisará acontecer mais ou menos vezes.

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

O corpo do seu Effect especifica como **iniciar a sincronização**:

```js {2-3}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

A função de limpeza retornada pelo seu Effect especifica como **parar a sincronização**:

```js {5}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

Intuitivamente, você pode pensar que React **iniciaria a sincronização** quando seu componente monta e **pararia a sincronização** quando seu componente desmonta. No entanto, esta não é a história toda! Às vezes, também pode ser necessário **iniciar e parar a sincronização várias vezes** enquanto o componente permanece montado.

Vamos ver _por que_ isso é necessário, _quando_ isso acontece e _como_ você pode controlar esse comportamento.

<Note>

Alguns Effects não retornam uma função de limpeza. [Na maioria das vezes,](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) você vai querer retornar uma -- mas se você não o fizer, React irá se comportar como se você tivesse retornado uma função de limpeza vazia.

</Note>

### Por que a sincronização pode precisar acontecer mais de uma vez {/*why-synchronization-may-need-to-happen-more-than-once*/}

Imagine que este componente `ChatRoom` recebe uma prop `roomId` que o usuário escolhe em um menu suspenso. Vamos dizer que inicialmente o usuário escolhe a sala `"general"` como `roomId`. Seu aplicativo exibe a sala de chat `"general"`:

```js {3}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

Depois que a UI é exibida, React executará seu Effect para **iniciar a sincronização.** Ele se conecta à sala `"general"`:

```js {3,4}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Desconecta-se da sala "general"
    };
  }, [roomId]);
  // ...
```

Até agora, tudo bem.

Mais tarde, o usuário escolhe uma sala diferente no menu suspenso (por exemplo, `"travel"`). Primeiro, React atualizará a UI:

```js {1}
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

Pense no que deve acontecer a seguir. O usuário vê que `"travel"` é a sala de chat selecionada na UI. No entanto, o Effect que foi executado da última vez ainda está conectado à sala `"general"`. **A prop `roomId` mudou, então o que seu Effect fez naquela época (conectar à sala `"general"`) não corresponde mais à UI.**

Neste ponto, você deseja que React faça duas coisas:

1. Pare de sincronizar com o `roomId` antigo (desconecte-se da sala `"general"`)
2. Comece a sincronizar com o novo `roomId` (conecte-se à sala `"travel"`)

**Felizmente, você já ensinou o React a fazer as duas coisas!** O corpo do seu Effect especifica como iniciar a sincronização, e sua função de limpeza especifica como parar a sincronização. Tudo o que o React precisa fazer agora é chamá-los na ordem correta e com as props e o state corretos. Vamos ver como exatamente isso acontece.

### Como React re-sincroniza seu Effect {/*how-react-re-synchronizes-your-effect*/}

Lembre-se que seu componente `ChatRoom` recebeu um novo valor para sua prop `roomId`. Costumava ser `"general"`, e agora é `"travel"`. React precisa re-sincronizar seu Effect para reconectá-lo a uma sala diferente.

Para **parar a sincronização,** React chamará a função de limpeza que seu Effect retornou após conectar à sala `"general"`. Como `roomId` era `"general"`, a função de limpeza se desconecta da sala `"general"`:

```js {6}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Desconecta-se da sala "general"
    };
    // ...
```

Então, React executará o Effect que você forneceu durante esta renderização. Desta vez, `roomId` é `"travel"`, então ele **começará a sincronizar** com a sala de chat `"travel"` (até que sua função de limpeza seja eventualmente chamada também):

```js {3,4}
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "travel"
    connection.connect();
    // ...
```

Graças a isso, você agora está conectado à mesma sala que o usuário escolheu na UI. Desastre evitado!

Toda vez que seu componente renderizar novamente com um `roomId` diferente, seu Effect re-sincronizará. Por exemplo, digamos que o usuário altere `roomId` de `"travel"` para `"music"`. React novamente **parará de sincronizar** seu Effect chamando sua função de limpeza (desconectando você da sala `"travel"`). Então ele **começará a sincronizar** novamente executando seu corpo com a nova prop `roomId` (conectando você à sala `"music"`).

Finalmente, quando o usuário for para uma tela diferente, `ChatRoom` desmontará. Agora não há necessidade de permanecer conectado. React irá **parar de sincronizar** seu Effect pela última vez e desconectá-lo da sala de chat `"music"`.

### Pensando na perspectiva do Effect {/*thinking-from-the-effects-perspective*/}

Vamos recapitular tudo o que aconteceu da perspectiva do componente `ChatRoom`:

1. `ChatRoom` montado com `roomId` definido como `"general"`
1. `ChatRoom` atualizado com `roomId` definido como `"travel"`
1. `ChatRoom` atualizado com `roomId` definido como `"music"`
1. `ChatRoom` desmontado

Durante cada um desses pontos no ciclo de vida do componente, seu Effect fez coisas diferentes:

1. Seu Effect conectou-se à sala `"general"`
1. Seu Effect desconectou-se da sala `"general"` e conectou-se à sala `"travel"`
1. Seu Effect desconectou-se da sala `"travel"` e conectou-se à sala `"music"`
1. Seu Effect desconectou-se da sala `"music"`

Agora, vamos pensar sobre o que aconteceu da perspectiva do próprio Effect:

```js
  useEffect(() => {
    // Seu Effect conectou-se à sala especificada com roomId...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...até que ele se desconectasse
      connection.disconnect();
    };
  }, [roomId]);
```

A estrutura deste código pode inspirá-lo a ver o que aconteceu como uma sequência de períodos de tempo não sobrepostos:

1. Seu Effect conectou-se à sala `"general"` (até que se desconectasse)
1. Seu Effect conectou-se à sala `"travel"` (até que se desconectasse)
1. Seu Effect conectou-se à sala `"music"` (até que se desconectasse)

Anteriormente, você estava pensando na perspectiva do componente. Quando você olhou da perspectiva do componente, era tentador pensar nos Effects como "callbacks" ou "eventos de ciclo de vida" que são acionados em um momento específico, como "após uma renderização" ou "antes de desmontar". Essa forma de pensar se torna complicada muito rapidamente, por isso é melhor evitar.

**Em vez disso, sempre se concentre em um único ciclo de início/parada de cada vez. Não deve importar se um componente está montando, atualizando ou desmontando. Tudo o que você precisa fazer é descrever como iniciar a sincronização e como pará-la. Se você fizer isso bem, seu Effect será resistente a ser iniciado e parado quantas vezes forem necessárias.**

Isso pode lembrá-lo de como você não pensa se um componente está montando ou atualizando ao escrever a lógica de renderização que cria JSX. Você descreve o que deve estar na tela e React [descobre o resto.](/learn/reacting-to-input-with-state)

### Como React verifica se seu Effect pode se re-sincronizar {/*how-react-verifies-that-your-effect-can-re-synchronize*/}

Aqui está um exemplo real com o qual você pode brincar. Pressione "Abrir chat" para montar o componente `ChatRoom`:

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

Observe que quando o componente monta pela primeira vez, você vê três logs:

1.  `✅ Conectando à sala "general" em https://localhost:1234...` *(somente desenvolvimento)*
2.  `❌ Desconectado da sala "general" em https://localhost:1234.` *(somente desenvolvimento)*
3.  `✅ Conectando à sala "general" em https://localhost:1234...`

Os dois primeiros logs são exclusivos para desenvolvimento. Em desenvolvimento, React sempre remonta cada componente uma vez.

**React verifica se seu Effect pode se re-sincronizar forçando-o a fazer isso imediatamente em desenvolvimento.** Isso pode lembrá-lo de abrir uma porta e fechá-la uma vez extra para verificar se a fechadura funciona. React inicia e para seu Effect uma vez extra em desenvolvimento para verificar [se você implementou sua limpeza bem.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

A principal razão pela qual seu Effect se re-sincronizará na prática é se alguns dados que ele usa foram alterados. No sandbox acima, altere a sala de chat selecionada. Observe como, quando o `roomId` muda, seu Effect re-sincroniza.

No entanto, também existem casos mais incomuns em que a re-sincronização é necessária. Por exemplo, tente editar o `serverUrl` no sandbox acima enquanto o chat estiver aberto. Observe como o Effect se re-sincroniza em resposta às suas edições no código. No futuro, React pode adicionar mais recursos que dependem da re-sincronização.

### Como React sabe que precisa re-sincronizar o Effect {/*how-react-knows-that-it-needs-to-re-synchronize-the-effect*/}

Você pode estar se perguntando como React sabia que seu Effect precisava ser re-sincronizado depois que `roomId` mudou. É porque *você disse ao React* que seu código depende de `roomId` incluindo-o na [lista de dependências:](/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)

```js {1,3,8}
function ChatRoom({ roomId }) { // A prop roomId pode mudar ao longo do tempo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Este Effect lê roomId 
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Então você diz ao React que este Effect "depende" de roomId
  // ...
```

Veja como isso funciona:

1.  Você sabia que `roomId` é uma prop, o que significa que pode mudar ao longo do tempo.
2.  Você sabia que seu Effect lê `roomId` (então sua lógica depende de um valor que pode mudar mais tarde).
3.  É por isso que você o especificou como dependência do seu Effect (para que ele se re-sincronize quando `roomId` mudar).

Toda vez que seu componente renderiza novamente, React examinará a array de dependências que você passou. Se algum dos valores na array for diferente do valor no mesmo local que você passou durante a renderização anterior, React re-sincronizará seu Effect.

Por exemplo, se você passou `["general"]` durante a renderização inicial, e mais tarde você passou `["travel"]` durante a próxima renderização, React comparará `"general"` e `"travel"`. Esses são valores diferentes (comparados com [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), então React irá re-sincronizar seu Effect. Por outro lado, se seu componente renderizar novamente, mas `roomId` não tiver mudado, seu Effect permanecerá conectado à mesma sala.

### Cada Effect representa um processo de sincronização separado {/*each-effect-represents-a-separate-synchronization-process*/}

Resista a adicionar lógica não relacionada ao seu Effect só porque essa lógica precisa ser executada ao mesmo tempo que um Effect que você já escreveu. Por exemplo, digamos que você queira enviar um evento de análise quando o usuário visitar a sala. Você já tem um Effect que depende de `roomId`, então pode ser tentador adicionar a chamada de análise lá:

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

Mas imagine que você adicione outra dependência a este Effect que precisa restabelecer a conexão. Se este Effect re-sincronizar, ele também chamará `logVisit(roomId)` para a mesma sala, o que você não pretendia. Registrar a visita **é um processo separado** de conexão. Escreva-os como dois Effects separados:

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

No exemplo acima, excluir um Effect não quebraria a lógica do outro Effect. Esta é uma boa indicação de que eles sincronizam coisas diferentes e, portanto, fez sentido separá-los. Por outro lado, se você dividir uma parte coesa de lógica em Effects separados, o código pode parecer "mais limpo", mas será [mais difícil de manter.](/learn/you-might-not-need-an-effect#chains-of-computations) É por isso que você deve pensar se os processos são iguais ou separados, não se o código parece mais limpo.

## Effects "reativos" a valores reativos {/*effects-react-to-reactive-values*/}

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

Isso ocorre porque o `serverUrl` nunca muda devido a uma nova renderização. É sempre o mesmo, não importa quantas vezes o componente renderize novamente e por quê. Como `serverUrl` nunca muda, não faria sentido especificá-lo como uma dependência. Afinal, as dependências só fazem algo quando mudam ao longo do tempo!

Por outro lado, `roomId` pode ser diferente em uma nova renderização. **Props, state e outros valores declarados dentro do componente são _reativos_ porque são calculados durante a renderização e participam do fluxo de dados React.**

Se `serverUrl` fosse uma variável de state, ela seria reativa. Valores reativos devem ser incluídos em dependências:
``````js {2,5,10}
function ChatRoom({ roomId }) { // As props mudam ao longo do tempo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // O estado pode mudar com o tempo

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

Ao incluir `serverUrl` como uma dependência, você garante que o Effect re-sincronize depois que ele mudar.

Tente mudar a sala de chat selecionada ou editar a URL do servidor neste sandbox:

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
        URL do Servidor:{' '}
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

Sempre que você muda um valor reativo como `roomId` ou `serverUrl`, o Effect se reconecta ao servidor de chat.

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
  }, []); // ✅ Todas as dependências declaradas
  // ...
}
```

Agora o código do seu Effect não usa *nenhum* valor reativo, então suas dependências podem estar vazias (`[]`).

Pensando na perspectiva do componente, a dependência de array vazia `[]` significa que este Effect se conecta à sala de chat apenas quando o componente monta e desconecta apenas quando o componente desmonta. (Lembre-se de que o React ainda [o re-sincronizaria uma vez extra](#how-react-verifies-that-your-effect-can-re-synchronize) no desenvolvimento para testar sua lógica.)


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

No entanto, se você [pensar na perspectiva do Effect,](#thinking-from-the-effects-perspective) não precisa pensar em montar e desmontar. O que é importante é que você especificou o que seu Effect faz para iniciar e parar a sincronização. Hoje, ele não tem dependências reativas. Mas se você quiser que o usuário mude `roomId` ou `serverUrl` com o tempo (e eles se tornariam reativos), o código do seu Effect não mudará. Você só precisará adicioná-los às dependências.

### Todas as variáveis declaradas no corpo do componente são reativas {/*all-variables-declared-in-the-component-body-are-reactive*/}

Props e state não são os únicos valores reativos. Os valores que você calcula a partir deles também são reativos. Se as props ou o estado mudarem, seu componente irá re-renderizar e os valores calculados a partir deles também mudarão. É por isso que todas as variáveis do corpo do componente usadas pelo Effect devem estar na lista de dependências do Effect.

Digamos que o usuário possa escolher um servidor de chat no dropdown, mas também pode configurar um servidor padrão nas configurações. Suponha que você já tenha colocado o estado das configurações em um [contexto](/learn/scaling-up-with-reducer-and-context) para que você leia as `settings` desse contexto. Agora você calcula o `serverUrl` com base no servidor selecionado nas props e no servidor padrão:

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
  }, [roomId, serverUrl]); // Então ele precisa re-sincronizar quando qualquer um deles mudar!
  // ...
}
```

Neste exemplo, `serverUrl` não é uma prop ou uma variável de estado. É uma variável regular que você calcula durante a renderização. Mas ela é calculada durante a renderização, então pode mudar devido a uma nova renderização. É por isso que é reativa.

**Todos os valores dentro do componente (incluindo props, estado e variáveis no corpo do seu componente) são reativos. Qualquer valor reativo pode mudar em uma nova renderização, então você precisa incluir valores reativos como dependências do Effect.**

Em outras palavras, os Effects "reagem" a todos os valores do corpo do componente.

<DeepDive>

#### As variáveis globais ou mutáveis ​​podem ser dependências? {/*can-global-or-mutable-values-be-dependencies*/}

Valores mutáveis ​​(incluindo variáveis globais) não são reativos.

**Um valor mutável ​​como [`location.pathname`](https://developer.mozilla.org/en-US/docs/Web/API/Location/pathname) não pode ser uma dependência.** É mutável, então pode mudar a qualquer momento completamente fora do fluxo de dados de renderização do React. Mudá-lo não acionaria uma nova renderização do seu componente. Portanto, mesmo que você o especificasse nas dependências, o React *não saberia* para re-sincronizar o Effect quando ele muda. Isso também quebra as regras do React porque ler dados mutáveis ​​durante a renderização (que é quando você calcula as dependências) quebra a [pureza da renderização.](/learn/keeping-components-pure) Em vez disso, você deve ler e se inscrever em um valor mutável externo com [`useSyncExternalStore`.](/learn/you-might-not-need-an-effect#subscribing-to-an-external-store)

**Um valor mutável como [`ref.current`](/reference/react/useRef#reference) ou coisas que você lê dele também não pode ser uma dependência.** O objeto ref retornado por `useRef` em si pode ser uma dependência, mas sua propriedade `current` é intencionalmente mutável. Ele permite que você [acompanhe algo sem acionar uma nova renderização.](/learn/referencing-values-with-refs) Mas, como mudá-lo não aciona uma nova renderização, não é um valor reativo e o React não saberá para executar seu Effect novamente quando ele mudar.

Como você aprenderá abaixo nesta página, um linter verificará esses problemas automaticamente.

</DeepDive>

### React verifica se você especificou todos os valores reativos como uma dependência {/*react-verifies-that-you-specified-every-reactive-value-as-a-dependency*/}

Se o seu linter estiver [configurado para React,](/learn/editor-setup#linting) ele verificará se cada valor reativo usado pelo código do seu Effect é declarado como sua dependência. Por exemplo, este é um erro de lint porque `roomId` e `serverUrl` são reativos:

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
        URL do Servidor:{' '}
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

Isso pode parecer um erro do React, mas na verdade o React está apontando um erro no seu código. Tanto `roomId` quanto `serverUrl` podem mudar com o tempo, mas você está esquecendo de re-sincronizar seu Effect quando eles mudam. Você permanecerá conectado ao `roomId` e `serverUrl` iniciais, mesmo depois que o usuário escolher valores diferentes na interface do usuário.

Para corrigir o erro, siga a sugestão do linter para especificar `roomId` e `serverUrl` como dependências do seu Effect:

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

Tente esta correção no sandbox acima. Verifique se o erro do linter desapareceu e se o chat se reconecta quando necessário.

<Note>

Em alguns casos, o React *sabe* que um valor nunca muda, embora seja declarado dentro do componente. Por exemplo, a função [`set`](/reference/react/useState#setstate) retornada de `useState` e o objeto ref retornado por [`useRef`](/reference/react/useRef) são *estáveis* -- eles têm a garantia de não mudar em uma nova renderização. Valores estáveis não são reativos, então você pode omiti-los da lista. Incluí-los é permitido: eles não mudarão, então não importa.

</Note>

### O que fazer quando você não quer re-sincronizar {/*what-to-do-when-you-dont-want-to-re-synchronize*/}

No exemplo anterior, você corrigiu o erro de lint listando `roomId` e `serverUrl` como dependências.

**No entanto, você poderia "provar" ao linter que esses valores não são valores reativos,** ou seja, que eles *não podem* mudar como resultado de uma nova renderização. Por exemplo, se `serverUrl` e `roomId` não dependerem da renderização e sempre tiverem os mesmos valores, você pode movê-los para fora do componente. Agora eles não precisam ser dependências:

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

Você também pode movê-los *para dentro do Effect.* Eles não são calculados durante a renderização, então não são reativos:

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

**Effects são blocos de código reativos.** Eles se re-sincronizam quando os valores que você lê dentro deles mudam. Ao contrário dos manipuladores de eventos, que só são executados uma vez por interação, os Effects são executados sempre que a sincronização é necessária.

**Você não pode "escolher" suas dependências.** Suas dependências devem incluir todos os [valores reativos](#all-variables-declared-in-the-component-body-are-reactive) que você lê no Effect. O linter força isso. Às vezes, isso pode levar a problemas como loops infinitos e ao seu Effect se re-sincronizar com muita frequência. Não conserte esses problemas suprimindo o linter! Veja o que tentar:

* **Verifique se seu Effect representa um processo de sincronização independente.** Se seu Effect não sincronizar nada, [pode ser desnecessário.](/learn/you-might-not-need-an-effect) Se ele sincroniza várias coisas independentes, [divida-o.](#each-effect-represents-a-separate-synchronization-process)

* **Se você deseja ler o valor mais recente de props ou estado sem "reagir" a ele e re-sincronizar o Effect,** você pode dividir seu Effect em uma parte reativa (que você manterá no Effect) e uma parte não reativa (que você extrairá para algo chamado de _Evento de Effect_). [Leia sobre como separar Eventos de Effects.](/learn/separating-events-from-effects)

* **Evite confiar em objetos e funções como dependências.** Se você criar objetos e funções durante a renderização e, em seguida, lê-los de um Effect, eles serão diferentes em cada renderização. Isso fará com que seu Effect seja re-sincronizado cada vez. [Leia mais sobre como remover dependências desnecessárias de Effects.](/learn/removing-effect-dependencies)

<Pitfall>

O linter é seu amigo, mas seus poderes são limitados. O linter só sabe quando as dependências estão *erradas*. Ele não sabe *a melhor* maneira de resolver cada caso. Se o linter sugere uma dependência, mas adicioná-la causa um loop, isso não significa que o linter deve ser ignorado. Você precisa mudar o código dentro (ou fora) do Effect para que esse valor não seja reativo e não *precisa* ser uma dependência.

Se você tiver uma base de código existente, pode ter alguns Effects que suprimem o linter assim:

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 Evite suprimir o linter assim:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

Nas [próximas](/learn/separating-events-from-effects) [páginas](/learn/removing-effect-dependencies), você aprenderá a corrigir este código sem quebrar as regras. Sempre vale a pena corrigir!

</Pitfall>

<Recap>

- Os componentes podem montar, atualizar e desmontar.
- Cada Effect tem um ciclo de vida separado do componente circundante.
- Cada Effect descreve um processo de sincronização separado que pode *iniciar* e *parar*.
- Ao escrever e ler Effects, pense na perspectiva de cada Effect individual (como iniciar e parar a sincronização) em vez da perspectiva do componente (como ele monta, atualiza ou desmonta).
- Valores declarados dentro do corpo do componente são "reativos".
- Valores reativos devem re-sincronizar o Effect porque podem mudar com o tempo.
- O linter verifica se todos os valores reativos usados ​​dentro do Effect são especificados como dependências.
- Todos os erros sinalizados pelo linter são legítimos. Sempre há uma maneira de corrigir o código para não quebrar as regras.

</Recap>

<Challenges>

#### Corrigir a reconexão a cada pressionamento de tecla {/*fix-reconnecting-on-every-keystroke*/}

Neste exemplo, o componente `ChatRoom` se conecta à sala de chat quando o componente monta, desconecta quando desmonta e se reconecta quando você seleciona uma sala de chat diferente. Este comportamento está correto, então você precisa mantê-lo funcionando.

No entanto, há um problema. Sempre que você digita na caixa de mensagem no final, o `ChatRoom` *também* se reconecta ao chat. (Você pode notar isso limpando o console e digitando no input.) Corrija o problema para que isso não aconteça.

<Hint>

Talvez seja necessário adicionar uma array de dependências para este Effect. Quais dependências devem estar lá?

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
}
```
```jsx
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
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
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
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ 🔐 Conectando à "' + roomId + '... (criptografado)');
    },
    disconnect() {
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à "' + roomId + '... (não criptografado)');
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

</Solution>
```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
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
  // A real implementation would actually connect to the server
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
  // A real implementation would actually connect to the server
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

É correto que `createConnection` é uma dependência. No entanto, este código é um pouco frágil porque alguém poderia editar o componente `App` para passar uma função inline como o valor desta prop. Nesse caso, seu valor seria diferente toda vez que o componente `App` renderizasse novamente, então o Effect poderia ressincronizar com muita frequência. Para evitar isso, você pode passar `isEncrypted` para baixo em vez disso:

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
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
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

  return <h1>Bem-vindo à sala {roomId} !</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // A real implementation would actually connect to the server
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
  // A real implementation would actually connect to the server
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

Nesta versão, o componente `App` passa uma prop booleana em vez de uma função. Dentro do Effect, você decide qual função usar. Como ambos `createEncryptedConnection` e `createUnencryptedConnection` são declarados fora do componente, eles não são reativos e não precisam ser dependências. Você aprenderá mais sobre isso em [Removendo as Dependências do Effect.](/learn/removing-effect-dependencies)

</Solution>

#### Preencher uma cadeia de caixas de seleção {/*populate-a-chain-of-select-boxes*/}

Neste exemplo, existem duas caixas de seleção. Uma caixa de seleção permite que o usuário escolha um planeta. Outra caixa de seleção permite que o usuário escolha um local *nesse planeta.* A segunda caixa ainda não funciona. Sua tarefa é fazê-la mostrar os locais no planeta escolhido.

Observe como a primeira caixa de seleção funciona. Ela preenche o estado `planetList` com o resultado da chamada de API `"/planets"`. O ID do planeta atualmente selecionado é mantido na variável de estado `planetId`. Você precisa encontrar onde adicionar algum código adicional para que a variável de estado `placeList` seja preenchida com o resultado da chamada de API `"/planets/" + planetId + "/places"`.

Se você implementar isso corretamente, a seleção de um planeta deverá preencher a lista de locais. A alteração de um planeta deve alterar a lista de locais.

<Hint>

Se você tiver dois processos de sincronização independentes, precisará escrever dois Effects separados.

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
        Escolha um planeta:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Escolha um local:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>Você vai para: {placeId || '???'} em {planetId || '???'} </p>
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

Há dois processos de sincronização independentes:

- A primeira caixa de seleção é sincronizada à lista remota de planetas.
- A segunda caixa de seleção é sincronizada à lista remota de locais para o `planetId` atual.

É por isso que faz sentido descrevê-los como dois Effects separados. Aqui está um exemplo de como você pode fazer isso:

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
        Escolha um planeta:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Escolha um local:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>Você vai para: {placeId || '???'} em {planetId || '???'} </p>
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

Este código é um pouco repetitivo. No entanto, essa não é uma boa razão para combiná-lo em um único Effect! Se você fizesse isso, teria que combinar as dependências dos dois Effects em uma lista e, em seguida, a alteração do planeta buscaria novamente a lista de todos os planetas. Os Effects não são uma ferramenta para reutilização de código.

Em vez disso, para reduzir a repetição, você pode extrair um pouco da lógica em um Hook personalizado, como `useSelectOptions` abaixo:

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
        Escolha um planeta:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList?.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Escolha um local:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList?.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>Você vai para: {placeId || '...'} em {planetId || '...'} </p>
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

Verifique a aba `useSelectOptions.js` na caixa de testes para ver como funciona. Idealmente, a maioria dos Effects em seu aplicativo deve eventualmente ser substituída por Hooks personalizados, seja escritos por você ou pela comunidade. Hooks personalizados ocultam a lógica de sincronização, para que o componente de chamada não saiba sobre o Effect. Conforme você continua trabalhando em seu aplicativo, você desenvolverá uma paleta de Hooks para escolher e, eventualmente, não precisará escrever Effects em seus componentes com muita frequência.

</Solution>

</Challenges>
```