---
title: 'Sincronizando com Effects'
---

<Intro>

Alguns componentes precisam sincronizar com sistemas externos. Por exemplo, você pode querer controlar um componente não React baseado no estado do React, configurar uma conexão com o servidor ou enviar um log de análise quando um componente aparece na tela. *Effects* permitem que você execute algum código após a renderização para que você possa sincronizar seu componente com algum sistema fora do React.

</Intro>

<YouWillLearn>

- O que são Effects
- Como Effects são diferentes de eventos
- Como declarar um Effect em seu componente
- Como pular a execução de um Effect desnecessariamente
- Por que Effects são executados duas vezes em desenvolvimento e como corrigi-los

</YouWillLearn>

## O que são Effects e como eles são diferentes de eventos? {/*what-are-effects-and-how-are-they-different-from-events*/}

Antes de chegar aos Effects, você precisa estar familiarizado com dois tipos de lógica dentro dos componentes React:

- **Código de renderização** (introduzido em [Descrevendo a UI](/learn/describing-the-ui)) vive no nível superior do seu componente. É aqui que você pega as props e o estado, as transforma e retorna o JSX que você quer ver na tela. [Código de renderização deve ser puro.](/learn/keeping-components-pure) Como uma fórmula matemática, ele deve somente _calcular_ o resultado, mas não fazer mais nada.

- **Manipuladores de eventos** (introduzidos em [Adicionando Interatividade](/learn/adding-interactivity)) são funções aninhadas dentro de seus componentes que *fazem* coisas ao invés de apenas calculá-las. Um manipulador de eventos pode atualizar um campo de entrada, enviar uma solicitação HTTP POST para comprar um produto ou navegar o usuário para outra tela. Manipuladores de eventos contêm ["efeitos colaterais"](https://pt.wikipedia.org/wiki/Efeito_colateral_(ci%C3%AAncia_da_computa%C3%A7%C3%A3o)) (eles alteram o estado do programa) causados por uma ação específica do usuário (por exemplo, um clique de botão ou digitação).

Às vezes, isso não é suficiente. Considere um componente `ChatRoom` que deve se conectar ao servidor de chat sempre que estiver visível na tela. Conectar-se a um servidor não é um cálculo puro (é um efeito colateral), então isso não pode acontecer durante a renderização. No entanto, não há nenhum evento específico como um clique que faça com que `ChatRoom` seja exibido.

***Effects* permitem que você especifique efeitos colaterais que são causados pela própria renderização, em vez de por um evento em particular.** Enviar uma mensagem no chat é um *evento* porque é diretamente causado pelo usuário clicar em um botão específico. No entanto, configurar uma conexão de servidor é um *Effect* porque ele deve acontecer, não importa qual interação fez com que o componente aparecesse. Effects são executados no final de um [commit](/learn/render-and-commit) após a atualização da tela. Este é um bom momento para sincronizar os componentes React com algum sistema externo (como rede ou uma biblioteca de terceiros).

<Note>

Aqui e mais adiante neste texto, a palavra "Effect" com letra maiúscula se refere à definição específica do React acima, ou seja, um efeito colateral causado pela renderização. Para se referir ao conceito mais amplo de programação, diremos "efeito colateral".

</Note>

## Você pode não precisar de um Effect {/*you-might-not-need-an-effect*/}

**Não se apresse em adicionar Effects aos seus componentes.** Tenha em mente que os Effects são tipicamente usados para "sair" de seu código React e sincronizar com algum sistema *externo*. Isso inclui APIs do navegador, widgets de terceiros, rede e assim por diante. Se seu Effect apenas ajusta algum estado baseado em outro estado, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

## Como escrever um Effect {/*how-to-write-an-effect*/}

Para escrever um Effect, siga estas três etapas:

1.  **Declare um Effect.** Por padrão, seu Effect será executado após cada [commit](/learn/render-and-commit).
2.  **Especifique as dependências do Effect.** A maioria dos Effects deve ser executada novamente *somente quando necessário*, em vez de após cada renderização. Por exemplo, uma animação de fade-in deve ser acionada somente quando um componente aparece. Conectar e desconectar de uma sala de bate-papo deve acontecer somente quando o componente aparece e desaparece, ou quando a sala de bate-papo muda. Você aprenderá como controlar isso especificando *dependências*.
3.  **Adicione a limpeza, se necessário.** Alguns Effects precisam especificar como parar, desfazer ou limpar o que estavam fazendo. Por exemplo, "conectar" precisa de "desconectar", "inscrever-se" precisa de "cancelar a inscrição" e "buscar" precisa de "cancelar" ou "ignorar". Você aprenderá como fazer isso retornando uma *função de limpeza*.

Vamos analisar cada uma dessas etapas em detalhes.

### Passo 1: Declare um Effect {/*step-1-declare-an-effect*/}

Para declarar um Effect em seu componente, importe o [`useEffect` Hook](/reference/react/useEffect) do React:

```js
import { useEffect } from 'react';
```

Então, chame-o no nível superior do seu componente e coloque algum código dentro do seu Effect:

```js
function MyComponent() {
  useEffect(() => {
    // Code here will run after *every* render
  });
  return <div />;
}
```

Toda vez que seu componente renderizar, React atualizará a tela *e então* executará o código dentro de `useEffect`. Em outras palavras, **`useEffect` "atrasa" a execução de um pedaço de código até que a renderização seja refletida na tela.**

Vamos ver como você pode usar um Effect para sincronizar com um sistema externo. Considere um componente React `<VideoPlayer>`. Seria bom controlar se ele está tocando ou pausado passando uma prop `isPlaying` para ele:

```js
<VideoPlayer isPlaying={isPlaying} />;
```

Seu componente `VideoPlayer` personalizado renderiza a tag [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) do navegador:

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  return <video src={src} />;
}
```

No entanto, a tag `<video>` do navegador não tem uma prop `isPlaying`. A única maneira de controlá-la é chamar manualmente os métodos [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) e [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) no elemento DOM. **Você precisa sincronizar o valor da prop `isPlaying`, que indica se o vídeo _deve_ estar tocando no momento, com chamadas como `play()` e `pause()`.**

Precisaremos primeiro [obter uma ref](/learn/manipulating-the-dom-with-refs) para o nó DOM do `<video>`.

Você pode ser tentado a tentar chamar `play()` ou `pause()` durante a renderização, mas isso não está correto:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
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
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

A razão pela qual esse código não está correto é que ele tenta fazer algo com o nó DOM durante a renderização. No React, [a renderização deve ser um cálculo puro](/learn/keeping-components-pure) de JSX e não deve conter side effects como modificar o DOM.

Além disso, quando `VideoPlayer` é chamado pela primeira vez, seu DOM ainda não existe! Não há um nó DOM ainda para chamar `play()` ou `pause()`, porque o React não sabe qual DOM criar até que você retorne o JSX.

A solução aqui é **encapsular o efeito colateral com `useEffect` para movê-lo para fora do cálculo da renderização:**

```js
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

Ao encapsular a atualização do DOM em um Effect, você deixa o React atualizar a tela primeiro. Então seu Effect é executado.

Quando seu componente `VideoPlayer` renderizar (seja a primeira vez ou se ele re-renderizar), algumas coisas acontecerão. Primeiro, o React atualizará a tela, garantindo que a tag `<video>` esteja no DOM com as props corretas. Em seguida, o React executará seu Effect. Finalmente, seu Effect chamará `play()` ou `pause()` dependendo do valor de `isPlaying`.

Pressione Play/Pause várias vezes e veja como o reprodutor de vídeo permanece sincronizado com o valor `isPlaying`:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
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
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

Neste exemplo, o "sistema externo" que você sincronizou com o estado React foi a API de mídia do navegador. Você pode usar uma abordagem semelhante para encapsular código não React legado (como plugins jQuery) em componentes React declarativos.

Observe que controlar um reprodutor de vídeo é muito mais complexo na prática. Chamar `play()` pode falhar, o usuário pode reproduzir ou pausar usando os controles integrados do navegador e assim por diante. Este exemplo é muito simplificado e incompleto.

<Pitfall>

Por padrão, Effects são executados após *cada* renderização. É por isso que o código como este **produzirá um loop infinito:**

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

Effects são executados como um *resultado* da renderização. Definir o estado *aciona* a renderização. Definir o estado imediatamente em um Effect é como conectar uma tomada de energia a ela mesma. O Effect é executado, ele define o estado, o que causa uma nova renderização, o que faz com que o Effect seja executado, ele define o estado novamente, isso causa outra nova renderização e assim por diante.

Effects geralmente devem sincronizar seus componentes com um sistema *externo*. Se não houver nenhum sistema externo e você só quiser ajustar algum estado com base em outro estado, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

</Pitfall>

### Passo 2: Especifique as dependências do Effect {/*step-2-specify-the-effect-dependencies*/}

Por padrão, Effects são executados após *cada* renderização. Frequentemente, isso **não é o que você quer:**

- Às vezes, é lento. Sincronizar com um sistema externo nem sempre é instantâneo, então você pode querer pular essa etapa, a menos que seja necessário. Por exemplo, você não quer se reconectar ao servidor de chat a cada pressionamento de tecla.
- Às vezes, está errado. Por exemplo, você não quer acionar uma animação de fade-in do componente a cada pressionamento de tecla. A animação deve ser reproduzida apenas uma vez quando o componente aparecer pela primeira vez.

Para demonstrar o problema, aqui está o exemplo anterior com algumas chamadas `console.log` e uma entrada de texto que atualiza o estado do componente pai. Observe como a digitação faz com que o Effect seja executado novamente:

<Sandpack>

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
  });

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

Você pode dizer ao React para **pular a execução do Effect desnecessariamente** especificando um array de *dependências* como o segundo argumento para a chamada `useEffect`. Comece adicionando um array `[]` vazio ao exemplo acima na linha 14:

```js
  useEffect(() => {
    // ...
  }, []);
```

Você deve ver um erro dizendo `React Hook useEffect has a missing dependency: 'isPlaying'`:

<Sandpack>

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
  }, []); // This causes an error

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

O problema é que o código dentro do seu Effect *depende* da prop `isPlaying` para decidir o que fazer, mas essa dependência não foi declarada explicitamente. Para corrigir este problema, adicione `isPlaying` ao array de dependência:

```js
  useEffect(() => {
    if (isPlaying) { // It's used here...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...so it must be declared here!
```

Agora todas as dependências estão declaradas, então não há erro. Especificar `[isPlaying]` como o array de dependências informa ao React que ele deve pular a execução do seu Effect se `isPlaying` for o mesmo que era durante a renderização anterior. Com essa alteração, a digitação na entrada não faz com que o Effect seja executado novamente, mas pressionar Play/Pause faz:

<Sandpack>

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

O array de dependências pode conter várias dependências. O React só pulará a execução do Effect se *todas* as dependências que você especificar tiverem exatamente os mesmos valores que tinham durante a renderização anterior. O React compara os valores de dependência usando a comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Veja a [`referência useEffect`](/reference/react/useEffect#reference) para detalhes.

**Observe que você não pode "escolher" suas dependências.** Você receberá um erro de lint se as dependências que você especificou não corresponderem ao que o React espera com base no código dentro do seu Effect. Isso ajuda a detectar muitos bugs no seu código. Se você não quer que algum código seja executado novamente, [*edite o próprio código do Effect* para que ele não "precise" dessa dependência.](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

<Pitfall>

Os comportamentos sem o array de dependência e com um array de dependência *vazio* `[]` são diferentes:

```js
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

Vamos dar uma olhada mais de perto no que "montar" significa no próximo passo.

</Pitfall>

<DeepDive>

#### Por que a ref foi omitida do array de dependência? {/*why-was-the-ref-omitted-from-the-dependency-array*/}

Este Effect usa _tanto_ `ref` quanto `isPlaying`, mas apenas `isPlaying` é declarado como uma dependência:

```js
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

Isso ocorre porque o objeto `ref` tem uma *identidade estável:* React garante [que você sempre obterá o mesmo objeto](/reference/react/useRef#returns) da mesma chamada `useRef` em cada renderização. Ele nunca muda, então ele nunca, por si só, fará com que o Effect seja executado novamente. Portanto, não importa se você o inclui ou não. Incluí-lo também é bom:

```js
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

As funções [`set`](/reference/react/useState#setstate) retornadas por `useState` também têm identidade estável, então você frequentemente as verá omitidas das dependências também. Se o linter permitir que você omita uma dependência sem erros, é seguro fazê-lo.

Omitir dependências sempre estáveis só funciona quando o linter consegue "ver" que o objeto é estável. Por exemplo, se `ref` foi passado de um componente pai, você teria que especificá-lo no array de dependência. No entanto, isso é bom porque você não pode saber se o componente pai sempre passa a mesma ref ou passa uma de várias refs condicionalmente. Então seu Effect _dependeria_ de qual ref é passado.

</DeepDive>

### Passo 3: Adicione a limpeza, se necessário {/*step-3-add-cleanup-if-needed*/}

Considere um exemplo diferente. Você está escrevendo um componente `ChatRoom` que precisa se conectar ao servidor de chat quando ele aparece. Você recebe uma API `createConnection()` que retorna um objeto com os métodos `connect()` e `disconnect()`. Como você mantém o componente conectado enquanto ele é exibido para o usuário?

Comece escrevendo a lógica do Effect:

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Seria lento conectar-se ao chat após cada nova renderização, então você adiciona o array de dependência:

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**O código dentro do Effect não usa nenhuma prop ou estado, então seu array de dependência é `[]` (vazio). Isso informa ao React para executar este código somente quando o componente "monta", ou seja, aparece na tela pela primeira vez.**

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
  return <h1>Welcome to the chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting...');
    },
    disconnect() {
      console.log('❌ Disconnected.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

Este Effect só é executado na montagem, então você pode esperar que `"✅ Connecting..."` seja impresso uma vez no console. **No entanto, se você verificar o console, `"✅ Connecting..."` é impresso duas vezes. Por que isso acontece?**

Imagine que o componente `ChatRoom` faz parte de um aplicativo maior com muitas telas diferentes. O usuário inicia sua jornada na página `ChatRoom`. O componente monta e chama `connection.connect()`. Então imagine que o usuário navega para outra tela -- por exemplo, para a página de Configurações. O componente `ChatRoom` desmonta. Finalmente, o usuário clica em Voltar e `ChatRoom` monta novamente. Isso criaria uma segunda conexão -- mas a primeira conexão nunca foi destruída! Conforme o usuário navega pelo aplicativo, as conexões continuariam se acumulando.

Bugs como este são fáceis de perder sem extensos testes manuais. Para ajudá-lo a detectá-los rapidamente, no desenvolvimento o React remontará cada componente uma vez imediatamente após sua montagem inicial.

Ver o log `"✅ Connecting..."` duas vezes ajuda você a perceber o problema real: seu código não fecha a conexão quando o componente desmonta.

Para corrigir o problema, retorne uma *função de limpeza* do seu Effect:

```js
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

O React chamará sua função de limpeza sempre antes que o Effect seja executado novamente e uma vez final quando o componente desmontar (for removido). Vamos ver o que acontece quando a função de limpeza é implementada:

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
  return <h1>Welcome to the chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting...');
    },
    disconnect() {
      console.log('❌ Disconnected.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

Agora você obtém três logs no console em desenvolvimento:

1.  `"✅ Connecting..."`
2.  `"❌ Disconnected."`
3.  `"✅ Connecting..."`

**Este é o comportamento correto em desenvolvimento.** Ao remontar seu componente, o React verifica se navegar para fora e voltar não quebraria seu código. Desconectar e então conectar novamente é exatamente o que deve acontecer! Quando você implementa a limpeza bem, não deve haver nenhuma diferença visível pelo usuário entre executar o Effect uma vez vs. executá-lo, limpá-lo e executá-lo novamente. Há um par extra de chamadas de conexão/desconexão porque o React está testando seu código em busca de bugs no desenvolvimento. Isso é normal -- não tente fazer com que isso desapareça!

**Na produção, você só veria `"✅ Connecting..."` impresso uma vez.** Remontar componentes só acontece em desenvolvimento para ajudá-lo a encontrar Effects que precisam de limpeza. Você pode desativar o [Modo Estrito](/reference/react/StrictMode) para não participar do comportamento de desenvolvimento, mas recomendamos mantê-lo ativado. Isso permite que você encontre muitos bugs como o acima.

## Como lidar com o Effect disparando duas vezes no desenvolvimento? {/*how-to-handle-the-effect-firing-twice-in-development*/}

O React intencionalmente remonta seus componentes em desenvolvimento para encontrar bugs como no último exemplo. **A pergunta certa não é "como executar um Effect uma vez", mas "como corrigir meu Effect para que ele funcione após a remontagem".**

Normalmente, a resposta é implementar a função de limpeza. A função de limpeza deve parar ou desfazer o que quer que o Effect estivesse fazendo. A regra geral é que o usuário não deve ser capaz de distinguir entre o Effect executando uma vez (como na produção) e uma sequência  _setup → cleanup → setup_ (como você veria no desenvolvimento).

A maioria dos Effects que você escreverá se encaixará em um dos padrões comuns abaixo.

<Pitfall>

#### Não use refs para evitar que os Effects disparem {/*dont-use-refs-to-prevent-effects-from-firing*/}

Uma armadilha comum para evitar que os Effects disparem duas vezes em desenvolvimento é usar uma `ref` para evitar que o Effect seja executado mais de uma vez. Por exemplo, você pode "consertar" o bug acima com um `useRef`:

```js
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 This wont fix the bug!!!
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

Isso faz com que você só veja `"✅ Connecting..."` uma vez no desenvolvimento, mas não corrige o bug.

Quando o usuário navega para fora, a conexão ainda não é fechada e quando ele navega de volta, uma nova conexão é criada. À medida que o usuário navega pelo aplicativo, as conexões continuariam se acumulando, da mesma forma que seria antes da "correção".

Para corrigir o bug, não basta apenas fazer o Effect ser executado uma vez. O effect precisa funcionar após a remontagem, o que significa que a conexão precisa ser limpa como na solução acima.

Veja os exemplos abaixo para saber como lidar com padrões comuns.

</Pitfall>

### Controlando widgets não React {/*controlling-non-react-widgets*/}

Às vezes, você precisa adicionar widgets de UI que não são escritos em React. Por exemplo, digamos que você está adicionando um componente de mapa à sua página. Ele tem um método `setZoomLevel()`, e você gostaria de manter o nível de zoom sincronizado com uma variável de estado `zoomLevel` no seu código React. Seu Effect seria semelhante a isso:

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Observe que não é necessária nenhuma limpeza neste caso. Em desenvolvimento, o React chamará o Effect duas vezes, mas isso não é um problema porque chamar `setZoomLevel` duas vezes com o mesmo valor não faz nada. Pode ser um pouco mais lento, mas isso não importa porque não remontará desnecessariamente na produção.

Algumas APIs podem não permitir que você as chame duas vezes seguidas. Por exemplo, o método [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) do elemento [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) integrado lança um erro se você o chamar duas vezes. Implemente a função de limpeza e faça com que ela feche a caixa de diálogo:

```js
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

Em desenvolvimento, seu Effect chamará `showModal()`, então imediatamente `close()`, e então `showModal()` novamente. Isso tem o mesmo comportamento visível pelo usuário que chamar `showModal()` uma vez, como você veria na produção.

### Assinando eventos {/*subscribing-to-events*/}

Se seu Effect assina algo, a função de limpeza deve cancelar a assinatura:

```js
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

Em desenvolvimento, seu Effect chamará `addEventListener()`, então imediatamente `removeEventListener()`, e então `addEventListener()` novamente com o mesmo manipulador. Então haveria apenas uma assinatura ativa de cada vez. Isso tem o mesmo comportamento visível pelo usuário que chamar `addEventListener()` uma vez, como na produção.

### Acionando animações {/*triggering-animations*/}

Se seu Effect anima algo, a função de limpeza deve redefinir a animação para os valores iniciais:

```js
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

Em desenvolvimento, a opacidade será definida como `1`, depois para `0` e depois para `1` novamente. Isso deve ter o mesmo comportamento visível pelo usuário que defini-lo como `1` diretamente, que é o que aconteceria na produção. Se você usar uma biblioteca de animação de terceiros com suporte a tweening, sua função de limpeza deve redefinir a linha do tempo para seu estado inicial.

### Buscando dados {/*fetching-data*/}

Se seu Effect busca algo, a função de limpeza deve ou [abortar a busca](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) ou ignorar seu resultado:

```js
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

Você não pode "desfazer" uma solicitação de rede que já aconteceu, mas sua função de limpeza deve garantir que a busca que _não é mais relevante_ não continue afetando seu aplicativo. Se o `userId` mudar de `'Alice'` para `'Bob'`, a limpeza garante que a resposta de `'Alice'` seja ignorada mesmo que ela chegue depois de `'Bob'`.

**Em desenvolvimento, você verá duas buscas na guia Rede.** Não há nada de errado com isso. Com a abordagem acima, o primeiro Effect será imediatamente limpo, então sua cópia da variável `ignore` será definida como `true`. Então, mesmo que haja uma solicitação extra, ela não afetará o estado graças à verificação `if (!ignore)`.

**Na produção, haverá apenas um pedido.** Se a segunda solicitação em desenvolvimento estiver incomodando você, a melhor abordagem é usar uma solução que deduplica as solicitações e armazena em cache suas respostas entre os componentes:

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

Isso não apenas melhorará a experiência de desenvolvimento, mas também fará com que seu aplicativo pareça mais rápido. Por exemplo, o usuário pressionar o botão Voltar não terá que esperar que alguns dados sejam carregados novamente porque eles serão armazenados em cache. Você pode criar tal cache sozinho ou usar uma das muitas alternativas para buscar dados manualmente em Effects.

<DeepDive>

#### Quais são as boas alternativas para buscar dados em Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Escrever chamadas `fetch` dentro dos Effects é uma [maneira popular de buscar dados](https://www.robinwieruch.de/react-hooks-fetch-data/), especialmente em aplicativos totalmente do lado do cliente. Essa é, no entanto, uma abordagem muito manual e tem desvantagens significativas:
``````
- **Effects não são executados no servidor.** Isso significa que o HTML inicial renderizado no servidor somente incluirá um estado de carregamento sem dados. O computador cliente terá que baixar todo o JavaScript e renderizar seu aplicativo somente para descobrir que agora ele precisa carregar os dados. Isso não é muito eficiente.
- **Buscar dados diretamente em Effects facilita a criação de "cascatas de rede".** Você renderiza o componente pai, ele busca alguns dados, renderiza os componentes filhos e, em seguida, eles começam a buscar seus dados. Se a rede não for muito rápida, isso é significativamente mais lento do que buscar todos os dados em paralelo.
- **Buscar dados diretamente em Effects geralmente significa que você não pré-carrega ou armazena dados em cache.** Por exemplo, se o componente for desmontado e, em seguida, montado novamente, ele teria que buscar os dados novamente.
- **Não é muito ergonômico.** Há uma quantidade considerável de código boilerplate envolvido ao escrever chamadas `fetch` de uma forma que não sofra com erros como [condições de corrida.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

Essa lista de desvantagens não é específica do React. Ela se aplica à busca de dados na montagem com qualquer biblioteca. Assim como com o roteamento, a busca de dados não é trivial de fazer bem, por isso recomendamos as seguintes abordagens:

- **Se você usar um [framework](/learn/start-a-new-react-project#production-grade-react-frameworks), use seu mecanismo de busca de dados integrado.** Os modernos frameworks React têm mecanismos integrados de busca de dados que são eficientes e não sofrem com as armadilhas acima.
- **Caso contrário, considere usar ou construir um cache do lado do cliente.** As soluções de código aberto populares incluem [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/) e [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) Você também pode criar sua própria solução, caso em que usaria Effects por baixo dos panos, mas adicionaria lógica para deduplicação de solicitações, armazenamento em cache de respostas e evitar cascatas de rede (pré-carregando dados ou elevando os requisitos de dados para as rotas).

Você pode continuar buscando dados diretamente nos Effects se nenhuma dessas abordagens for adequada para você.

</DeepDive>

### Enviando análises {/*sending-analytics*/}

Considere este código que envia um evento de análise na visita à página:

```js
useEffect(() => {
  logVisit(url); // Envia uma requisição POST
}, [url]);
```

No desenvolvimento, `logVisit` será chamado duas vezes para cada URL, então você pode ser tentado a tentar corrigir isso. **Recomendamos manter este código como está.** Assim como nos exemplos anteriores, não há diferença de comportamento *visível ao usuário* entre executá-lo uma vez e executá-lo duas vezes. De um ponto de vista prático, `logVisit` não deve fazer nada no desenvolvimento, pois você não deseja que os logs das máquinas de desenvolvimento distorçam as métricas de produção. Seu componente é remontado toda vez que você salva seu arquivo, então ele registra visitas extras no desenvolvimento de qualquer maneira.

**Na produção, não haverá logs de visitas duplicados.**

Para depurar os eventos de análise que você está enviando, você pode implantar seu aplicativo em um ambiente de teste (que é executado no modo de produção) ou desativar temporariamente o [Modo Estrito](/reference/react/StrictMode) e suas verificações de remontagem apenas para desenvolvimento. Você também pode enviar análises dos manipuladores de eventos de alteração de rota em vez de Effects. Para análises mais precisas, [observadores de interseção](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) podem ajudar a rastrear quais componentes estão na janela de visualização e por quanto tempo eles permanecem visíveis.

### Não é um Effect: Inicializando o aplicativo {/*not-an-effect-initializing-the-application*/}

Alguma lógica deve ser executada apenas uma vez quando o aplicativo é iniciado. Você pode colocá-la fora de seus componentes:

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

Às vezes, mesmo que você escreva uma função de limpeza, não há como evitar as consequências visíveis ao usuário de executar o Effect duas vezes. Por exemplo, talvez seu Effect envie uma solicitação POST como a compra de um produto:

```js {2-3}
useEffect(() => {
  // 🔴 Errado: Este Effect dispara duas vezes no desenvolvimento, expondo um problema no código.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

Você não gostaria de comprar o produto duas vezes. No entanto, é também por isso que você não deve colocar essa lógica em um Effect. E se o usuário for para outra página e, em seguida, pressionar Voltar? Seu Effect seria executado novamente. Você não quer comprar o produto quando o usuário *visita* uma página; você quer comprá-lo quando o usuário *clica* no botão Comprar.

A compra não é causada pela renderização; ela é causada por uma interação específica. Ela deve ser executada apenas quando o usuário pressiona o botão. **Exclua o Effect e mova sua solicitação `/api/buy` para o manipulador de eventos do botão Comprar:**

```js {2-3}
  function handleClick() {
    // ✅ Comprar é um evento porque é causado por uma interação específica.
    fetch('/api/buy', { method: 'POST' });
  }
```

**Isso ilustra que, se a remontagem quebrar a lógica do seu aplicativo, isso geralmente revela erros existentes.** Do ponto de vista do usuário, visitar uma página não deve ser diferente de visitá-la, clicar em um link e, em seguida, pressionar Voltar para visualizar a página novamente. O React verifica se seus componentes cumprem esse princípio remontando-os uma vez no desenvolvimento.

## Juntando tudo {/*putting-it-all-together*/}

Este playground pode te ajudar a "ter uma ideia" de como os Effects funcionam na prática.

Este exemplo usa [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) para agendar um log no console com o texto de entrada para aparecer três segundos após a execução do Effect. A função de limpeza cancela o tempo limite pendente. Comece pressionando "Mount the component":

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

Você verá três logs no início: `Agendar log "a"`, `Cancelar log "a"` e `Agendar log "a"` novamente. Três segundos depois, também haverá um log dizendo `a`. Como você aprendeu antes, o par extra agendar/cancelar acontece porque o React remonta o componente uma vez no desenvolvimento para verificar se você implementou a limpeza bem.

Agora, edite a entrada para dizer `abc`. Se você fizer isso rápido o suficiente, verá `Agendar log "ab"` imediatamente seguido por `Cancelar log "ab"` e `Agendar log "abc"`. **O React sempre limpa o Effect da renderização anterior antes do Effect da próxima renderização.** É por isso que, mesmo que você digite na entrada rapidamente, há no máximo um tempo limite agendado por vez. Edite a entrada algumas vezes e observe o console para ter uma ideia de como os Effects são limpos.

Digite algo na entrada e, em seguida, pressione imediatamente "Desmontar o componente". Observe como a desmontagem limpa o Effect da última renderização. Aqui, ela limpa o último tempo limite antes que ele tenha a chance de disparar.

Finalmente, edite o componente acima e comente a função de limpeza para que os tempos limite não sejam cancelados. Tente digitar `abcde` rápido. O que você espera que aconteça em três segundos? `console.log(text)` dentro do tempo limite imprimirá o `text` *mais recente* e produzirá cinco logs `abcde`? Experimente para verificar sua intuição!

Três segundos depois, você deverá ver uma sequência de logs (`a`, `ab`, `abc`, `abcd` e `abcde`) em vez de cinco logs `abcde`. **Cada Effect "captura" o valor `text` de sua renderização correspondente.** Não importa que o estado `text` tenha mudado: um Effect da renderização com `text = 'ab'` sempre verá `'ab'`. Em outras palavras, os Effects de cada renderização são isolados uns dos outros. Se você está curioso sobre como isso funciona, pode ler sobre [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures).

<DeepDive>

#### Cada renderização tem seus próprios Effects {/*each-render-has-its-own-effects*/}

Você pode pensar no `useEffect` como "anexar" um pedaço de comportamento à saída da renderização. Considere este Effect:

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

Vamos ver o que exatamente acontece à medida que o usuário navega no aplicativo.

#### Renderização inicial {/*initial-render*/}

O usuário visita `<ChatRoom roomId="general" />`. Vamos [substituir mentalmente](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` por `'general'`:

```js
  // JSX para a primeira renderização (roomId = "general")
  return <h1>Bem-vindo ao general!</h1>;
```

**O Effect é *também* uma parte da saída da renderização.** O Effect da primeira renderização se torna:

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

O React executa esse Effect, que se conecta à sala de bate-papo `'general'`.

#### Re-render com as mesmas dependências {/*re-render-with-same-dependencies*/}

Digamos que `<ChatRoom roomId="general" />` seja renderizado novamente. A saída JSX é a mesma:

```js
  // JSX para a segunda renderização (roomId = "general")
  return <h1>Bem-vindo ao general!</h1>;
```

O React percebe que a saída da renderização não foi alterada, portanto, ele não atualiza o DOM.

O Effect da segunda renderização se parece com isso:

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

#### Re-render com dependências diferentes {/*re-render-with-different-dependencies*/}

Então, o usuário visita `<ChatRoom roomId="travel" />`. Desta vez, o componente retorna um JSX diferente:

```js
  // JSX para a terceira renderização (roomId = "travel")
  return <h1>Bem-vindo ao travel!</h1>;
```

O React atualiza o DOM para alterar `"Bem-vindo ao general"` para `"Bem-vindo ao travel"`.

O Effect da terceira renderização se parece com isso:

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

#### Desmontagem {/*unmount*/}

Finalmente, digamos que o usuário navegue para longe e o componente `ChatRoom` seja desmontado. O React executa a função de limpeza do último Effect. O último Effect foi da terceira renderização. A limpeza da terceira renderização destrói a conexão `createConnection('travel')`. Assim, o aplicativo se desconecta da sala `'travel'`.

#### Comportamentos apenas para desenvolvimento {/*development-only-behaviors*/}

Quando o [Modo Estrito](/reference/react/StrictMode) está ativado, o React remonta cada componente uma vez após a montagem (o estado e o DOM são preservados). Isso [ajuda você a encontrar Effects que precisam de limpeza](#step-3-add-cleanup-if-needed) e expõe erros como condições de corrida no início. Além disso, o React remontará os Effects sempre que você salvar um arquivo no desenvolvimento. Ambos esses comportamentos são apenas para desenvolvimento.

</DeepDive>

<Recap>

- Ao contrário dos eventos, os Effects são causados pela própria renderização, em vez de uma interação específica.
- Os Effects permitem que você sincronize um componente com algum sistema externo (API de terceiros, rede, etc).
- Por padrão, os Effects são executados após cada renderização (incluindo a inicial).
- O React ignorará o Effect se todas as suas dependências tiverem os mesmos valores que durante a última renderização.
- Você não pode "escolher" suas dependências. Elas são determinadas pelo código dentro do Effect.
- A matriz de dependência vazia (`[]`) corresponde à "montagem" do componente, ou seja, sendo adicionada à tela.
- No Modo Estrito, o React monta os componentes duas vezes (somente no desenvolvimento!) para testar seus Effects.
- Se seu Effect quebrar por causa da remontagem, você precisa implementar uma função de limpeza.
- O React chamará sua função de limpeza antes que o Effect seja executado na próxima vez e durante a desmontagem.

</Recap>

<Challenges>

#### Focar em um campo na montagem {/*focus-a-field-on-mount*/}

Neste exemplo, o formulário renderiza um componente `<MyInput />`.

Use o método [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) da entrada para fazer com que `MyInput` foque automaticamente quando aparecer na tela. Já existe uma implementação comentada, mas ela não funciona totalmente. Descubra por que ela não funciona e corrija-a. (Se você estiver familiarizado com o atributo `autoFocus`, finja que ele não existe: estamos reimplementando a mesma funcionalidade do zero.)

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
            Torná-lo maiúsculo
          </label>
          <p>Olá, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```
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

Para verificar se a sua solução funciona, pressione "Mostrar formulário" e verifique se a entrada recebe foco (fica realçada e o cursor é posicionado dentro dela). Pressione "Ocultar formulário" e "Mostrar formulário" novamente. Verifique se a entrada está realçada novamente.

`MyInput` deve focar apenas _na montagem_ em vez de após cada renderização. Para verificar se o comportamento está correto, pressione "Mostrar formulário" e, em seguida, pressione repetidamente a caixa de seleção "Colocar em caixa alta". Clicar na caixa de seleção _não_ deve focar a entrada acima dela.

<Solution>

Chamar `ref.current.focus()` durante a renderização é errado porque é um *efeito colateral*. Efeitos colaterais devem ser colocados dentro de um manipulador de eventos ou serem declarados com `useEffect`. Nesse caso, o efeito colateral é _causado_ pelo componente aparecer em vez de qualquer interação específica, então faz sentido colocá-lo em um Effect.

Para corrigir o erro, envolva a chamada `ref.current.focus()` em uma declaração Effect. Em seguida, para garantir que esse Effect seja executado somente na montagem e não após cada renderização, adicione as dependências `[]` vazias a ele.

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

Pressione "Mostrar formulário" e observe que o segundo campo recebe foco automaticamente. Isso ocorre porque ambos os componentes `<MyInput />` tentam focar o campo dentro. Quando você chama `focus()` para dois campos de entrada em sequência, o último sempre "vence".

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

Para verificar sua solução, pressione "Mostrar formulário" e "Ocultar formulário" repetidamente. Quando o formulário aparecer, somente a *primeira* entrada deverá receber foco. Isso ocorre porque o componente pai renderiza a primeira entrada com `shouldFocus={true}` e a segunda entrada com `shouldFocus={false}`. Verifique também se ambas as entradas ainda funcionam e você pode digitar em ambas.

<Hint>

Você não pode declarar um Effect condicionalmente, mas seu Effect pode incluir lógica condicional.

</Hint>

<Solution>

Coloque a lógica condicional dentro do Effect. Você precisará especificar `shouldFocus` como uma dependência porque você está usando-o dentro do Effect. (Isso significa que, se o `shouldFocus` de alguma entrada mudar de `false` para `true`, ela focará após a montagem.)

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

Este componente `Counter` exibe um contador que deve ser incrementado a cada segundo. Na montagem, ele chama [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) Isso faz com que `onTick` seja executado a cada segundo. A função `onTick` incrementa o contador.

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

Quando o [Modo Strict](/reference/react/StrictMode) está ativado (como nos sandboxes deste site), o React remontará cada componente uma vez no desenvolvimento. Isso faz com que o intervalo seja configurado duas vezes, e é por isso que a cada segundo o contador incrementa duas vezes.

No entanto, o comportamento do React não é a *causa* do erro: o erro já existe no código. O comportamento do React torna o erro mais perceptível. A causa real é que este Effect inicia um processo, mas não fornece uma maneira de limpá-lo.

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

No desenvolvimento, o React ainda remontará seu componente uma vez para verificar se você implementou a limpeza corretamente. Então, haverá uma chamada `setInterval`, seguida imediatamente por `clearInterval` e `setInterval` novamente. Na produção, haverá apenas uma chamada `setInterval`. O comportamento visível para o usuário em ambos os casos é o mesmo: o contador incrementa uma vez por segundo.

</Solution>

#### Corrigir a busca dentro de um Effect {/*fix-fetching-inside-an-effect*/}

Este componente mostra a biografia da pessoa selecionada. Ele carrega a biografia chamando uma função assíncrona `fetchBio(person)` na montagem e sempre que `person` muda. Essa função assíncrona retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que eventualmente é resolvida para uma string. Quando a busca é concluída, ele chama `setBio` para exibir essa string abaixo da caixa de seleção.

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
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

Há um erro neste código. Comece selecionando "Alice". Em seguida, selecione "Bob" e, imediatamente após, selecione "Taylor". Se você fizer isso rápido o suficiente, notará que o erro: Taylor é selecionado, mas o parágrafo abaixo diz "This is Bob's bio".

Por que isso acontece? Corrija o erro dentro deste Effect.

<Hint>

Se um Effect buscar algo de forma assíncrona, geralmente precisa de limpeza.

</Hint>

<Solution>

Para disparar o erro, as coisas precisam acontecer nesta ordem:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')`
- **Buscar `'Taylor'` é concluído *antes* de buscar `'Bob'`**
- O Effect da renderização `'Taylor'` chama `setBio('This is Taylor’s bio')`
- A busca de `'Bob'` é concluída
- O Effect da renderização `'Bob'` chama `setBio('This is Bob’s bio')`

É por isso que você vê a biografia de Bob, embora Taylor esteja selecionado. Erros como esse são chamados de [condições de corrida](https://pt.wikipedia.org/wiki/Condição_de_corrida) porque duas operações assíncronas estão "competindo" entre si e podem chegar em uma ordem inesperada.

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
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

O Effect de cada renderização tem sua própria variável `ignore`. Inicialmente, a variável `ignore` é definida como `false`. No entanto, se um Effect for limpo (como quando você seleciona uma pessoa diferente), sua variável `ignore` se torna `true`. Então, agora não importa em que ordem as solicitações sejam concluídas. Somente o Effect da última pessoa terá `ignore` definido como `false`, então ele chamará `setBio(result)`. Os Effects anteriores foram limpos, então a verificação `if (!ignore)` impedirá que eles chamem `setBio`:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')` **e limpa o Effect anterior (de Bob)**
- Buscar `'Taylor'` é concluído *antes* de buscar `'Bob'`
- O Effect da renderização `'Taylor'` chama `setBio('This is Taylor’s bio')`
- A busca de `'Bob'` é concluída
- O Effect da renderização `'Bob'` **não faz nada porque sua flag `ignore` foi definida como `true`**

Além de ignorar o resultado de uma chamada de API desatualizada, você também pode usar [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) para cancelar as solicitações que não são mais necessárias. No entanto, por si só, isso não é suficiente para proteger contra condições de corrida. Mais etapas assíncronas podem ser encadeadas após a busca, então usar uma flag explícita como `ignore` é a maneira mais confiável de corrigir esse tipo de problema.

</Solution>

</Challenges>
```