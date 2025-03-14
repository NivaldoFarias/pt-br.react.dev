```js
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

Here's how it works:

*   **Without dependencies:** `useEffect` always runs after every render.
*   **With empty dependencies (`[]`):** `useEffect` runs *only* after the first render.
*   **With dependencies (`[dep1, dep2, dep3]`):** `useEffect` runs after the first render, and after any subsequent renders if `dep1`, `dep2`, or `dep3` are different from what they were during the last render. React compares each dependency value with its value from the *previous* render.

In other words, **dependencies tell React to re-run the Effect if some values that the Effect *uses* have changed since the last render.**

Here are some additional examples of dependency arrays:

```js
// ✅ Effect runs only when the component *mounts* (appears on the screen)
useEffect(() => {
  // ...
}, []);

// ✅ Effect runs when the component mounts, and every time `tabId` changes
useEffect(() => {
  // ...
}, [tabId]);

// ✅ Effect runs when the component mounts, and every time `tabId` or `postId` changes
useEffect(() => {
  // ...
}, [tabId, postId]);
```

### Step 3: Add cleanup if needed {/*step-3-add-cleanup-if-needed*/}

Some Effects may require additional code to *clean up* after they have run. For example, if you're setting up a connection, you need to close it. If you subscribe to something, you need to unsubscribe. To do this, **your Effect can optionally return a *cleanup function*.**

Here's how it works. React will:

1.  Run your Effect.
2.  If your Effect returns a function, React will save that function.
3.  Before running the Effect again, React will run the cleanup function from the previous run.
4.  After the final render, React will run the cleanup function from the last run.

Here is an example of an Effect that sets up a chat connection when a component mounts (appears on the screen) and disconnects when the component unmounts (disappears).

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
  return <h1>Welcome to room: {roomId}</h1>;
}

function createConnection(roomId) {
  // A mock implementation
  return {
    connect() {
      console.log('✅ Connecting to ' + roomId + '...');
    },
    disconnect() {
      console.log('❌ Disconnecting from ' + roomId);
    },
  };
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={(e) => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

In this example:

*   The `ChatRoom` component receives a `roomId` prop.
*   The `useEffect` Hook sets up a chat connection in the Effect.
*   It returns a cleanup function that disconnects the connection.
*   The dependencies `[roomId]` ensure the effect only re-connects if `roomId` changes.

Try switching to a different chat room in the example above. You'll see that React disconnects from the previous room and connects to the new one. When you remove `ChatRoom` from the screen (for example, by conditionally rendering it inside the `App` component), you'll see that it also disconnects.

<Note>

The cleanup function runs *before* the Effect runs the next time, and after the last render.

</Note>

## Running Effects twice in development {/*running-effects-twice-in-development*/}

To help you write Effects correctly, **React calls some of your Effect functions twice in development.** This is because Effects, by their nature, depend on an external system. This can easily lead to bugs if you're not careful.

For example, consider this `ChatRoom` component:

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
  return <h1>Welcome to room: {roomId}</h1>;
}
```

If `connection.connect()` accidentally sends a "join" message to the existing chat room, and `connection.disconnect()` accidentally sends a "leave" message, then the same "join" and "leave" events would be triggered, causing a problem. If your logic is implemented incorrectly, an extra message would be printed or sent.

To find these kinds of bugs during development, React intentionally does the following:

1.  **Runs your mount Effect.** For example, it connects to the chat room.
2.  **Runs the cleanup function.** For example, it disconnects from the chat room.
3.  **Runs your mount Effect again.** For example, it connects to the chat room for a second time.

This means that your Effect logic must be able to handle being mounted, torn down, and mounted again. If the cleanup function is missing, or if the Effect doesn't handle being "re-mounted", this will reveal subtle bugs.

**In production, this will not happen, so your app will not behave like this in production mode**. In production, React will run each Effect only once, with no cleanup.

## How to fix the double Effect calls {/*how-to-fix-the-double-effect-calls*/}

You should review your code to make sure your Effects are idempotent. An effect is *idempotent* when it runs multiple times without causing problems, just as if it ran a single time.

For example, this Effect is idempotent because multiple calls to `setFriendList` with the same value are safe:

```js
useEffect(() => {
  setFriendList(myInitialFriendList);
}, []);
```

However, this Effect isn't safe:

```js
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log('⏱️ Tick');
  }, 1000);
  return () => {
    clearInterval(intervalId);
  };
}, []);
```

If you run this effect twice, you'll end up with two intervals running in parallel, causing problems. To prevent this, you need to make sure that the Effect can tolerate being run multiple times. In this case, you might need to keep the `intervalId ` and `clearInterval` outside of the Effect's function body:

```js
let intervalId = null;
useEffect(() => {
  intervalId = setInterval(() => {
    console.log('⏱️ Tick');
  }, 1000);
  return () => {
    clearInterval(intervalId);
  };
}, []);
```

Alternatively, you might be able to avoid the Effect altogether:

```js
// ✅ You don't need an effect here
setInterval(() => {
  console.log('⏱️ Tick');
}, 1000);
```

In general, when you find that your Effect runs multiple times, here are some actions to take:

*   **If the effect is idempotent,** that means it can run multiple times safely. This is the case if you're just setting a state to some initial value, and it's fine how often that happens.
*   **If the Effect manages an external resource:** If the effect creates and tears something down, make sure the teardown logic correctly reverses the operations that the effect performed.
*   **If the Effect isn't idempotent:** Try to make it idempotent. Often, the problem occurs because the `subscribe` and `unsubscribe` calls do not match.
*   **If the double calls are a serious problem:** If the double calls make it difficult to use development builds, you can disable them. However, this is not recommended, because it will hide the bugs described above.

```js
// Exemplo
import { useState, useEffect } from 'react';

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

O array de dependência pode conter múltiplas dependências. o React irá pular a re-execução do Effect apenas se *todas* as dependências que você especificar tiverem exatamente os mesmos valores que tinham durante a renderização anterior. o React compara os valores da dependência usando a comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Veja a [`useEffect` reference](/reference/react/useEffect#reference) para detalhes.

**Perceba que você não pode "escolher" suas dependências.** Você receberá um erro de lint se as dependências que você especificar não corresponderem ao que o o React espera com base no código dentro do seu Effect. Isso ajuda a detectar muitos erros no seu código. Se você não quer que algum código seja re-executado, [*edite o próprio código do Effect* para que não "precise" dessa dependência.](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

<Pitfall>

Os comportamentos sem o array de dependência e com um array de dependência `[]` *vazio* são diferentes:

```js {3,7,11}
useEffect(() => {
  // Isto roda após toda renderização
});

useEffect(() => {
  // Isto roda apenas no mount (quando o componente aparece)
}, []);

useEffect(() => {
  // Isto roda no mount *e também* se a ou b mudaram desde a última renderização
}, [a, b]);
```

Nós estaremos dando uma boa olhada no que "mount" significa no próximo passo.

</Pitfall>

<DeepDive>

#### Por que a ref foi omitida do array de dependência? {/*why-was-the-ref-omitted-from-the-dependency-array*/}

Este Effect usa _ambos_ `ref` e `isPlaying`, mas apenas `isPlaying` é declarada como uma dependência:

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

Isso é porque o objeto `ref` tem uma *identidade estável:* o React garante que [você sempre receberá o mesmo objeto](/reference/react/useRef#returns) da mesma chamada `useRef` em cada renderização. Ele nunca muda, então ele nunca, por si só, fará o Effect re-executar. Portanto, não importa se você o inclui ou não. Incluí-lo também é bom:

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

As funções [`set`](/reference/react/useState#setstate) retornadas por `useState` também têm uma identidade estável, então você frequentemente as verá omitidas das dependências também. Se o linter permitir que você omita uma dependência sem erros, é seguro fazê-lo.

Omitir dependências sempre-estáveis só funciona quando o linter pode "ver" que o objeto é estável. Por exemplo, se `ref` fosse passado a partir de um componente pai, você teria que especificá-lo no array de dependência. Contudo, isto é bom porque você não pode saber se o componente pai sempre passa a mesma ref, ou passa uma de várias refs condicionalmente. Então o seu Effect _dependeria_ de qual ref é passada.

</DeepDive>

### Passo 3: Adicione cleanup se necessário {/*step-3-add-cleanup-if-needed*/}

Considere um exemplo diferente. Você está escrevendo um componente `ChatRoom` que precisa conectar-se ao servidor de chat quando ele aparece. Você recebe uma API `createConnection()` que retorna um objeto com métodos `connect()` e `disconnect()`. Como você mantém o componente conectado enquanto ele é mostrado ao usuário?

Comece escrevendo a lógica do Effect:

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Seria lento conectar ao chat após toda re-renderização, então você adiciona o array de dependência:

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**O código dentro do Effect não usa quaisquer props ou state, então seu array de dependência é `[]` (vazio). Isso diz ao React para apenas rodar este código quando o componente "monta", ou seja, aparece na tela pela primeira vez.**

Vamos tentar rodar este código:

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
  // Uma implementação de verdade realmente se conectaria ao servidor
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

Este Effect roda apenas no mount, então você pode esperar que o `"✅ Conectando..."` seja impresso uma vez no console. **No entanto, se você verificar o console, o `"✅ Conectando..."` é impresso duas vezes. Por que isso acontece?**

Imagine que o componente `ChatRoom` é parte de uma aplicação maior com muitas telas diferentes. O usuário começa sua jornada na página `ChatRoom`. O componente monta e chama `connection.connect()`. Então imagine que o usuário navega para outra tela -- por exemplo, para a página de Configurações. O componente `ChatRoom` desmonta. Finalmente, o usuário clica em Voltar e `ChatRoom` monta novamente. Isso configuraria uma segunda conexão -- mas a primeira conexão nunca foi destruída! Conforme o usuário navega pela aplicação, as conexões continuariam aumentando.

Erros como esse são fáceis de perder sem testes manuais extensivos. Para ajudá-lo a detectá-los rapidamente, em desenvolvimento, o React remonta cada componente uma vez imediatamente após seu mount inicial.

Ver o log `"✅ Conectando..."` duas vezes ajuda você a perceber o problema real: seu código não fecha a conexão quando o componente desmonta.

Para corrigir o problema, retorne uma *função de cleanup* do seu Effect:

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

o React irá chamar sua função de cleanup cada vez antes que o Effect rode novamente, e uma última vez quando o componente desmonta (é removido). Vamos ver o que acontece quando a função de cleanup é implementada:

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
  // Uma implementação de verdade realmente se conectaria ao servidor
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

Agora você recebe três logs no console em desenvolvimento:

1. `"✅ Conectando..."`
2. `"❌ Desconectado."`
3. `"✅ Conectando..."`

**Este é o comportamento correto em desenvolvimento.** Ao remountar seu componente, o o React verifica que navegar para fora e para trás não quebrará o seu código. Desconectar e então conectar novamente é exatamente o que deveria acontecer! Quando você implementa bem o cleanup, não deveria haver nenhuma diferença visível ao usuário entre rodar o Effect uma vez vs rodá-lo, limpá-lo, e rodá-lo novamente. Há um par extra de chamada connect/disconnect porque o React está sondando seu código por erros em desenvolvimento. Isso é normal -- não tente fazer isso desaparecer!

**Em produção, você veria apenas `"✅ Conectando..."` impresso uma vez.** Remountar componentes só acontece em desenvolvimento para ajudá-lo a encontrar Effects que precisam de cleanup. Você pode desativar [Strict Mode](/reference/react/StrictMode) para não participar do comportamento de desenvolvimento, mas nós recomendamos mantê-lo ativado. Isso permite que você encontre muitos erros como o acima.

## Como lidar com o Effect disparando duas vezes em desenvolvimento? {/*how-to-handle-the-effect-firing-twice-in-development*/}

o React intencionalmente remonta seus componentes em desenvolvimento para encontrar erros como no último exemplo. **A pergunta correta não é "como rodar o Effect uma vez", mas "como corrigir meu Effect para que ele funcione após o remount".**

Geralmente, a resposta é implementar a função de cleanup. A função de cleanup deveria parar ou desfazer o que o Effect estava fazendo. A regra geral é que o usuário não deveria ser capaz de distinguir entre o Effect rodando uma vez (como em produção) e uma sequência _setup → cleanup → setup_ (como você veria em desenvolvimento).

A maioria dos Effects que você irá escrever se encaixará em um dos padrões comuns abaixo.

<Pitfall>

#### Não use refs para prevenir Effects de dispararem {/*dont-use-refs-to-prevent-effects-from-firing*/}

Uma armadilha comum para prevenir Effects de dispararem duas vezes em desenvolvimento é usar uma `ref` para prevenir que o Effect rode mais de uma vez. Por exemplo, você pode "corrigir" o erro acima com um `useRef`:

```js {1,3-4}
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 Isso NÃO vai corrigir o erro!!!
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

Isso faz com que você veja `"✅ Conectando..."` apenas uma vez em desenvolvimento, mas não corrige o erro.

Quando o usuário navega para fora, a conexão ainda não é fechada e quando ele navega para trás, uma nova conexão é criada. Conforme o usuário navega pela aplicação, as conexões continuariam aumentando, da mesma forma que aconteceria antes da "correção".

Para corrigir o erro, não é suficiente apenas fazer com que o Effect rode uma vez. O effect precisa funcionar após o remount, o que significa que a conexão precisa ser limpa como na solução acima.

Veja os exemplos abaixo para como lidar com padrões comuns.

</Pitfall>

### Controlando widgets não-React {/*controlling-non-react-widgets*/}

Às vezes você precisa adicionar widgets de UI que não são escritos em React. Por exemplo, vamos dizer que você está adicionando um componente de mapa à sua página. Ele tem um método `setZoomLevel()`, e você gostaria de manter o nível do zoom em sincronia com uma variável de estado `zoomLevel` no seu código React. Seu Effect iria se parecer com isto:

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Note que não há necessidade de cleanup neste caso. Em desenvolvimento, o React chamará o Effect duas vezes, mas isso não é um problema porque chamar `setZoomLevel` duas vezes com o mesmo valor não faz nada. Pode ser um pouco mais lento, mas isso não importa porque ele não irá remountar desnecessariamente em produção.

Algumas APIs podem não permitir que você as chame duas vezes seguidas. Por exemplo, o método [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) do elemento integrado [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) lança um erro se você o chama duas vezes. Implemente a função de cleanup e faça com que ela feche o diálogo:

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

Em desenvolvimento, seu Effect irá chamar `showModal()`, então imediatamente `close()`, e então `showModal()` novamente. Isso tem o mesmo comportamento visível ao usuário que chamar `showModal()` uma vez, como você veria em produção.

### Assinando eventos {/*subscribing-to-events*/}

Se seu Effect assina algo, a função de cleanup deve cancelar a assinatura:

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

Em desenvolvimento, seu Effect irá chamar `addEventListener()`, então imediatamente `removeEventListener()`, e então `addEventListener()` novamente com o mesmo manipulador. Então haverá apenas uma assinatura ativa por vez. Isso tem o mesmo comportamento visível ao usuário que chamar `addEventListener()` uma vez, como em produção.

### Acionando animações {/*triggering-animations*/}

Se seu Effect anima algo, a função de cleanup deve resetar a animação para os valores iniciais:

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Dispara a animação
  return () => {
    node.style.opacity = 0; // Reseta para o valor inicial
  };
}, []);
```

Em desenvolvimento, opacity será definida para `1`, então para `0`, e então para `1` novamente. Isso deveria ter o mesmo comportamento visível ao usuário que definir para `1` diretamente, o que é o que aconteceria em produção. Se você usar uma biblioteca de animação de terceiros com suporte para tweening, sua função de cleanup deve resetar a linha do tempo para seu estado inicial.

### Buscando dados {/*fetching-data*/}

Se seu Effect busca algo, a função de cleanup deve ou [abortar a fetch](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) ou ignorar seu resultado:

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

Você não pode "desfazer" uma requisição de rede que já aconteceu, mas sua função de cleanup deve garantir que a fetch que _não é mais relevante_ não continue afetando sua aplicação. Se o `userId` mudar de `'Alice'` para `'Bob'`, o cleanup garante que a resposta de `'Alice'` seja ignorada mesmo se ela chegar depois de `'Bob'`.

**Em desenvolvimento, você verá duas fetches na aba Network.** Não há nada de errado com isso. Com a abordagem acima, o primeiro Effect será imediatamente limpo então sua cópia da variável `ignore` será definida para `true`. Então mesmo que haja uma requisição extra, ela não afetará o estado graças à verificação `if (!ignore)`.

**Em produção, haverá apenas uma requisição.** Se a segunda requisição em desenvolvimento está incomodando, a melhor abordagem é usar uma solução que deduplica requisições e armazena em cache suas respostas entre os componentes:

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

Isso não apenas irá melhorar a experiência de desenvolvimento, mas também fará com que sua aplicação pareça mais rápida. Por exemplo, o usuário pressionando o botão Voltar não terá que esperar por alguns dados carregarem novamente porque eles estarão armazenados em cache. Você pode ou construir um cache como este sozinho ou usar uma das muitas alternativas para a busca manual em Effects.

<DeepDive>

#### Quais são boas alternativas para buscar dados em Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Escrever chamadas `fetch` dentro de Effects é uma [maneira popular de buscar dados](https://www.robinwieruch.de/react-hooks-fetch-data/), especialmente em aplicações totalmente no lado do cliente. Esta é, no entanto, uma abordagem muito manual e tem desvantagens significativas:
```text
- **Effects não são executados no servidor.** Isso significa que o HTML inicial renderizado no servidor só incluirá um estado de carregamento sem dados. O computador cliente terá que baixar todo o JavaScript e renderizar seu aplicativo apenas para descobrir que agora ele precisa carregar os dados. Isso não é muito eficiente.
- **Buscar diretamente em Effects facilita a criação de "cascatas de rede".** Você renderiza o componente pai, ele busca alguns dados, renderiza os componentes filhos e, em seguida, eles começam a buscar seus dados. Se a rede não for muito rápida, isso é significativamente mais lento do que buscar todos os dados em paralelo.
- **Buscar diretamente em Effects geralmente significa que você não pré-carrega ou armazena dados em cache.** Por exemplo, se o componente for desmontado e, em seguida, montado novamente, ele terá que buscar os dados novamente.
- **Não é muito ergonômico.** Há uma quantidade considerável de código boilerplate envolvido ao escrever chamadas `fetch` de uma forma que não sofra de bugs como [condições de corrida.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

Esta lista de desvantagens não é específica do React. Ela se aplica à busca de dados na montagem com qualquer biblioteca. Assim como com o roteamento, a busca de dados não é trivial de ser feita corretamente, por isso recomendamos as seguintes abordagens:

- **Se você usa um [framework](/learn/start-a-new-react-project#production-grade-react-frameworks), use seu mecanismo integrado de busca de dados.** Frameworks React modernos têm mecanismos integrados de busca de dados que são eficientes e não sofrem das armadilhas acima.
- **Caso contrário, considere usar ou criar um cache do lado do cliente.** Soluções de código aberto populares incluem [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/), e [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) Você pode construir sua própria solução também, caso em que você usaria efeitos em um nível inferior, mas adicionaria lógica para dedupicar solicitações, armazenar respostas em cache e evitar cascatas de rede (pré-carregando dados ou elevando os requisitos de dados para as rotas).

Você pode continuar buscando dados diretamente em Effects se nenhuma dessas abordagens for adequada para você.

</DeepDive>

### Enviando analytics {/*sending-analytics*/}

Considere este código que envia um evento de análise na visita à página:

```js
useEffect(() => {
  logVisit(url); // Envia uma requisição POST
}, [url]);
```

Em desenvolvimento, `logVisit` será chamado duas vezes para cada URL, então você pode ser tentado a tentar corrigir isso. **Recomendamos manter este código como está.** Como nos exemplos anteriores, não há diferença de comportamento *visível ao usuário* entre executá-lo uma vez e executá-lo duas vezes. Do ponto de vista prático, `logVisit` não deve fazer nada em desenvolvimento porque você não quer que os logs das máquinas de desenvolvimento distorçam as métricas de produção. Seu componente é remontado toda vez que você salva seu arquivo, então ele registra visitas extras em desenvolvimento de qualquer maneira.

**Em produção, não haverá logs de visita duplicados.**

Para depurar os eventos de análise que você está enviando, você pode implantar seu aplicativo em um ambiente de teste (que é executado no modo de produção) ou desabilitar temporariamente o [Modo Strict](/reference/react/StrictMode) e suas verificações de remounting apenas para desenvolvimento. Você também pode enviar análises dos manipuladores de eventos de alteração de rota em vez de Effects. Para análises mais precisas, [observadores de interseção](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) podem ajudar a rastrear quais componentes estão na viewport e por quanto tempo eles permanecem visíveis.

### Não é um Effect: Inicializando a aplicação {/*not-an-effect-initializing-the-application*/}

Alguma lógica deve ser executada apenas uma vez quando o aplicativo iniciar. Você pode colocá-la fora de seus componentes:

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

Às vezes, mesmo que você escreva uma função de limpeza, não há como evitar as consequências visíveis ao usuário de executar o Effect duas vezes. Por exemplo, talvez seu Effect envie uma requisição POST como comprar um produto:

```js {2-3}
useEffect(() => {
  // 🔴 Errado: Este Effect é acionado duas vezes em desenvolvimento, expondo um problema no código.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

Você não gostaria de comprar o produto duas vezes. No entanto, é também por isso que você não deve colocar essa lógica em um Effect. E se o usuário for para outra página e pressionar Voltar? Seu Effect seria executado novamente. Você não quer comprar o produto quando o usuário *visita* uma página; você quer comprá-lo quando o usuário *clica* no botão Comprar.

Comprar não é causado pela renderização; é causado por uma interação específica. Ele deve ser executado apenas quando o usuário pressiona o botão. **Exclua o Effect e mova sua solicitação `/api/buy` para o manipulador de eventos do botão Comprar:**

```js {2-3}
  function handleClick() {
    // ✅ Comprar é um evento porque é causado por uma interação específica.
    fetch('/api/buy', { method: 'POST' });
  }
```

**Isso ilustra que, se a remontagem quebrar a lógica do seu aplicativo, isso geralmente revela bugs existentes.** Do ponto de vista do usuário, visitar uma página não deve ser diferente de visitá-la, clicar em um link e, em seguida, pressionar Voltar para visualizar a página novamente. O React verifica se seus componentes obedecem a este princípio remontando-os uma vez em desenvolvimento.

## Reunindo tudo {/*putting-it-all-together*/}

Este playground pode ajudá-lo a "ter uma ideia" de como os Effects funcionam na prática.

Este exemplo usa [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) para agendar um log no console com o texto de entrada para aparecer três segundos após a execução do Effect. A função de limpeza cancela o tempo limite pendente. Comece pressionando "Montar o componente":

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Agendar "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancelar "' + text + '" log');
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

Você verá três logs no início: `Agendar "a" log`, `Cancelar "a" log` e `Agendar "a" log` novamente. Três segundos depois, também haverá um log dizendo `a`. Como você aprendeu anteriormente, o par extra de agendamento/cancelamento ocorre porque o React remonta o componente uma vez em desenvolvimento para verificar se você implementou a limpeza corretamente.

Agora edite a entrada para dizer `abc`. Se você fizer isso rápido o suficiente, verá `Agendar "ab" log` imediatamente seguido por `Cancelar "ab" log` e `Agendar "abc" log`. **O React sempre limpa o Effect da renderização anterior antes do Effect da renderização seguinte.** É por isso que, mesmo que você digite na entrada rapidamente, há no máximo um tempo limite agendado por vez. Edite a entrada algumas vezes e observe o console para ter uma ideia de como os Effects são limpos.

Digite algo na entrada e, em seguida, pressione imediatamente "Desmontar o componente". Observe como a desmontagem limpa o Effect da última renderização. Aqui, ele limpa o último tempo limite antes que ele tenha a chance de disparar.

Finalmente, edite o componente acima e comente a função de limpeza para que os tempos limites não sejam cancelados. Tente digitar `abcde` rapidamente. O que você espera que aconteça em três segundos? `console.log(text)` dentro do tempo limite imprimirá o `text` *mais recente* e produzirá cinco logs `abcde`? Tente para verificar sua intuição!

Três segundos depois, você deve ver uma sequência de logs (`a`, `ab`, `abc`, `abcd` e `abcde`) em vez de cinco logs `abcde`. **Cada Effect "captura" o valor `text` de sua renderização correspondente.** Não importa que o estado `text` tenha mudado: um Effect da renderização com `text = 'ab'` sempre verá `'ab'`. Em outras palavras, os Effects de cada renderização são isolados uns dos outros. Se você está curioso sobre como isso funciona, você pode ler sobre [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures).

<DeepDive>

#### Cada renderização tem seus próprios Effects {/*each-render-has-its-own-effects*/}

Você pode pensar em `useEffect` como "anexar" um pedaço de comportamento à saída da renderização. Considere este Effect:

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Bem-vindo ao {roomId}!</h1>;
}
```

Vamos ver exatamente o que acontece à medida que o usuário navega pelo aplicativo.

#### Renderização inicial {/*initial-render*/}

O usuário visita `<ChatRoom roomId="general" />`. Vamos [substituir mentalmente](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` com `'general'`:

```js
  // JSX para a primeira renderização (roomId = "general")
  return <h1>Bem-vindo ao general!</h1>;
```

**O Effect é *também* parte da saída da renderização.** O Effect da primeira renderização se torna:

```js
  // Effect para a primeira renderização (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the first render (roomId = "general")
  ['general']
```

O React executa este Effect, que se conecta à sala de bate-papo `'general'`.

#### Re-renderizar com as mesmas dependências {/*re-render-with-same-dependencies*/}

Digamos que `<ChatRoom roomId="general" />` renderize novamente. A saída JSX é a mesma:

```js
  // JSX para a segunda renderização (roomId = "general")
  return <h1>Bem-vindo ao general!</h1>;
```

O React vê que a saída da renderização não mudou, então não atualiza o DOM.

O Effect da segunda renderização se parece com isto:

```js
  // Effect para a segunda renderização (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the second render (roomId = "general")
  ['general']
```

O React compara `['general']` da segunda renderização com `['general']` da primeira renderização. **Como todas as dependências são as mesmas, o React *ignora* o Effect da segunda renderização.** Ele nunca é chamado.

#### Re-renderizar com dependências diferentes {/*re-render-with-different-dependencies*/}

Em seguida, o usuário visita `<ChatRoom roomId="travel" />`. Desta vez, o componente retorna JSX diferente:

```js
  // JSX para a terceira renderização (roomId = "travel")
  return <h1>Bem-vindo ao travel!</h1>;
```

O React atualiza o DOM para mudar `"Bem-vindo ao general"` para `"Bem-vindo ao travel"`.

O Effect da terceira renderização se parece com isto:

```js
  // Effect para a terceira renderização (roomId = "travel")
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the third render (roomId = "travel")
  ['travel']
```

O React compara `['travel']` da terceira renderização com `['general']` da segunda renderização. Uma dependência é diferente: `Object.is('travel', 'general')` é `false`. O Effect não pode ser ignorado.

**Antes que o React possa aplicar o Effect da terceira renderização, ele precisa limpar o último Effect que _foi_ executado.** O Effect da segunda renderização foi ignorado, então o React precisa limpar o Effect da primeira renderização. Se você rolar para cima até a primeira renderização, verá que sua limpeza chama `disconnect()` na conexão que foi criada com `createConnection('general')`. Isso desconecta o aplicativo da sala de bate-papo `'general'`.

Depois disso, o React executa o Effect da terceira renderização. Ele se conecta à sala de bate-papo `'travel'`.

#### Desmontar {/*unmount*/}

Finalmente, digamos que o usuário navegue para longe e o componente `ChatRoom` seja desmontado. O React executa a função de limpeza do último Effect. O último Effect foi da terceira renderização. A limpeza da terceira renderização destrói a conexão `createConnection('travel')`. Então, o aplicativo se desconecta da sala `'travel'`.

#### Comportamentos apenas para desenvolvimento {/*development-only-behaviors*/}

Quando o [Modo Strict](/reference/react/StrictMode) está ativado, o React remonta cada componente uma vez após a montagem (estado e DOM são preservados). Isso [ajuda a encontrar Effects que precisam de limpeza](#step-3-add-cleanup-if-needed) e expõe bugs como condições de corrida no início. Além disso, o React remontará os Effects sempre que você salvar um arquivo em desenvolvimento. Ambos esses comportamentos são apenas para desenvolvimento.

</DeepDive>

<Recap>

- Ao contrário dos eventos, os Effects são causados pela própria renderização, em vez de uma interação específica.
- Effects permitem que você sincronize um componente com algum sistema externo (API de terceiros, rede, etc).
- Por padrão, os Effects são executados após cada renderização (incluindo a inicial).
- O React ignorará o Effect se todas as suas dependências tiverem os mesmos valores que durante a última renderização.
- Você não pode "escolher" suas dependências. Elas são determinadas pelo código dentro do Effect.
- Uma matriz de dependência vazia (`[]`) corresponde à "montagem" do componente, ou seja, sendo adicionado à tela.
- No Modo Strict, o React monta os componentes duas vezes (apenas em desenvolvimento!) para testar seus Effects.
- Se seu Effect quebrar por causa da remontagem, você precisa implementar uma função de limpeza.
- O React chamará sua função de limpeza antes que o Effect seja executado na próxima vez e durante a desmontagem.

</Recap>

<Challenges>

#### Focar um campo na montagem {/*focus-a-field-on-mount*/}

Neste exemplo, o formulário renderiza um componente `<MyInput />`.

Use o método [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) da entrada para fazer com que `MyInput` foque automaticamente quando aparecer na tela. Já existe uma implementação comentada, mas ela não funciona totalmente. Descubra por que não funciona e corrija-o. (Se você estiver familiarizado com o atributo `autoFocus`, finja que ele não existe: estamos reimplementando a mesma funcionalidade do zero.)

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO: Isso não funciona totalmente. Corrija-o.
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
            Colocar em caixa alta
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

Para verificar que sua solução funciona, pressione "Mostrar formulário" e verifique se a entrada recebe foco (fica realçada e o cursor é colocado dentro dela). Pressione "Ocultar formulário" e "Mostrar formulário" novamente. Verifique se a entrada é realçada novamente.

`MyInput` deve focar apenas _no monte_ e não após cada renderização. Para verificar se o comportamento está correto, pressione "Mostrar formulário" e, em seguida, pressione repetidamente a caixa de seleção "Colocar em caixa alta". Clicar na caixa de seleção _não_ deve focar a entrada acima dela.

<Solution>

Chamar `ref.current.focus()` durante a renderização é errado porque é um *efeito colateral*. Efeitos colaterais devem ser colocados dentro de um manipulador de eventos ou ser declarado com `useEffect`. Nesse caso, o efeito colateral é _causado_ pelo componente aparecer, em vez de qualquer interação específica, então faz sentido colocá-lo em um Effect.

Para corrigir o erro, envolva a chamada `ref.current.focus()` em uma declaração Effect. Em seguida, para garantir que esse Effect seja executado apenas no monte, em vez de após cada renderização, adicione as dependências `[]` vazias a ele.

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
            Colocar em caixa alta
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

Este formulário renderiza dois componentes `<MyInput />`.

Pressione "Mostrar formulário" e observe que o segundo campo recebe foco automaticamente. Isso ocorre porque ambos os componentes `<MyInput />` tentam focar o campo dentro. Quando você chama `focus()` para dois campos de entrada em uma sequência, o último sempre "vence".

Digamos que você queira focar o primeiro campo. O primeiro componente `MyInput` agora recebe uma prop booleana `shouldFocus` definida como `true`. Altere a lógica para que `focus()` seja chamado somente se a prop `shouldFocus` recebida por `MyInput` for `true`.

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

Para verificar sua solução, pressione "Mostrar formulário" e "Ocultar formulário" repetidamente. Quando o formulário aparecer, apenas a *primeira* entrada deve receber foco. Isso ocorre porque o componente pai renderiza a primeira entrada com `shouldFocus={true}` e a segunda entrada com `shouldFocus={false}`. Verifique também se ambas as entradas ainda funcionam e se você pode digitar nas duas.

<Hint>

Você não pode declarar um Effect condicionalmente, mas seu Effect pode incluir lógica condicional.

</Hint>

<Solution>

Coloque a lógica condicional dentro do Effect. Você precisará especificar `shouldFocus` como uma dependência, porque está usando-a dentro do Effect. (Isso significa que se o `shouldFocus` de alguma entrada mudar de `false` para `true`, ele focará após a montagem.)

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

#### Corrigir um intervalo que é disparado duas vezes {/*fix-an-interval-that-fires-twice*/}

Este componente `Contador` exibe um contador que deve ser incrementado a cada segundo. No monte, ele chama [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) Isso faz com que `onTick` seja executado a cada segundo. A função `onTick` incrementa o contador.

No entanto, em vez de incrementar uma vez por segundo, ele incrementa duas vezes. Por que isso acontece? Encontre a causa do erro e corrija-o.

<Hint>

Tenha em mente que `setInterval` retorna um ID de intervalo, que você pode passar para [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) para parar o intervalo.

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

Quando o [Modo Restrito](/reference/react/StrictMode) está ativado (como nos sandboxes deste site), o React remonta cada componente uma vez no desenvolvimento. Isso faz com que o intervalo seja configurado duas vezes e é por isso que a cada segundo o contador é incrementado duas vezes.

No entanto, o comportamento do React não é a *causa* do erro: o erro já existe no código. O comportamento do React torna o erro mais perceptível. A verdadeira causa é que este Effect inicia um processo, mas não fornece uma maneira de limpá-lo.

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

No desenvolvimento, o React ainda remontará seu componente uma vez para verificar se você implementou a limpeza corretamente. Portanto, haverá uma chamada `setInterval`, seguida imediatamente por `clearInterval` e `setInterval` novamente. Na produção, haverá apenas uma chamada `setInterval`. O comportamento visível pelo usuário em ambos os casos é o mesmo: o contador incrementa uma vez por segundo.

</Solution>

#### Corrigir a busca dentro de um Effect {/*fix-fetching-inside-an-effect*/}

Este componente mostra a biografia da pessoa selecionada. Ele carrega a biografia chamando uma função assíncrona `fetchBio(pessoa)` na montagem e sempre que `pessoa` muda. Essa função assíncrona retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que eventualmente resolve para uma string. Quando a busca for concluída, ela chama `setBio` para exibir essa string sob a caixa de seleção.

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
      <p><i>{bio ?? 'Carregando...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('Esta é a biografia de ' + person + '.');
    }, delay);
  })
}

```

</Sandpack>

Existe um erro neste código. Comece selecionando "Alice". Em seguida, selecione "Bob" e, imediatamente após isso, selecione "Taylor". Se você fizer isso rápido o suficiente, notará o erro: Taylor está selecionado, mas o parágrafo abaixo diz "Esta é a biografia de Bob".

Por que isso acontece? Corrija o erro dentro deste Effect.

<Hint>

Se um Effect busca algo assincronamente, geralmente precisa de limpeza.

</Hint>

<Solution>

Para acionar o erro, as coisas precisam acontecer nesta ordem:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')`
- **Buscar `'Taylor'` é concluído *antes* de buscar `'Bob'`**
- O Effect da renderização de `'Taylor'` chama `setBio('Esta é a biografia de Taylor.')`
- Busca `'Bob'` concluída
- O Effect da renderização de `'Bob'` chama `setBio('Esta é a biografia de Bob.')`

É por isso que você vê a biografia de Bob, mesmo que Taylor esteja selecionado. Bugs como esse são chamados de [condições de corrida](https://pt.wikipedia.org/wiki/Condi%C3%A7%C3%A3o_de_corrida) porque duas operações assíncronas estão "correndo" uma com a outra e podem chegar em uma ordem inesperada.

Para corrigir essa condição de corrida, adicione uma função de limpeza:

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
      <p><i>{bio ?? 'Carregando...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('Esta é a biografia de ' + person + '.');
    }, delay);
  })
}

```

</Sandpack>

O Effect de cada renderização tem sua própria variável `ignore`. Inicialmente, a variável `ignore` está definida como `false`. No entanto, se um Effect for limpo (como quando você seleciona uma pessoa diferente), sua variável `ignore` se tornará `true`. Portanto, agora não importa em que ordem as solicitações são concluídas. Apenas o Effect da última pessoa terá `ignore` definido como `false`, então ele chamará `setBio(result)`. Os Effects anteriores foram limpos, portanto, a verificação `if (!ignore)` impedirá que chamem `setBio`:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')` **e limpa o Effect anterior (de Bob)**
- Buscar `'Taylor'` é concluído *antes* de buscar `'Bob'`
- O Effect da renderização  de `'Taylor'` chama `setBio('Esta é a biografia de Taylor.')`
- Busca `'Bob'` concluída
- O Effect da renderização `'Bob'` **não faz nada porque seu sinalizador `ignore` foi definido como `true`**

Além de ignorar o resultado de uma chamada de API desatualizada, você também pode usar [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) para cancelar as solicitações que não são mais necessárias. No entanto, por si só, isso não é suficiente para proteger contra condições de corrida. Mais etapas assíncronas podem ser encadeadas após a busca, portanto, usar um sinalizador explícito como `ignore` é a maneira mais confiável de corrigir esse tipo de problema.

</Solution>

</Challenges>
```