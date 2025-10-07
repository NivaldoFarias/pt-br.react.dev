<Intro>

Efeitos têm um ciclo de vida diferente dos componentes. Componentes podem montar, atualizar ou desmontar. Um Efeito só pode fazer duas coisas: iniciar a sincronização de algo e, mais tarde, parar essa sincronização. Esse ciclo pode acontecer várias vezes se o seu Efeito depender de props e estado que mudam com o tempo. O React fornece uma regra de linter para verificar se você especificou as dependências do seu Efeito corretamente. Isso mantém seu Efeito sincronizado com as últimas props e estado.

</Intro>

<YouWillLearn>

- Como o ciclo de vida de um Efeito é diferente do ciclo de vida de um componente
- Como pensar sobre cada Efeito individualmente
- Quando seu Efeito precisa ressincronizar e por quê
- Como as dependências do seu Efeito são determinadas
- O que significa um valor ser reativo
- O que significa um array de dependências vazio
- Como o React verifica se suas dependências estão corretas com um linter
- O que fazer quando você discorda do linter

</YouWillLearn>

## O ciclo de vida de um Efeito {/*the-lifecycle-of-an-effect*/}

Todo componente React passa pelo mesmo ciclo de vida:

- Um componente _monta_ quando é adicionado à tela.
- Um componente _atualiza_ quando recebe novas props ou estado, geralmente em resposta a uma interação.
- Um componente _desmonta_ quando é removido da tela.

**É uma boa maneira de pensar sobre componentes, mas _não_ sobre Efeitos.** Em vez disso, tente pensar em cada Efeito independentemente do ciclo de vida do seu componente. Um Efeito descreve como [sincronizar um sistema externo](/learn/synchronizing-with-effects) com as props e o estado atuais. À medida que seu código muda, a sincronização precisará acontecer com mais ou menos frequência.

Para ilustrar este ponto, considere este Efeito que conecta seu componente a um servidor de chat:

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

O corpo do seu Efeito especifica como **iniciar a sincronização:**

```js {2-3}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

A função de limpeza retornada pelo seu Efeito especifica como **parar a sincronização:**

```js {5}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

Intuitivamente, você pode pensar que o React **iniciaria a sincronização** quando seu componente montasse e **pararia a sincronização** quando seu componente desmontasse. No entanto, este não é o fim da história! Às vezes, também pode ser necessário **iniciar e parar a sincronização várias vezes** enquanto o componente permanece montado.

Vamos ver _por que_ isso é necessário, _quando_ isso acontece e _como_ você pode controlar esse comportamento.

<Note>

Alguns Efeitos não retornam uma função de limpeza. [Na maioria das vezes,](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) você vai querer retornar uma — mas se não o fizer, o React se comportará como se você tivesse retornado uma função de limpeza vazia.

</Note>

### Por que a sincronização pode precisar acontecer mais de uma vez {/*why-synchronization-may-need-to-happen-more-than-once*/}

Imagine que este componente `ChatRoom` recebe uma prop `roomId` que o usuário seleciona em um dropdown. Vamos dizer que inicialmente o usuário escolhe a sala `"general"` como `roomId`. Seu aplicativo exibe a sala de chat `"general"`:

```js {3}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

Após a UI ser exibida, o React executará seu Efeito para **iniciar a sincronização.** Ele se conecta à sala `"general"`:

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

Mais tarde, o usuário escolhe uma sala diferente no dropdown (por exemplo, `"travel"`). Primeiro, o React atualizará a UI:

```js {1}
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

Pense no que deve acontecer a seguir. O usuário vê que `"travel"` é a sala de chat selecionada na UI. No entanto, o Efeito que foi executado da última vez ainda está conectado à sala `"general"`. **A prop `roomId` mudou, então o que seu Efeito fez naquela época (conectar-se à sala `"general"`) não corresponde mais à UI.**

Neste ponto, você quer que o React faça duas coisas:

1. Pare de sincronizar com o `roomId` antigo (desconecte-se da sala `"general"`)
2. Comece a sincronizar com o novo `roomId` (conecte-se à sala `"travel"`)

**Felizmente, você já ensinou ao React como fazer ambas as coisas!** O corpo do seu Efeito especifica como iniciar a sincronização, e sua função de limpeza especifica como parar a sincronização. Tudo o que o React precisa fazer agora é chamá-los na ordem correta e com as props e o estado corretos. Vamos ver exatamente como isso acontece.

### Como o React ressincroniza seu Efeito {/*how-react-re-synchronizes-your-effect*/}

Lembre-se que seu componente `ChatRoom` recebeu um novo valor para sua prop `roomId`. Costumava ser `"general"`, e agora é `"travel"`. O React precisa ressincronizar seu Efeito para reconectá-lo a uma sala diferente.

Para **parar a sincronização,** o React chamará a função de limpeza que seu Efeito retornou após conectar-se à sala `"general"`. Como `roomId` era `"general"`, a função de limpeza desconecta da sala `"general"`:

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

Em seguida, o React executará o Efeito que você forneceu durante esta renderização. Desta vez, `roomId` é `"travel"`, então ele **iniciará a sincronização** com a sala de chat `"travel"` (até que sua função de limpeza também seja eventualmente chamada):

```js {3,4}
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta-se à sala "travel"
    connection.connect();
    // ...
```

Graças a isso, você agora está conectado à mesma sala que o usuário escolheu na UI. Desastre evitado!

Toda vez que seu componente renderizar novamente com um `roomId` diferente, seu Efeito ressincronizará. Por exemplo, digamos que o usuário mude `roomId` de `"travel"` para `"music"`. O React novamente **parará a sincronização** do seu Efeito chamando sua função de limpeza (desconectando você da sala `"travel"`). Em seguida, ele **iniciará a sincronização** novamente executando seu corpo com a nova prop `roomId` (conectando você à sala `"music"`).

Finalmente, quando o usuário for para uma tela diferente, `ChatRoom` desmontará. Agora não há necessidade de permanecer conectado. O React **parará a sincronização** do seu Efeito uma última vez e desconectará você da sala de chat `"music"`.

### Pensando da perspectiva do Efeito {/*thinking-from-the-effects-perspective*/}

Vamos recapitular tudo o que aconteceu da perspectiva do componente `ChatRoom`:

1. `ChatRoom` montou com `roomId` definido como `"general"`
1. `ChatRoom` atualizou com `roomId` definido como `"travel"`
1. `ChatRoom` atualizou com `roomId` definido como `"music"`
1. `ChatRoom` desmontou

Durante cada um desses pontos no ciclo de vida do componente, seu Efeito fez coisas diferentes:

1. Seu Efeito conectou-se à sala `"general"`
1. Seu Efeito desconectou-se da sala `"general"` e conectou-se à sala `"travel"`
1. Seu Efeito desconectou-se da sala `"travel"` e conectou-se à sala `"music"`
1. Seu Efeito desconectou-se da sala `"music"`

Agora vamos pensar sobre o que aconteceu da perspectiva do próprio Efeito:

```js
  useEffect(() => {
    // Seu Efeito conectou-se à sala especificada com roomId...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...até que se desconectou
      connection.disconnect();
    };
  }, [roomId]);
```

A estrutura deste código pode inspirá-lo a ver o que aconteceu como uma sequência de períodos de tempo não sobrepostos:

1. Seu Efeito conectou-se à sala `"general"` (até que se desconectou)
1. Seu Efeito conectou-se à sala `"travel"` (até que se desconectou)
1. Seu Efeito conectou-se à sala `"music"` (até que se desconectou)

Anteriormente, você estava pensando da perspectiva do componente. Quando você olhava da perspectiva do componente, era tentador pensar em Efeitos como "callbacks" ou "eventos de ciclo de vida" que disparam em um momento específico, como "após uma renderização" ou "antes de desmontar". Essa forma de pensar fica complicada muito rapidamente, então é melhor evitá-la.

**Em vez disso, concentre-se sempre em um único ciclo de início/parada por vez. Não deve importar se um componente está montando, atualizando ou desmontando. Tudo o que você precisa fazer é descrever como iniciar a sincronização e como pará-la. Se você fizer isso bem, seu Efeito será resiliente a ser iniciado e parado quantas vezes for necessário.**

Isso pode lembrá-lo de como você não pensa se um componente está montando ou atualizando quando escreve a lógica de renderização que cria JSX. Você descreve o que deve estar na tela, e o React [descobre o resto.](/learn/reacting-to-input-with-state)

### Como o React verifica se seu Efeito pode ressincronizar {/*how-react-verifies-that-your-effect-can-re-synchronize*/}

Aqui está um exemplo interativo que você pode experimentar. Pressione "Open chat" para montar o componente `ChatRoom`:

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

Observe que quando o componente monta pela primeira vez, você vê três logs:

1. `✅ Connecting to "general" room at https://localhost:1234...` *(apenas em desenvolvimento)*
1. `❌ Disconnected from "general" room at https://localhost:1234.` *(apenas em desenvolvimento)*
1. `✅ Connecting to "general" room at https://localhost:1234...`

Os dois primeiros logs são apenas em desenvolvimento. Em desenvolvimento, o React sempre remonta cada componente uma vez.

**O React verifica se seu Efeito pode ressincronizar forçando-o a fazer isso imediatamente em desenvolvimento.** Isso pode lembrá-lo de abrir uma porta e fechá-la uma vez extra para verificar se a fechadura da porta funciona. O React inicia e para seu Efeito uma vez extra em desenvolvimento para verificar se [você implementou sua limpeza bem.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

A principal razão pela qual seu Efeito ressincronizará na prática é se alguns dados que ele usa mudaram. Na sandbox acima, altere a sala de chat selecionada. Observe como, quando o `roomId` muda, seu Efeito ressincroniza.

No entanto, também existem casos mais incomuns em que a ressincronização é necessária. Por exemplo, tente editar o `serverUrl` na sandbox acima enquanto o chat está aberto. Observe como o Efeito ressincroniza em resposta às suas edições no código. No futuro, o React pode adicionar mais recursos que dependem da ressincronização.

### Como o React sabe que precisa ressincronizar o Efeito {/*how-react-knows-that-it-needs-to-re-synchronize-the-effect*/}

Você pode estar se perguntando como o React sabia que seu Efeito precisava ressincronizar após a mudança de `roomId`. É porque *você disse ao React* que seu código depende de `roomId` incluindo-o na [lista de dependências:](/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)

```js {1,3,8}
function ChatRoom({ roomId }) { // A prop roomId pode mudar com o tempo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Este Efeito lê roomId 
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Então você diz ao React que este Efeito "depende de" roomId
  // ...
```

Veja como isso funciona:

1. Você sabia que `roomId` é uma prop, o que significa que ela pode mudar com o tempo.
2. Você sabia que seu Efeito lê `roomId` (então sua lógica depende de um valor que pode mudar mais tarde).
3. É por isso que você o especificou como dependência do seu Efeito (para que ele ressincronize quando `roomId` mudar).

Toda vez que seu componente renderizar novamente, o React olhará para o array de dependências que você passou. Se algum dos valores no array for diferente do valor no mesmo local que você passou durante a renderização anterior, o React ressincronizará seu Efeito.

Por exemplo, se você passou `["general"]` durante a renderização inicial e, mais tarde, passou `["travel"]` durante a próxima renderização, o React comparará `"general"` e `"travel"`. Estes são valores diferentes (comparados com [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), então o React ressincronizará seu Efeito. Por outro lado, se seu componente renderizar novamente, mas `roomId` não mudou, seu Efeito permanecerá conectado à mesma sala.

### Cada Efeito representa um processo de sincronização separado {/*each-effect-represents-a-separate-synchronization-process*/}

Resista a adicionar lógica não relacionada ao seu Efeito apenas porque essa lógica precisa ser executada ao mesmo tempo que um Efeito que você já escreveu. Por exemplo, digamos que você queira enviar um evento de análise quando o usuário visita a sala. Você já tem um Efeito que depende de `roomId`, então você pode se sentir tentado a adicionar a chamada de análise lá:

```js {3}
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () =>