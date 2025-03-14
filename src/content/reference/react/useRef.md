---
title: useRef
---

<Intro>

`useRef` é um Hook do React que permite que você referencie um valor que não é necessário para renderização.

```js
const ref = useRef(initialValue)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `useRef(initialValue)` {/*useref*/}

Chame `useRef` no nível superior do seu componente para declarar uma [ref.](/learn/referencing-values-with-refs)

```js
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

*   `initialValue`: O valor que você deseja que a propriedade `current` do objeto ref tenha inicialmente. Pode ser um valor de qualquer tipo. Esse argumento é ignorado após a renderização inicial.

#### Retorna {/*returns*/}

`useRef` retorna um objeto com uma única propriedade:

*   `current`: Inicialmente, ele é definido para o `initialValue` que você passou. Você pode defini-lo para outra coisa mais tarde. Se você passar o objeto ref para o React como um atributo `ref` para um nó JSX, o React definirá sua propriedade `current`.

Nas próximas renderizações, `useRef` retornará o mesmo objeto.

#### Ressalvas {/*caveats*/}

*   Você pode mutar a propriedade `ref.current`. Diferente do state, ela é mutável. No entanto, se ela contiver um objeto que é usado para renderização (por exemplo, um pedaço do seu state), então você não deve mutar esse objeto.
*   Quando você muda a propriedade `ref.current`, o React não re-renderiza seu componente. O React não está ciente de quando você o muda porque uma ref é um objeto JavaScript simples.
*   Não escreva _ou leia_ `ref.current` durante a renderização, exceto para [inicialização.](#avoiding-recreating-the-ref-contents) Isso torna o comportamento do seu componente imprevisível.
*   No Modo Strict, o React vai **chamar a função do seu componente duas vezes** para [ajudá-lo a encontrar impurezas acidentais.](/reference/react/useState#my-initializer-or-updater-function-runs-twice) Isso é um comportamento de desenvolvimento e não afeta a produção. Cada objeto ref será criado duas vezes, mas uma das versões será descartada. Se a função do seu componente for pura (como deve ser), isso não deve afetar o comportamento.

---

## Uso {/*usage*/}

### Referenciando um valoro com uma ref {/*referencing-a-value-with-a-ref*/}

Chame `useRef` no nível superior do seu componente para declarar uma ou mais [refs.](/learn/referencing-values-with-refs)

```js [[1, 4, "intervalRef"], [3, 4, "0"]]
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

`useRef` retorna um <CodeStep step={1}>objeto ref</CodeStep> com uma única <CodeStep step={2}>propriedade `current`</CodeStep> inicialmente definida para o <CodeStep step={3}>valor inicial</CodeStep> que você forneceu.

Nas próximas renderizações, `useRef` retornará o mesmo objeto. Você pode mudar sua propriedade `current` para armazenar informação e lê-la mais tarde. Isso pode te lembrar de [state](/reference/react/useState), mas existe uma diferença importante.

**Mudar uma ref não aciona uma re-renderização.** Isso signifca que refs são perfeitas para armazenar informação que não afeta a saída visual do seu componente. Por exemplo, se você precisa armazenar um [ID de interval](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) e recuperar ele mais tarde, você pode colocá-lo em uma ref. Para atualizar o valor dentro da ref, você precisa mudar manualmente sua <CodeStep step={2}>propriedade `current`</CodeStep>:

```js [[2, 5, "intervalRef.current"]]
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

Mais tarde, você pode ler esse ID de interval da ref, para que possa chamar [clear o interval](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval):

```js [[2, 2, "intervalRef.current"]]
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

Ao usar uma ref, você garante que:

-   Você pode **armazenar informação** entre re-renderizações (diferente de variáveis regulares, que resetam em cada renderização).
-   Mudá-la **não aciona uma re-renderização** (diferente das variáveis de state, que acionam uma re-renderização).
-   A **informação é local** para cada cópia do seu componente (diferente das variáveis fora, que são compartilhadas).

Mudar uma ref não aciona um re-render, então refs não são apropriadas para armazenar informação que você quer mostrar na tela. Use state para isso no lugar. Leia mais sobre [escolhendo entre `useRef` e `useState`.](/learn/referencing-values-with-refs#differences-between-refs-and-state)

<Recipes titleText="Exemplos de referenciar um valor com useRef" titleId="examples-value">

#### Contador de clique {/*click-counter*/}

Este componente usa uma ref para rastrear quantas vezes o botão foi clicado. Note que é okay usar uma ref ao invés de state aqui porque a contagem de cliques só é lida e escrita em um manipulador de evento.

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

</Sandpack>

Se você mostrar `{ref.current}` no JSX, o número não irá atualizar ao clicar. Isso é porque setar `ref.current` não aciona uma re-renderização. Informação que é usada para renderizar deve ser state ao invés disso.

<Solution />

#### Um cronômetro {/*a-stopwatch*/}

Esse exemplo usa uma combinação de state e refs. Ambos `startTime` e `now` são variáveis de state porque eles são usados para renderizar. Mas nós também precisamos manter um [ID de interval](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) para que possamos parar o interval ao pressionar o botão. Já que o ID de interval não é usado para renderizar, é apropriado mantê-lo em uma ref, e atualizá-la manualmente.

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Tempo passado: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Iniciar
      </button>
      <button onClick={handleStop}>
        Parar
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

<Pitfall>

**Não escreva _ou leia_ `ref.current` durante a renderização.**

React espera que o corpo do seu componente [se comporte como uma função pura](/learn/keeping-components-pure):

-   Se as entradas ([props](/learn/passing-props-to-a-component), [state](/learn/state-a-components-memory), e [context](/learn/passing-data-deeply-with-context)) são as mesmas, ele deve retornar exatamente o mesmo JSX.
-   Chamá-lo em uma ordem diferente ou com argumentos diferentes não deve afetar os resultados de outras chamadas.

Ler ou escrever uma ref **durante a renderização** quebra essas expectativas.

```js {3-4,6-7}
function MyComponent() {
  // ...
  // 🚩 Não escreva uma ref durante a renderização
  myRef.current = 123;
  // ...
  // 🚩 Não leia uma ref durante a renderização
  return <h1>{myOtherRef.current}</h1>;
}
```

Você pode ler ou escrever refs **de manipuladores de eventos ou effects no lugar**.

```js {4-5,9-10}
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ Você pode ler ou escrever refs em effects
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ Você pode ler ou escrever refs em manipuladores de eventos
    doSomething(myOtherRef.current);
  }
  // ...
}
```

Se você *tem que* ler [ou escrever](/reference/react/useState#storing-information-from-previous-renders) alguma coisa durante a renderização, [use state](/reference/react/useState) no lugar.

Quando você quebra essas regras, seu componente pode ainda funcionar, mas a maior parte das novas funcionalidades que estamos adicionando ao React vão depender dessas expectativas. Leia mais sobre [mantendo seus componentes puros.](/learn/keeping-components-pure#where-you-_can_-cause-side-effects)

</Pitfall>

---

### Manipulando o DOM com uma ref {/*manipulating-the-dom-with-a-ref*/}

É particularmente comum usar uma ref para manipular o [DOM.](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API) React tem suporte nativo para isso.

Primeiro, declare um <CodeStep step={1}>objeto ref</CodeStep> com um <CodeStep step={3}>valor inicial</CodeStep> de `null`:

```js [[1, 4, "inputRef"], [3, 4, "null"]]
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

Então passe seu objeto ref como o atributo `ref` para o JSX do nó do DOM que você quer manipular:

```js [[1, 2, "inputRef"]]
  // ...
  return <input ref={inputRef} />;
```

Após o React criar o nó DOM e colocá-lo na tela, React irá definir a <CodeStep step={2}>propriedade `current`</CodeStep> do seu objeto ref para aquele nó DOM. Agora você pode acessar o nó DOM do `input` e chamar os métodos, por exermplo, [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus):

```js [[2, 2, "inputRef.current"]]
  function handleClick() {
    inputRef.current.focus();
  }
```

O React irá definir a propriedade `current` de volta para `null` quando o nó for removido da tela.

Leia mais sobre [manipulação no DOM com refs.](/learn/manipulating-the-dom-with-refs)

<Recipes titleText="Exemplos de manipulação no DOM com useRef" titleId="examples-dom">

#### Focando um input de texto {/*focusing-a-text-input*/}

Nesse exemplo, clicar no botão irá focar o input:

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focar o input
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### Rolando uma imagem para a tela {/*scrolling-an-image-into-view*/}

Nesse exemplo, clicar no botão irá rolar uma imagem para a tela. Ele usa uma ref ao nó DOM da lista, e então chama a API do DOM [`querySelectorAll`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) para encontrar a imagem que queremos rolar.

<Sandpack>

```js
import { useRef } from 'react';

export default function CatFriends() {
  const listRef = useRef(null);

  function scrollToIndex(index) {
    const listNode = listRef.current;
    // Essa linha assume uma estrutura DOM particular:
    const imgNode = listNode.querySelectorAll('li > img')[index];
    imgNode.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToIndex(0)}>
          Neo
        </button>
        <button onClick={() => scrollToIndex(1)}>
          Millie
        </button>
        <button onClick={() => scrollToIndex(2)}>
          Bella
        </button>
      </nav>
      <div>
        <ul ref={listRef}>
          <li>
            <img
              src="https://placecats.com/neo/300/200"
              alt="Neo"
            />
          </li>
          <li>
            <img
              src="https://placecats.com/millie/200/200"
              alt="Millie"
            />
          </li>
          <li>
            <img
              src="https://placecats.com/bella/199/200"
              alt="Bella"
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

<Solution />

#### Tocando e pausando um vídeo {/*playing-and-pausing-a-video*/}

Esse exemplo usa uma ref para chamar [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) e [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) em um nó DOM do `<video>`.

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);
  const ref = useRef(null);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);

    if (nextIsPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? 'Pausar' : 'Tocar'}
      </button>
      <video
        width="250"
        ref={ref}
        onPlay={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
      >
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

<Solution />

#### Expondo uma ref para o seu próprio componente {/*exposing-a-ref-to-your-own-component*/}

Às vezes, você pode querer deixar o componente pai manipular o DOM dentro do seu componente. Por exemplo, talvez você esteja escrevendo um componente `MyInput`, mas você quer que o pai consiga focar no input (o qual o pai não tem acesso). Você pode criar uma `ref` no pai e passar a `ref` como prop para o componente filho. Leia um [walkthrough detalhado](/learn/manipulating-the-dom-with-refs#accessing-another-components-dom-nodes) aqui.

<Sandpack>

```js
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
};

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focar o input
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Evitando recriar o conteúdo da ref {/*avoiding-recreating-the-ref-contents*/}

React salva o valor initial da ref uma vez e ignora nas próximas renderizações.

```js
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

Embora o resultado de `new VideoPlayer()` seja usado apenas para a renderização inicial, você ainda está chamando esta função em cada render. Isso pode ser um desperdício se estiver criando objetos caros.

Para resolver isso, você pode inicializar a ref como esta também:

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

Normalmente, escrever ou ler `ref.current` durante a renderização não é permitido. No entanto, é bom nesse caso porque o resultado é sempre o mesmo, os a condição só executa durante a inicialização para que seja totalmente previsível.

<DeepDive>

#### Como evitar checagens null ao inicializar useRef mais tarde {/*how-to-avoid-null-checks-when-initializing-use-ref-later*/}

Se você usa um verificador de tipo e não quer sempre verificar `null`, você pode tentar um padrão como este:

```js
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // ...
```

Aqui, o próprio `playerRef` é anulável. No entanto, você deveria conseguir convencer seu verificador de tipos que não existe caso em que `getPlayer()` retorna `null`. Então use `getPlayer()` nos seus manipuladores de eventos.

</DeepDive>

---

## Solução de problemas {/*troubleshooting*/}

### Eu não consigo obter uma ref para um componente personalizado {/*i-cant-get-a-ref-to-a-custom-component*/}

Se você tentar passar uma `ref` para o seu próprio componente como esse:

```js
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

Você pode obter um erro no console:

<ConsoleBlock level="error">

TypeError: Não pode ler as propriedades de null

</ConsoleBlock>

Por padrão, seu componente não expõe refs para os nós DOM dentro deles.

Para consertar isso, encontre o componente que você quer obter a ref para:

```js
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```

E então adicione `ref` para a lista de props que seu componente aceita e passe `ref` como uma prop para o [componente nativo](/reference/react-dom/components/common) filho como este:

```js {1,6}
function MyInput({ value, onChange, ref }) {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
};

export default MyInput;
```

Então o componente pai pode obter uma ref para ele.

Leia mais sobre [acessando nós DOM de outro componente.](/learn/manipulating-the-dom-with-refs#accessing-another-components-dom-nodes)
```