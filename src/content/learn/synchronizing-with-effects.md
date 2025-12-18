---
title: 'Sincronizando com Efeitos'
---

<Intro>

Alguns componentes precisam se sincronizar com sistemas externos. Por exemplo, você pode querer controlar um componente não-React com base no estado do React, configurar uma conexão de servidor ou enviar um log de análise quando um componente aparece na tela. *Efeitos* permitem que você execute algum código após a renderização para que possa sincronizar seu componente com algum sistema fora do React.

</Intro>

<YouWillLearn>

- O que são Efeitos
- Como os Efeitos são diferentes dos eventos
- Como declarar um Efeito em seu componente
- Como evitar reexecutar um Efeito desnecessariamente
- Por que os Efeitos rodam duas vezes em desenvolvimento e como corrigi-los

</YouWillLearn>

## O que são Efeitos e como eles são diferentes dos eventos? {/*what-are-effects-and-how-are-they-different-from-events*/}

Antes de chegar aos Efeitos, você precisa se familiarizar com dois tipos de lógica dentro dos componentes React:

- **Código de renderização** (introduzido em [Descrevendo a UI](/learn/describing-the-ui)) vive no nível superior do seu componente. É aqui que você pega os props e o estado, os transforma e retorna o JSX que deseja ver na tela. [O código de renderização deve ser puro.](/learn/keeping-components-pure) Como uma fórmula matemática, ele deve apenas _calcular_ o resultado, mas não fazer mais nada.

- **Manipuladores de eventos** (introduzidos em [Adicionando Interatividade](/learn/adding-interactivity)) são funções aninhadas dentro de seus componentes que _fazem_ coisas em vez de apenas calculá-las. Um manipulador de eventos pode atualizar um campo de entrada, enviar uma solicitação HTTP POST para comprar um produto ou navegar o usuário para outra tela. Manipuladores de eventos contêm ["efeitos colaterais"](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) (eles mudam o estado do programa) causados por uma ação específica do usuário (por exemplo, um clique de botão ou digitação).

Às vezes, isso não é suficiente. Considere um componente `ChatRoom` que deve se conectar ao servidor de chat sempre que estiver visível na tela. Conectar-se a um servidor não é um cálculo puro (é um efeito colateral), portanto, não pode ocorrer durante a renderização. No entanto, não há um único evento específico, como um clique, que cause a exibição do `ChatRoom`.

***Efeitos* permitem que você especifique efeitos colaterais que são causados pela própria renderização, em vez de por um evento específico.** Enviar uma mensagem no chat é um *evento* porque é diretamente causado pelo usuário clicando em um botão específico. No entanto, configurar uma conexão de servidor é um *Efeito* porque deve acontecer independentemente de qual interação causou o aparecimento do componente. Efeitos rodam no final de um [commit](/learn/render-and-commit) após a tela ser atualizada. Este é um bom momento para sincronizar os componentes React com algum sistema externo (como rede ou uma biblioteca de terceiros).

<Note>

Aqui e mais adiante neste texto, "Efeito" capitalizado refere-se à definição específica do React acima, ou seja, um efeito colateral causado pela renderização. Para se referir ao conceito de programação mais amplo, diremos "efeito colateral".

</Note>


## Você pode não precisar de um Efeito {/*you-might-not-need-an-effect*/}

**Não se apresse em adicionar Efeitos aos seus componentes.** Tenha em mente que os Efeitos são tipicamente usados para "sair" do seu código React e sincronizar com algum sistema *externo*. Isso inclui APIs do navegador, widgets de terceiros, rede e assim por diante. Se o seu Efeito apenas ajusta algum estado com base em outro estado, [você pode não precisar de um Efeito.](/learn/you-might-not-need-an-effect)

## Como escrever um Effect {/*how-to-write-an-effect*/}

Para escrever um Effect, siga estes três passos:

1.  **Declare um Effect.** Por padrão, seu Effect será executado após cada [commit](/learn/render-and-commit).
2.  **Especifique as dependências do Effect.** A maioria dos Effects só deve ser reexecutada *quando necessário*, em vez de após cada renderização. Por exemplo, uma animação de fade-in só deve ser acionada quando um componente aparece. Conectar e desconectar de uma sala de chat só deve acontecer quando o componente aparece e desaparece, ou quando a sala de chat muda. Você aprenderá como controlar isso especificando *dependências*.
3.  **Adicione limpeza, se necessário.** Alguns Effects precisam especificar como parar, desfazer ou limpar o que estavam fazendo. Por exemplo, "conectar" precisa de "desconectar", "inscrever" precisa de "cancelar inscrição", e "buscar" precisa de "cancelar" ou "ignorar". Você aprenderá como fazer isso retornando uma *função de limpeza*.

Vamos analisar cada um desses passos em detalhes.

### Passo 1: Declare um Effect {/*step-1-declare-an-effect*/}

Para declarar um Effect em seu componente, importe o [`useEffect` Hook](/reference/react/useEffect) do React:

```js
import { useEffect } from 'react';
```

Em seguida, chame-o no nível superior do seu componente e coloque algum código dentro do seu Effect:

```js {2-4}
function MeuComponente() {
  useEffect(() => {
    // O código aqui será executado após *cada* renderização
  });
  return <div />;
}
```

Toda vez que seu componente renderizar, o React atualizará a tela *e então* executará o código dentro de `useEffect`. Em outras palavras, **`useEffect` "atrasa" a execução de um trecho de código até que essa renderização seja refletida na tela.**

Vamos ver como você pode usar um Effect para sincronizar com um sistema externo. Considere um componente React `<VideoPlayer>`. Seria bom controlar se ele está tocando ou pausado passando uma prop `isPlaying` para ele:

```js
<VideoPlayer isPlaying={isPlaying} />;
```

Seu componente `VideoPlayer` personalizado renderiza a tag [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) do navegador:

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: faça algo com isPlaying
  return <video src={src} />;
}
```

No entanto, a tag `<video>` do navegador não possui uma prop `isPlaying`. A única maneira de controlá-la é chamar manualmente os métodos [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) e [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) no elemento DOM. **Você precisa sincronizar o valor da prop `isPlaying`, que indica se o vídeo _deve_ estar tocando no momento, com chamadas como `play()` e `pause()`.**

Precisaremos primeiro [obter uma ref](/learn/manipulating-the-dom-with-refs) para o nó DOM do `<video>`.

Você pode ser tentado a tentar chamar `play()` ou `pause()` durante a renderização, mas isso não está correto:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Chamar isso durante a renderização não é permitido.
  } else {
    ref.current.pause(); // Além disso, isso causa um erro.
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

A razão pela qual este código não está correto é que ele tenta fazer algo com o nó DOM durante a renderização. No React, [a renderização deve ser um cálculo puro](/learn/keeping-components-pure) de JSX e não deve conter efeitos colaterais como modificar o DOM.

Além disso, quando `VideoPlayer` é chamado pela primeira vez, seu DOM ainda não existe! Ainda não há um nó DOM para chamar `play()` ou `pause()` nele, porque o React não sabe qual DOM criar até que você retorne o JSX.

A solução aqui é **envolver o efeito colateral com `useEffect` para removê-lo do cálculo de renderização:**

```js {6,12}
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

Ao envolver a atualização do DOM em um Effect, você permite que o React atualize a tela primeiro. Em seguida, seu Effect é executado.

Quando seu componente `VideoPlayer` renderizar (seja na primeira vez ou se re-renderizar), algumas coisas acontecerão. Primeiro, o React atualizará a tela, garantindo que a tag `<video>` esteja no DOM com as props corretas. Em seguida, o React executará seu Effect. Finalmente, seu Effect chamará `play()` ou `pause()` dependendo do valor de `isPlaying`.

Pressione Play/Pause várias vezes e veja como o player de vídeo permanece sincronizado com o valor de `isPlaying`:

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

Neste exemplo, o "sistema externo" ao qual você sincronizou o estado do React foi a API de mídia do navegador. Você pode usar uma abordagem semelhante para envolver código legado não-React (como plugins jQuery) em componentes React declarativos.

Note que controlar um player de vídeo é muito mais complexo na prática. Chamar `play()` pode falhar, o usuário pode reproduzir ou pausar usando os controles nativos do navegador, e assim por diante. Este exemplo é muito simplificado e incompleto.

<Pitfall>

Por padrão, os Effects são executados após *cada* renderização. É por isso que um código como este **produzirá um loop infinito:**

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

Os Effects são executados como um *resultado* da renderização. Definir o estado *desencadeia* a renderização. Definir o estado imediatamente em um Effect é como conectar uma tomada a si mesma. O Effect é executado, ele define o estado, o que causa uma re-renderização, que causa a execução do Effect, ele define o estado novamente, isso causa outra re-renderização, e assim por diante.

Os Effects geralmente devem sincronizar seus componentes com um sistema *externo*. Se não houver um sistema externo e você quiser apenas ajustar algum estado com base em outro estado, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

</Pitfall>

### Etapa 2: Especifique as dependências do Effect {/*step-2-specify-the-effect-dependencies*/}

Por padrão, os Effects são executados após *cada* renderização. Frequentemente, isso **não é o que você deseja:**

- Às vezes, é lento. Sincronizar com um sistema externo nem sempre é instantâneo, então você pode querer evitar fazê-lo a menos que seja necessário. Por exemplo, você não quer reconectar ao servidor de chat a cada pressionamento de tecla.
- Às vezes, está errado. Por exemplo, você não quer acionar uma animação de fade-in do componente a cada pressionamento de tecla. A animação só deve ser reproduzida uma vez quando o componente aparece pela primeira vez.

Para demonstrar o problema, aqui está o exemplo anterior com algumas chamadas `console.log` e um campo de texto que atualiza o estado do componente pai. Observe como a digitação faz com que o Effect seja reexecutado:

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

Você pode dizer ao React para **pular a reexecução desnecessária do Effect** especificando um array de *dependências* como o segundo argumento para a chamada `useEffect`. Comece adicionando um array vazio `[]` ao exemplo acima na linha 14:

```js {3}
  useEffect(() => {
    // ...
  }, []);
```

Você verá um erro dizendo `React Hook useEffect has a missing dependency: 'isPlaying'` (Hook useEffect do React tem uma dependência ausente: 'isPlaying'):

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
  }, []); // Isso causa um erro

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

O problema é que o código dentro do seu Effect *depende* da prop `isPlaying` para decidir o que fazer, mas essa dependência não foi explicitamente declarada. Para corrigir esse problema, adicione `isPlaying` ao array de dependências:

```js {2,7}
  useEffect(() => {
    if (isPlaying) { // É usado aqui...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...então deve ser declarado aqui!
```

Agora todas as dependências estão declaradas, então não há erro. Especificar `[isPlaying]` como o array de dependências diz ao React que ele deve pular a reexecução do seu Effect se `isPlaying` for o mesmo que era durante a renderização anterior. Com essa alteração, digitar no input não faz o Effect ser reexecutado, mas pressionar Play/Pause sim:

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

O array de dependências pode conter múltiplas dependências. O React só pulará a reexecução do Effect se *todas* as dependências que você especificar tiverem exatamente os mesmos valores que tinham durante a renderização anterior. O React compara os valores das dependências usando a comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Veja a [referência do `useEffect`](/reference/react/useEffect#reference) para detalhes.

**Observe que você não pode "escolher" suas dependências.** Você receberá um erro de lint se as dependências que você especificou não corresponderem ao que o React espera com base no código dentro do seu Effect. Isso ajuda a capturar muitos bugs no seu código. Se você não quer que algum código seja reexecutado, [*edite o próprio código do Effect* para não "precisar" dessa dependência.](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

<Pitfall>

Os comportamentos sem o array de dependências e com um array de dependências *vazio* `[]` são diferentes:

```js {3,7,11}
useEffect(() => {
  // Isso executa após cada renderização
});

useEffect(() => {
  // Isso executa apenas na montagem (quando o componente aparece)
}, []);

useEffect(() => {
  // Isso executa na montagem *e também* se a ou b mudaram desde a última renderização
}, [a, b]);
```

Analisaremos de perto o que "montagem" significa na próxima etapa.

</Pitfall>

<DeepDive>

#### Por que o ref foi omitido do array de dependências? {/*why-was-the-ref-omitted-from-the-dependency-array*/}

Este Effect usa *tanto* `ref` quanto `isPlaying`, mas apenas `isPlaying` é declarado como dependência:

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

Isso ocorre porque o objeto `ref` tem uma *identidade estável*: o React garante [que você sempre receberá o mesmo objeto](/reference/react/useRef#returns) da mesma chamada `useRef` em cada renderização. Ele nunca muda, então nunca causará por si só a reexecução do Effect. Portanto, não importa se você o inclui ou não. Incluí-lo também é aceitável:

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

As funções [`set` de atualização](/reference/react/useState#setstate) retornadas por `useState` também têm identidade estável, então você frequentemente as verá omitidas das dependências também. Se o linter permitir que você omita uma dependência sem erros, é seguro fazê-lo.

Omitir dependências sempre estáveis só funciona quando o linter pode "ver" que o objeto é estável. Por exemplo, se `ref` fosse passado de um componente pai, você teria que especificá-lo no array de dependências. No entanto, isso é bom porque você não pode saber se o componente pai sempre passa o mesmo ref, ou passa um de vários refs condicionalmente. Então seu Effect *dependeria* de qual ref é passado.

</DeepDive>

### Etapa 3: Adicione limpeza se necessário {/*step-3-add-cleanup-if-needed*/}

Considere um exemplo diferente. Você está escrevendo um componente `ChatRoom` que precisa se conectar ao servidor de chat quando ele aparece. Você recebe uma API `createConnection()` que retorna um objeto com métodos `connect()` e `disconnect()`. Como você mantém o componente conectado enquanto ele é exibido ao usuário?

Comece escrevendo a lógica do Effect:

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Seria lento conectar ao chat após cada re-renderização, então você adiciona o array de dependências:

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**O código dentro do Effect não usa nenhuma prop ou estado, então seu array de dependências é `[]` (vazio). Isso diz ao React para executar esse código apenas quando o componente "montar", ou seja, aparecer na tela pela primeira vez.**

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
  // Uma implementação real realmente se conectaria ao servidor
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

Imagine que o componente `ChatRoom` faz parte de um aplicativo maior com muitas telas diferentes. O usuário inicia sua jornada na página `ChatRoom`. O componente monta e chama `connection.connect()`. Em seguida, imagine que o usuário navega para outra tela - por exemplo, para a página de Configurações. O componente `ChatRoom` desmonta. Finalmente, o usuário clica em Voltar e `ChatRoom` monta novamente. Isso configuraria uma segunda conexão - mas a primeira conexão nunca foi destruída! À medida que o usuário navega pelo aplicativo, as conexões continuariam se acumulando.

Bugs como esse são fáceis de perder sem testes manuais extensivos. Para ajudá-lo a identificá-los rapidamente, em desenvolvimento, o React remonta cada componente uma vez imediatamente após sua montagem inicial.

Ver o log `"✅ Connecting..."` duas vezes ajuda você a perceber o problema real: seu código não fecha a conexão quando o componente desmonta.

Para corrigir o problema, retorne uma *função de limpeza* do seu Effect:

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

O React chamará sua função de limpeza cada vez antes que o Effect seja executado novamente, e uma última vez quando o componente desmontar (for removido). Vamos ver o que acontece quando a função de limpeza é implementada:

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
  // Uma implementação real realmente se conectaria ao servidor
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

1. `"✅ Connecting..."`
2. `"❌ Disconnected."`
3. `"✅ Connecting..."`

**Este é o comportamento correto em desenvolvimento.** Ao remontar seu componente, o React verifica se a navegação para longe e de volta não quebraria seu código. Desconectar e depois conectar novamente é exatamente o que deveria acontecer! Quando você implementa a limpeza corretamente, não deve haver diferença visível para o usuário entre executar o Effect uma vez e executá-lo, limpá-lo e executá-lo novamente. Há um par extra de chamadas connect/disconnect porque o React está sondando seu código em busca de bugs em desenvolvimento. Isso é normal - não tente fazer com que desapareça!

**Em produção, você veria apenas `"✅ Connecting..."` impresso uma vez.** A remontagem de componentes só acontece em desenvolvimento para ajudá-lo a encontrar Effects que precisam de limpeza. Você pode desativar o [Strict Mode](/reference/react/StrictMode) para optar por não participar do comportamento de desenvolvimento, mas recomendamos mantê-lo ativado. Isso permite que você encontre muitos bugs como o acima.

## Como lidar com o Effect disparando duas vezes em desenvolvimento? {/*how-to-handle-the-effect-firing-twice-in-development*/}

O React remonta intencionalmente seus componentes em desenvolvimento para encontrar bugs como no último exemplo. **A pergunta certa não é "como executar um Effect uma vez", mas sim "como corrigir meu Effect para que ele funcione após a remontagem".**

Geralmente, a resposta é implementar a função de limpeza. A função de limpeza deve parar ou desfazer o que quer que o Effect estivesse fazendo. A regra geral é que o usuário não deve ser capaz de distinguir entre o Effect sendo executado uma vez (como na produção) e uma sequência de _configuração → limpeza → configuração_ (como você veria em desenvolvimento).

A maioria dos Effects que você escreverá se encaixará em um dos padrões comuns abaixo.

<Pitfall>

#### Não use refs para impedir que Effects disparem {/*dont-use-refs-to-prevent-effects-from-firing*/}

Uma armadilha comum para impedir que Effects disparem duas vezes em desenvolvimento é usar uma `ref` para evitar que o Effect seja executado mais de uma vez. Por exemplo, você poderia "corrigir" o bug acima com um `useRef`:

```js {1,3-4}
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 Isso não corrigirá o bug!!!
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

Isso faz com que você veja apenas `"✅ Connecting..."` uma vez em desenvolvimento, mas não corrige o bug.

Quando o usuário navega para longe, a conexão ainda não é fechada e, quando ele volta, uma nova conexão é criada. À medida que o usuário navega pelo aplicativo, as conexões continuariam se acumulando, da mesma forma que antes da "correção".

Para corrigir o bug, não basta fazer o Effect ser executado uma vez. O Effect precisa funcionar após a remontagem, o que significa que a conexão precisa ser limpa, como na solução acima.

Veja os exemplos abaixo para saber como lidar com padrões comuns.

</Pitfall>

### Controlando widgets não-React {/*controlling-non-react-widgets*/}

Às vezes, você precisa adicionar widgets de UI que não foram escritos em React. Por exemplo, digamos que você esteja adicionando um componente de mapa à sua página. Ele tem um método `setZoomLevel()` e você gostaria de manter o nível de zoom sincronizado com uma variável de estado `zoomLevel` em seu código React. Seu Effect se pareceria com isto:

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Note que não há necessidade de limpeza neste caso. Em desenvolvimento, o React chamará o Effect duas vezes, mas isso não é um problema porque chamar `setZoomLevel` duas vezes com o mesmo valor não faz nada. Pode ser um pouco mais lento, mas isso não importa porque não haverá remontagem desnecessária em produção.

Algumas APIs podem não permitir que você as chame duas vezes seguidas. Por exemplo, o método [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) do elemento [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) integrado lança um erro se você o chamar duas vezes. Implemente a função de limpeza e faça-a fechar a caixa de diálogo:

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

Em desenvolvimento, seu Effect chamará `showModal()`, depois imediatamente `close()`, e então `showModal()` novamente. Isso tem o mesmo comportamento visível para o usuário que chamar `showModal()` uma vez, como ocorreria em produção.

### Assinando eventos {/*subscribing-to-events*/}

Se o seu Effect se inscrever em algo, a função de limpeza deve cancelar a inscrição:

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

Em desenvolvimento, seu Effect chamará `addEventListener()`, depois imediatamente `removeEventListener()`, e então `addEventListener()` novamente com o mesmo manipulador. Assim, haverá apenas uma assinatura ativa por vez. Isso tem o mesmo comportamento visível para o usuário que chamar `addEventListener()` uma vez, como em produção.

### Acionando animações {/*triggering-animations*/}

Se o seu Effect animar algo, a função de limpeza deve redefinir a animação para os valores iniciais:

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Aciona a animação
  return () => {
    node.style.opacity = 0; // Redefine para o valor inicial
  };
}, []);
```

Em desenvolvimento, a opacidade será definida para `1`, depois para `0`, e então para `1` novamente. Isso deve ter o mesmo comportamento visível para o usuário que defini-la para `1` diretamente, que é o que aconteceria em produção. Se você usar uma biblioteca de animação de terceiros com suporte para tweening, sua função de limpeza deve redefinir a linha do tempo para seu estado inicial.

### Buscando dados {/*fetching-data*/}

Se o seu Effect buscar algo, a função de limpeza deve [abortar o fetch](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) ou ignorar seu resultado:

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

Você não pode "desfazer" uma solicitação de rede que já ocorreu, mas sua função de limpeza deve garantir que o fetch que _não é mais relevante_ não continue afetando sua aplicação. Se o `userId` mudar de `'Alice'` para `'Bob'`, a limpeza garante que a resposta de `'Alice'` seja ignorada, mesmo que chegue depois de `'Bob'`.

**Em desenvolvimento, você verá dois fetches na aba Network.** Não há nada de errado com isso. Com a abordagem acima, o primeiro Effect será imediatamente limpo, de modo que sua cópia da variável `ignore` será definida como `true`. Portanto, embora haja uma solicitação extra, ela não afetará o estado graças à verificação `if (!ignore)`.

**Em produção, haverá apenas uma solicitação.** Se a segunda solicitação em desenvolvimento estiver incomodando você, a melhor abordagem é usar uma solução que deduplique solicitações e armazene em cache suas respostas entre os componentes:

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

Isso não apenas melhorará a experiência de desenvolvimento, mas também tornará sua aplicação mais rápida. Por exemplo, o usuário que pressiona o botão Voltar não terá que esperar que alguns dados sejam carregados novamente porque eles estarão em cache. Você pode criar esse cache você mesmo ou usar uma das muitas alternativas para buscar dados manualmente em Effects.

<DeepDive>

#### Quais são as boas alternativas para buscar dados em Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Escrever chamadas `fetch` dentro de Effects é uma [maneira popular de buscar dados](https://www.robinwieruch.de/react-hooks-fetch-data/), especialmente em aplicativos totalmente do lado do cliente. Esta, no entanto, é uma abordagem muito manual e tem desvantagens significativas:

- **Effects não são executados no servidor.** Isso significa que o HTML renderizado inicialmente pelo servidor incluirá apenas um estado de carregamento sem dados. O computador do cliente terá que baixar todo o JavaScript e renderizar seu aplicativo apenas para descobrir que agora ele precisa carregar os dados. Isso não é muito eficiente.
- **Buscar dados diretamente em Effects facilita a criação de "cadeias de rede".** Você renderiza o componente pai, ele busca alguns dados, renderiza os componentes filhos, e então eles começam a buscar seus dados. Se a rede não for muito rápida, isso é significativamente mais lento do que buscar todos os dados em paralelo.
- **Buscar dados diretamente em Effects geralmente significa que você não pré-carrega ou armazena dados em cache.** Por exemplo, se o componente desmontar e depois montar novamente, ele terá que buscar os dados novamente.
- **Não é muito ergonômico.** Há uma quantidade considerável de código repetitivo envolvido ao escrever chamadas `fetch` de uma maneira que não sofra de bugs como [condições de corrida.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

Esta lista de desvantagens não é específica do React. Ela se aplica à busca de dados ao montar com qualquer biblioteca. Assim como no roteamento, buscar dados não é trivial de fazer bem, então recomendamos as seguintes abordagens:

- **Se você usa um [framework](/learn/start-a-new-react-project#full-stack-frameworks), use seu mecanismo de busca de dados integrado.** Frameworks modernos de React têm mecanismos de busca de dados integrados que são eficientes e não sofrem das armadilhas acima.
- **Caso contrário, considere usar ou construir um cache do lado do cliente.** Soluções populares de código aberto incluem [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/) e [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) Você também pode construir sua própria solução, caso em que usaria Effects internamente, mas adicionaria lógica para deduplicar solicitações, armazenar em cache respostas e evitar cadeias de rede (pré-carregando dados ou elevando os requisitos de dados para rotas).

Você pode continuar buscando dados diretamente em Effects se nenhuma dessas abordagens for adequada para você.

</DeepDive>

### Enviando análises {/*sending-analytics*/}

Considere este código que envia um evento de análise na visita à página:

```js
useEffect(() => {
  logVisit(url); // Envia uma solicitação POST
}, [url]);
```

Em desenvolvimento, `logVisit` será chamado duas vezes para cada URL, então você pode ser tentado a tentar corrigir isso. **Recomendamos manter este código como está.** Assim como nos exemplos anteriores, não há diferença de comportamento *visível para o usuário* entre executá-lo uma vez e executá-lo duas vezes. De um ponto de vista prático, `logVisit` não deve fazer nada em desenvolvimento, pois você não quer que os logs das máquinas de desenvolvimento distorçam as métricas de produção. Seu componente é remontado toda vez que você salva seu arquivo, então ele registra visitas extras em desenvolvimento de qualquer maneira.

**Em produção, não haverá logs de visita duplicados.**

Para depurar os eventos de análise que você está enviando, você pode implantar seu aplicativo em um ambiente de staging (que é executado em modo de produção) ou optar temporariamente por não usar o [Strict Mode](/reference/react/StrictMode) e suas verificações de remontagem apenas em desenvolvimento. Você também pode enviar análises dos manipuladores de eventos de mudança de rota em vez de Effects. Para análises mais precisas, [observadores de interseção](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) podem ajudar a rastrear quais componentes estão na viewport e por quanto tempo permanecem visíveis.

### Não é um Effect: Inicializando a aplicação {/*not-an-effect-initializing-the-application*/}

Alguma lógica deve ser executada apenas uma vez quando a aplicação inicia. Você pode colocá-la fora de seus componentes:

```js {2-3}
if (typeof window !== 'undefined') { // Verifica se estamos executando no navegador.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

Isso garante que tal lógica seja executada apenas uma vez após o navegador carregar a página.

### Não é um Effect: Comprando um produto {/*not-an-effect-buying-a-product*/}

Às vezes, mesmo que você escreva uma função de limpeza, não há como evitar as consequências visíveis para o usuário de executar o Effect duas vezes. Por exemplo, talvez seu Effect envie uma solicitação POST como a compra de um produto:

```js {2-3}
useEffect(() => {
  // 🔴 Errado: Este Effect dispara duas vezes em desenvolvimento, expondo um problema no código.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

Você não gostaria de comprar o produto duas vezes. No entanto, é por isso que você também não deve colocar essa lógica em um Effect. E se o usuário for para outra página e depois pressionar Voltar? Seu Effect seria executado novamente. Você não quer comprar o produto quando o usuário *visita* uma página; você quer comprá-lo quando o usuário *clica* no botão Comprar.

A compra não é causada pela renderização; é causada por uma interação específica. Ela deve ser executada apenas quando o usuário pressiona o botão. **Exclua o Effect e mova sua solicitação `/api/buy` para o manipulador de eventos do botão Comprar:**

```js {2-3}
  function handleClick() {
    // ✅ Comprar é um evento porque é causado por uma interação específica.
    fetch('/api/buy', { method: 'POST' });
  }
```

**Isso ilustra que, se a remontagem quebrar a lógica do seu aplicativo, isso geralmente descobre bugs existentes.** Do ponto de vista do usuário, visitar uma página não deve ser diferente de visitá-la, clicar em um link e depois pressionar Voltar para ver a página novamente. O React verifica se seus componentes cumprem esse princípio, remontando-os uma vez em desenvolvimento.

## Juntando tudo {/*putting-it-all-together*/}

Este playground pode ajudar você a "sentir" como os Effects funcionam na prática.

Este exemplo usa [`setTimeout`](https://developer.mozilla.org/pt-BR/docs/Web/API/setTimeout) para agendar um log no console com o texto de entrada para aparecer três segundos após o Effect ser executado. A função de limpeza cancela o timeout pendente. Comece pressionando "Mount the component":

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{' '}
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
        {show ? 'Unmount' : 'Mount'} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

</Sandpack>

Você verá três logs inicialmente: `Schedule "a" log`, `Cancel "a" log`, e `Schedule "a" log` novamente. Três segundos depois, haverá também um log dizendo `a`. Como você aprendeu anteriormente, o par extra de agendamento/cancelamento ocorre porque o React remonta o componente uma vez em desenvolvimento para verificar se você implementou a limpeza corretamente.

Agora edite a entrada para dizer `abc`. Se você fizer isso rápido o suficiente, verá `Schedule "ab" log` imediatamente seguido por `Cancel "ab" log` e `Schedule "abc" log`. **O React sempre limpa o Effect do render anterior antes do Effect do próximo render.** É por isso que, mesmo que você digite na entrada rapidamente, haverá no máximo um timeout agendado por vez. Edite a entrada algumas vezes e observe o console para ter uma ideia de como os Effects são limpos.

Digite algo na entrada e imediatamente pressione "Unmount the component". Observe como a desmontagem limpa o Effect do último render. Aqui, ele cancela o último timeout antes que ele tenha a chance de disparar.

Finalmente, edite o componente acima e comente a função de limpeza para que os timeouts não sejam cancelados. Tente digitar `abcde` rapidamente. O que você espera que aconteça em três segundos? `console.log(text)` dentro do timeout imprimirá o `text` *mais recente* e produzirá cinco logs de `abcde`? Tente para verificar sua intuição!

Três segundos depois, você deverá ver uma sequência de logs (`a`, `ab`, `abc`, `abcd`, e `abcde`) em vez de cinco logs de `abcde`. **Cada Effect "captura" o valor `text` de seu render correspondente.** Não importa que o estado `text` tenha mudado: um Effect de um render com `text = 'ab'` sempre verá `'ab'`. Em outras palavras, os Effects de cada render são isolados uns dos outros. Se você estiver curioso sobre como isso funciona, pode ler sobre [closures](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Closures).

<DeepDive>

#### Cada render tem seus próprios Effects {/*each-render-has-its-own-effects*/}

Você pode pensar em `useEffect` como "anexar" um pedaço de comportamento à saída do render. Considere este Effect:

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

Vamos ver o que exatamente acontece enquanto o usuário navega pelo aplicativo.

#### Render inicial {/*initial-render*/}

O usuário visita `<ChatRoom roomId="general" />`. Vamos [substituir mentalmente](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` por `'general'`:

```js
  // JSX para o primeiro render (roomId = "general")
  return <h1>Welcome to general!</h1>;
```

**O Effect também é parte da saída do render.** O Effect do primeiro render se torna:

```js
  // Effect para o primeiro render (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para o primeiro render (roomId = "general")
  ['general']
```

O React executa este Effect, que se conecta à sala de chat `'general'`.

#### Re-render com as mesmas dependências {/*re-render-with-same-dependencies*/}

Vamos dizer que `<ChatRoom roomId="general" />` re-renderiza. A saída JSX é a mesma:

```js
  // JSX para o segundo render (roomId = "general")
  return <h1>Welcome to general!</h1>;
```

O React vê que a saída do render não mudou, então ele não atualiza o DOM.

O Effect do segundo render se parece com isto:

```js
  // Effect para o segundo render (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para o segundo render (roomId = "general")
  ['general']
```

O React compara `['general']` do segundo render com `['general']` do primeiro render. **Como todas as dependências são as mesmas, o React *ignora* o Effect do segundo render.** Ele nunca é chamado.

#### Re-render com dependências diferentes {/*re-render-with-different-dependencies*/}

Então, o usuário visita `<ChatRoom roomId="travel" />`. Desta vez, o componente retorna um JSX diferente:

```js
  // JSX para o terceiro render (roomId = "travel")
  return <h1>Welcome to travel!</h1>;
```

O React atualiza o DOM para mudar `"Welcome to general"` para `"Welcome to travel"`.

O Effect do terceiro render se parece com isto:

```js
  // Effect para o terceiro render (roomId = "travel")
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependências para o terceiro render (roomId = "travel")
  ['travel']
```

O React compara `['travel']` do terceiro render com `['general']` do segundo render. Uma dependência é diferente: `Object.is('travel', 'general')` é `false`. O Effect não pode ser ignorado.

**Antes que o React possa aplicar o Effect do terceiro render, ele precisa limpar o último Effect que _foi_ executado.** O Effect do segundo render foi ignorado, então o React precisa limpar o Effect do primeiro render. Se você rolar para cima até o primeiro render, verá que sua limpeza chama `disconnect()` na conexão que foi criada com `createConnection('general')`. Isso desconecta o aplicativo da sala de chat `'general'`.

Depois disso, o React executa o Effect do terceiro render. Ele se conecta à sala de chat `'travel'`.

#### Desmontar {/*unmount*/}

Finalmente, digamos que o usuário navega para longe, e o componente `ChatRoom` é desmontado. O React executa a função de limpeza do último Effect. O último Effect foi do terceiro render. A limpeza do terceiro render destrói a conexão `createConnection('travel')`. Assim, o aplicativo se desconecta da sala `'travel'`.

#### Comportamentos apenas em desenvolvimento {/*development-only-behaviors*/}

Quando o [Modo Estrito](/reference/react/StrictMode) está ativado, o React remonta cada componente uma vez após a montagem (estado e DOM são preservados). Isso [ajuda você a encontrar Effects que precisam de limpeza](#step-3-add-cleanup-if-needed) e expõe bugs como condições de corrida precocemente. Além disso, o React remontará os Effects sempre que você salvar um arquivo em desenvolvimento. Ambos esses comportamentos são apenas para desenvolvimento.

</DeepDive>

<Recap>

- Ao contrário dos eventos, os Effects são causados pelo próprio render, e não por uma interação específica.
- Os Effects permitem que você sincronize um componente com algum sistema externo (API de terceiros, rede, etc.).
- Por padrão, os Effects são executados após cada render (incluindo o inicial).
- O React pulará o Effect se todas as suas dependências tiverem os mesmos valores da última renderização.
- Você não pode "escolher" suas dependências. Elas são determinadas pelo código dentro do Effect.
- Um array de dependências vazio (`[]`) corresponde à "montagem" do componente, ou seja, à sua adição à tela.
- No Modo Estrito, o React monta os componentes duas vezes (apenas em desenvolvimento!) para testar seus Effects.
- Se o seu Effect falhar devido à remontagem, você precisará implementar uma função de limpeza.
- O React chamará sua função de limpeza antes que o Effect seja executado na próxima vez e durante a desmontagem.

</Recap>

<Challenges>

#### Focar um campo na montagem {/*focus-a-field-on-mount*/}

Neste exemplo, o formulário renderiza um componente `<MyInput />`.

Use o método [`focus()`](https://developer.mozilla.org/pt-BR/docs/Web/API/HTMLElement/focus) da entrada para fazer `MyInput` receber foco automaticamente quando ele aparecer na tela. Já existe uma implementação comentada, mas ela não funciona exatamente. Descubra por que ela não funciona e corrija-a. (Se você está familiarizado com o atributo `autoFocus`, finja que ele não existe: estamos reimplementando a mesma funcionalidade do zero.)

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO: This doesn't quite work. Fix it.
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
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your name:
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
            Make it uppercase
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
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


Para verificar se sua solução funciona, pressione "Show form" e verifique se a entrada recebe foco (fica destacada e o cursor é colocado dentro). Pressione "Hide form" e "Show form" novamente. Verifique se a entrada é destacada novamente.

`MyInput` deve focar apenas _na montagem_, e não após cada renderização. Para verificar se o comportamento está correto, pressione "Show form" e, em seguida, pressione repetidamente a caixa de seleção "Make it uppercase". Clicar na caixa de seleção não deve focar a entrada acima dela.

<Solution>

Chamar `ref.current.focus()` durante a renderização está errado porque é um *efeito colateral*. Efeitos colaterais devem ser colocados dentro de um manipulador de eventos ou declarados com `useEffect`. Neste caso, o efeito colateral é *causado* pelo aparecimento do componente em vez de qualquer interação específica, então faz sentido colocá-lo em um Effect.

Para corrigir o erro, envolva a chamada `ref.current.focus()` em uma declaração de Effect. Em seguida, para garantir que este Effect seja executado apenas na montagem, em vez de após cada renderização, adicione o `[]` vazio como dependências a ele.

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
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your name:
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
            Make it uppercase
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
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

Pressione "Mostrar formulário" e observe que o segundo campo é focado automaticamente. Isso ocorre porque ambos os componentes `<MyInput />` tentam focar o campo interno. Quando você chama `focus()` para dois campos de entrada em sequência, o último sempre "vence".

Vamos supor que você queira focar o primeiro campo. O primeiro componente `MyInput` agora recebe uma prop booleana `shouldFocus` definida como `true`. Altere a lógica para que `focus()` seja chamado apenas se a prop `shouldFocus` recebida por `MyInput` for `true`.

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  // TODO: chame focus() apenas se shouldFocus for true.
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

Para verificar sua solução, pressione "Mostrar formulário" e "Ocultar formulário" repetidamente. Quando o formulário aparecer, apenas o *primeiro* campo de entrada deve ser focado. Isso ocorre porque o componente pai renderiza o primeiro campo de entrada com `shouldFocus={true}` e o segundo campo de entrada com `shouldFocus={false}`. Verifique também se ambos os campos de entrada ainda funcionam e se você pode digitar em ambos.

<Hint>

Você não pode declarar um Effect condicionalmente, mas seu Effect pode incluir lógica condicional.

</Hint>

<Solution>

Coloque a lógica condicional dentro do Effect. Você precisará especificar `shouldFocus` como uma dependência porque você a está usando dentro do Effect. (Isso significa que se o `shouldFocus` de algum campo mudar de `false` para `true`, ele será focado após a montagem.)

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

Este componente `Counter` exibe um contador que deve incrementar a cada segundo. Na montagem, ele chama [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) Isso faz com que `onTick` seja executado a cada segundo. A função `onTick` incrementa o contador.

No entanto, em vez de incrementar uma vez por segundo, ele incrementa duas vezes. Por quê? Encontre a causa do erro e corrija-o.

<Hint>

Lembre-se de que `setInterval` retorna um ID de intervalo, que você pode passar para [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) para parar o intervalo.

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

Quando o [Modo Estrito](/reference/react/StrictMode) está ativado (como nas sandboxes deste site), o React remonta cada componente uma vez em desenvolvimento. Isso faz com que o intervalo seja configurado duas vezes, e é por isso que a cada segundo o contador incrementa duas vezes.

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

Em desenvolvimento, o React ainda remontará seu componente uma vez para verificar se você implementou a limpeza corretamente. Assim, haverá uma chamada `setInterval`, seguida imediatamente por `clearInterval`, e `setInterval` novamente. Em produção, haverá apenas uma chamada `setInterval`. O comportamento visível ao usuário em ambos os casos é o mesmo: o contador incrementa uma vez por segundo.

</Solution>

#### Corrigir busca dentro de um Effect {/*fix-fetching-inside-an-effect*/}

Este componente exibe a biografia da pessoa selecionada. Ele carrega a biografia chamando uma função assíncrona `fetchBio(person)` na montagem e sempre que `person` muda. Essa função assíncrona retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que eventualmente resolve para uma string. Quando a busca é concluída, ela chama `setBio` para exibir essa string abaixo da caixa de seleção.

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


Há um erro neste código. Comece selecionando "Alice". Em seguida, selecione "Bob" e, imediatamente depois, selecione "Taylor". Se você fizer isso rápido o suficiente, notará o erro: Taylor é selecionado, mas o parágrafo abaixo diz "This is Bob's bio.".

Por que isso acontece? Corrija o erro dentro deste Effect.

<Hint>

Se um Effect busca algo assincronamente, ele geralmente precisa de limpeza.

</Hint>

<Solution>

Para acionar o erro, as coisas precisam acontecer nesta ordem:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')`
- **A busca por `'Taylor'` é concluída *antes* da busca por `'Bob'`**
- O Effect da renderização de `'Taylor'` chama `setBio('This is Taylor’s bio')`
- A busca por `'Bob'` é concluída
- O Effect da renderização de `'Bob'` chama `setBio('This is Bob’s bio')`

É por isso que você vê a biografia de Bob, mesmo que Taylor esteja selecionado. Erros como este são chamados de [condições de corrida](https://en.wikipedia.org/wiki/Race_condition) porque duas operações assíncronas estão "competindo" umas com as outras e podem chegar em uma ordem inesperada.

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
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

Cada Effect de renderização tem sua própria variável `ignore`. Inicialmente, a variável `ignore` é definida como `false`. No entanto, se um Effect for limpo (como quando você seleciona uma pessoa diferente), sua variável `ignore` se tornará `true`. Assim, não importa mais em que ordem as requisições são concluídas. Apenas o Effect da última pessoa terá `ignore` definido como `false`, então ele chamará `setBio(result)`. Past Effects foram limpos, então a verificação `if (!ignore)` os impedirá de chamar `setBio`:

- Selecionar `'Bob'` aciona `fetchBio('Bob')`
- Selecionar `'Taylor'` aciona `fetchBio('Taylor')` **e limpa o Effect anterior (de Bob)**
- A busca por `'Taylor'` é concluída *antes* da busca por `'Bob'`
- O Effect da renderização de `'Taylor'` chama `setBio('This is Taylor’s bio')`
- A busca por `'Bob'` é concluída
- O Effect da renderização de `'Bob'` **não faz nada porque sua flag `ignore` foi definida como `true`**

Além de ignorar o resultado de uma chamada de API desatualizada, você também pode usar [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) para cancelar as requisições que não são mais necessárias. No entanto, por si só, isso não é suficiente para proteger contra condições de corrida. Mais etapas assíncronas poderiam ser encadeadas após a busca, então usar uma flag explícita como `ignore` é a maneira mais confiável de corrigir esse tipo de problema.

</Solution>

</Challenges>