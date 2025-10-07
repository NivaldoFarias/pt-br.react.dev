<Intro>

Alguns componentes precisam se sincronizar com sistemas externos. Por exemplo, você pode querer controlar um componente não-React com base no estado do React, configurar uma conexão com o servidor ou enviar um log de análise quando um componente aparece na tela. *Efeitos* permitem que você execute algum código após a renderização, para que possa sincronizar seu componente com algum sistema fora do React.

</Intro>

<YouWillLearn>

- O que são Efeitos
- Como os Efeitos diferem dos eventos
- Como declarar um Efeito em seu componente
- Como evitar reexecutar um Efeito desnecessariamente
- Por que os Efeitos são executados duas vezes em desenvolvimento e como corrigi-los

</YouWillLearn>

## O que são Efeitos e como eles diferem dos eventos? {/*what-are-effects-and-how-are-they-different-from-events*/}

Antes de chegar aos Efeitos, você precisa se familiarizar com dois tipos de lógica dentro dos componentes React:

- **Código de renderização** (introduzido em [Descrevendo a UI](/learn/describing-the-ui)) vive no nível superior do seu componente. É aqui que você pega as props e o estado, os transforma e retorna o JSX que deseja ver na tela. [O código de renderização deve ser puro.](/learn/keeping-components-pure) Como uma fórmula matemática, ele deve apenas _calcular_ o resultado, mas não fazer mais nada.

- **Manipuladores de eventos** (introduzidos em [Adicionando Interatividade](/learn/adding-interactivity)) são funções aninhadas dentro de seus componentes que _fazem_ coisas em vez de apenas calculá-las. Um manipulador de eventos pode atualizar um campo de entrada, enviar uma solicitação HTTP POST para comprar um produto ou navegar o usuário para outra tela. Manipuladores de eventos contêm ["efeitos colaterais"](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) (eles mudam o estado do programa) causados por uma ação específica do usuário (por exemplo, um clique em um botão ou digitação).

Às vezes, isso não é suficiente. Considere um componente `ChatRoom` que deve se conectar ao servidor de chat sempre que estiver visível na tela. Conectar-se a um servidor não é um cálculo puro (é um efeito colateral), portanto, não pode ocorrer durante a renderização. No entanto, não há um único evento específico como um clique que cause a exibição do `ChatRoom`.

***Efeitos* permitem que você especifique efeitos colaterais que são causados pela própria renderização, em vez de por um evento específico.** Enviar uma mensagem no chat é um *evento* porque é diretamente causado pelo usuário clicando em um botão específico. No entanto, configurar uma conexão de servidor é um *Efeito* porque deve acontecer independentemente de qual interação causou o aparecimento do componente. Efeitos são executados no final de um [commit](/learn/render-and-commit) após a atualização da tela. Este é um bom momento para sincronizar os componentes React com algum sistema externo (como rede ou uma biblioteca de terceiros).

<Note>

Aqui e mais adiante neste texto, "Efeito" capitalizado refere-se à definição específica do React acima, ou seja, um efeito colateral causado pela renderização. Para se referir ao conceito de programação mais amplo, diremos "efeito colateral".

</Note>


## Você pode não precisar de um Efeito {/*you-might-not-need-an-effect*/}

**Não se apresse em adicionar Efeitos aos seus componentes.** Tenha em mente que os Efeitos são tipicamente usados para "sair" do seu código React e sincronizar com algum sistema *externo*. Isso inclui APIs do navegador, widgets de terceiros, rede e assim por diante. Se o seu Efeito apenas ajusta algum estado com base em outro estado, [você pode não precisar de um Efeito.](/learn/you-might-not-need-an-effect)

## Como escrever um Efeito {/*how-to-write-an-effect*/}

Para escrever um Efeito, siga estas três etapas:

1. **Declare um Efeito.** Por padrão, seu Efeito será executado após cada [commit](/learn/render-and-commit).
2. **Especifique as dependências do Efeito.** A maioria dos Efeitos deve ser reexecutada *apenas quando necessário*, em vez de após cada renderização. Por exemplo, uma animação de fade-in deve ser acionada apenas quando um componente aparece. Conectar e desconectar de uma sala de chat só deve acontecer quando o componente aparece e desaparece, ou quando a sala de chat muda. Você aprenderá como controlar isso especificando *dependências*.
3. **Adicione limpeza, se necessário.** Alguns Efeitos precisam especificar como parar, desfazer ou limpar o que estavam fazendo. Por exemplo, "conectar" precisa de "desconectar", "inscrever" precisa de "cancelar inscrição" e "buscar" precisa de "cancelar" ou "ignorar". Você aprenderá como fazer isso retornando uma *função de limpeza*.

Vamos analisar cada uma dessas etapas em detalhes.

### Etapa 1: Declare um Efeito {/*step-1-declare-an-effect*/}

Para declarar um Efeito em seu componente, importe o [`useEffect` Hook](/reference/react/useEffect) do React:

```js
import { useEffect } from 'react';
```

Em seguida, chame-o no nível superior do seu componente e coloque algum código dentro do seu Efeito:

```js {2-4}
function MyComponent() {
  useEffect(() => {
    // O código aqui será executado após *cada* renderização
  });
  return <div />;
}
```

Toda vez que seu componente renderizar, o React atualizará a tela *e então* executará o código dentro do `useEffect`. Em outras palavras, **`useEffect` "atrasa" a execução de um trecho de código até que essa renderização seja refletida na tela.**

Vamos ver como você pode usar um Efeito para sincronizar com um sistema externo. Considere um componente React `<VideoPlayer>`. Seria bom controlar se ele está tocando ou pausado passando uma prop `isPlaying` para ele:

```js
<VideoPlayer isPlaying={isPlaying} />;
```

Seu componente `VideoPlayer` personalizado renderiza a tag [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) integrada do navegador:

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: faça algo com isPlaying
  return <video src={src} />;
}
```

No entanto, a tag `<video>` do navegador não possui uma prop `isPlaying`. A única maneira de controlá-la é chamar manualmente os métodos [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) e [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) no elemento DOM. **Você precisa sincronizar o valor da prop `isPlaying`, que indica se o vídeo _deve_ estar tocando no momento, com chamadas como `play()` e `pause()`.**

Precisaremos primeiro [obter uma ref](/learn/manipulating-the-dom-with-refs) para o nó DOM `<video>`.

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

Ao envolver a atualização do DOM em um Efeito, você permite que o React atualize a tela primeiro. Em seguida, seu Efeito é executado.

Quando seu componente `VideoPlayer` renderizar (seja pela primeira vez ou se re-renderizar), algumas coisas acontecerão. Primeiro, o React atualizará a tela, garantindo que a tag `<video>` esteja no DOM com as props corretas. Em seguida, o React executará seu Efeito. Finalmente, seu Efeito chamará `play()` ou `pause()` dependendo do valor de `isPlaying`.

Pressione Play/Pause várias vezes e veja como o player de vídeo permanece sincronizado com o valor `isPlaying`:

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

Note que controlar um player de vídeo é muito mais complexo na prática. Chamar `play()` pode falhar, o usuário pode reproduzir ou pausar usando os controles integrados do navegador, e assim por diante. Este exemplo é muito simplificado e incompleto.

<Pitfall>

Por padrão, os Efeitos são executados após *cada* renderização. É por isso que um código como este **produzirá um loop infinito:**

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

Os Efeitos são executados como um *resultado* da renderização. Definir o estado _desencadeia_ a renderização. Definir o estado imediatamente em um Efeito é como conectar uma tomada a si mesma. O Efeito é executado, ele define o estado, o que causa uma re-renderização, o que causa a execução do Efeito, ele define o estado novamente, isso causa outra re-renderização, e assim por diante.

Efeitos geralmente devem sincronizar seus componentes com um sistema *externo*. Se não houver um sistema externo e você quiser apenas ajustar algum estado com base em outro estado, [você pode não precisar de um Efeito.](/learn/you-might-not-need-an-effect)

</Pitfall>

### Etapa 2: Especifique as dependências do Efeito {/*step-2-specify-the-effect-dependencies*/}

Por padrão, os Efeitos são executados após *cada* renderização. Frequentemente, isso **não é o que você quer:**

- Às vezes, é lento. Sincronizar com um sistema externo nem sempre é instantâneo, então você pode querer pular a execução, a menos que seja necessário. Por exemplo, você não quer reconectar ao servidor de chat a cada pressionamento de tecla.
- Às vezes, está errado. Por exemplo, você não quer acionar uma animação de fade-in do componente a cada pressionamento de tecla. A animação deve ser reproduzida apenas uma vez quando o componente aparece pela primeira vez.

Para demonstrar o problema, aqui está o exemplo anterior com algumas chamadas `console.log` e um campo de texto que atualiza o estado do componente pai. Observe como digitar faz com que o Efeito seja reexecutado:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Chamando video.play()');
      ref.current.play();
    } else {
      console.log('Chamando video.pause()');
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

Você pode dizer ao React para **ignorar a reexecução desnecessária do Efeito** especificando um array de *dependências* como o segundo argumento para a chamada `useEffect`. Comece adicionando um array vazio `[]` ao exemplo acima na linha 14:

```js {3}
  useEffect(() => {
    // ...
  }, []);
```

Você verá um erro dizendo `React Hook useEffect has a missing dependency: 'isPlaying'` (Hook do React useEffect tem uma dependência ausente: 'isPlaying'):

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Chamando video.play()');
      ref.current.play();
    } else {
      console.log('Chamando video.pause()');
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

O problema é que o código dentro do seu Efeito *depende* da prop `isPlaying` para decidir o que fazer, mas essa dependência não foi explicitamente declarada. Para corrigir esse problema, adicione `isPlaying` ao array de dependências:

```js {2,7}
  useEffect(() => {
    if (isPlaying) { // É usado aqui...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...então deve ser declarado aqui!
```

Agora todas as dependências estão declaradas, então não há erro. Especificar `[isPlaying]` como o array de dependências diz ao React que ele deve pular a reexecução do seu Efeito se `isPlaying` for o mesmo que era durante a renderização anterior. Com essa alteração, digitar no input não faz o Efeito ser reexecutado, mas pressionar Play/Pause sim:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef