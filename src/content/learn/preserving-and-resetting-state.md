title: Preserving and Resetting State
---

<Intro>

O estado é isolado entre componentes. O React rastreia qual estado pertence a qual componente com base em sua posição na árvore de renderização. Você pode controlar quando preservar o estado e quando redefini-lo entre as re-renderizações.

</Intro>

<YouWillLearn>

* Quando o React escolhe preservar ou redefinir o estado
* Como forçar o React a redefinir o estado de um componente
* Como as chaves (`keys`) e os tipos afetam se o estado é preservado

</YouWillLearn>

## O estado está vinculado a uma posição na árvore de renderização {/*state-is-tied-to-a-position-in-the-tree*/}

O React constrói [árvores de renderização](learn/understanding-your-ui-as-a-tree#the-render-tree) para a estrutura de componentes em sua interface de usuário.

Quando você dá estado a um componente, pode pensar que o estado "vive" dentro do componente. Mas o estado é, na verdade, mantido dentro do React. O React associa cada pedaço de estado que está mantendo ao componente correto com base em onde esse componente se encontra na árvore de renderização.

Aqui, há apenas uma tag JSX `<Counter />`, mas ela é renderizada em duas posições diferentes:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Veja como eles se parecem em uma árvore:    

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado 'div' e tem dois filhos. Cada um dos filhos é rotulado 'Counter' e ambos contêm uma bolha de estado rotulada 'count' com valor 0.">

Árvore React

</Diagram>

</DiagramGroup>

**Estes são dois contadores separados porque cada um é renderizado em sua própria posição na árvore.** Você normalmente não precisa pensar sobre essas posições para usar o React, mas pode ser útil entender como funciona.

No React, cada componente na tela tem estado totalmente isolado. Por exemplo, se você renderizar dois componentes `Counter` lado a lado, cada um deles receberá seu próprio estado `score` e `hover` independente.

Tente clicar em ambos os contadores e observe que eles não afetam um ao outro:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Como você pode ver, quando um contador é atualizado, apenas o estado desse componente é atualizado:


<DiagramGroup>

<Diagram name="preserving_state_increment" height={248} width={441} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de estado rotulada 'count' com valor 0. O filho direito é rotulado 'Counter' e contém uma bolha de estado rotulada 'count' com valor 1. A bolha de estado do filho direito está destacada em amarelo para indicar que seu valor foi atualizado.">

Atualizando o estado

</Diagram>

</DiagramGroup>


O React manterá o estado pelo tempo que você renderizar o mesmo componente na mesma posição na árvore. Para ver isso, incremente ambos os contadores, depois remova o segundo componente desmarcando a caixa de seleção "Renderizar o segundo contador" e, em seguida, adicione-o novamente marcando-a:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />} 
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        Render the second counter
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Note como no momento em que você para de renderizar o segundo contador, seu estado desaparece completamente. Isso ocorre porque quando o React remove um componente, ele destrói seu estado.

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de estado rotulada 'count' com valor 0. O filho direito está faltando, e em seu lugar há uma imagem de 'poof' amarela, destacando o componente sendo excluído da árvore.">

Excluindo um componente

</Diagram>

</DiagramGroup>

Quando você marca "Renderizar o segundo contador", um segundo `Counter` e seu estado são inicializados do zero (`score = 0`) e adicionados ao DOM.

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de estado rotulada 'count' com valor 0. O filho direito é rotulado 'Counter' e contém uma bolha de estado rotulada 'count' com valor 0. Todo o nó filho direito está destacado em amarelo, indicando que ele acabou de ser adicionado à árvore.">

Adicionando um componente

</Diagram>

</DiagramGroup>

**O React preserva o estado de um componente enquanto ele estiver sendo renderizado em sua posição na árvore de UI.** Se ele for removido, ou um componente diferente for renderizado na mesma posição, o React descartará seu estado.

## O mesmo componente na mesma posição preserva o estado {/*same-component-at-the-same-position-preserves-state*/}

Neste exemplo, existem duas tags `<Counter />` diferentes:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Quando você marca ou desmarca a caixa de seleção, o estado do contador não é redefinido. Se `isFancy` for `true` ou `false`, você sempre terá um `<Counter />` como o primeiro filho do `div` retornado do componente raiz `App`:

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="Diagrama com duas seções separadas por uma seta fazendo a transição entre elas. Cada seção contém um layout de componentes com um pai rotulado 'App' contendo uma bolha de estado rotulada isFancy. Este componente tem um filho rotulado 'div', que leva a uma bolha de props contendo isFancy (destacada em roxo) passada para o único filho. O último filho é rotulado 'Counter' e contém uma bolha de estado com o rótulo 'count' e valor 3 em ambos os diagramas. Na seção esquerda do diagrama, nada está destacado e o valor do estado pai isFancy é falso. Na seção direita do diagrama, o valor do estado pai isFancy mudou para true e ele está destacado em amarelo, assim como a bolha de props abaixo, que também mudou seu valor isFancy para true.">

Atualizar o estado do `App` não redefine o `Counter` porque o `Counter` permanece na mesma posição

</Diagram>

</DiagramGroup>


É o mesmo componente na mesma posição, então da perspectiva do React, é o mesmo contador.

<Pitfall>

Lembre-se que **é a posição na árvore de UI — não no markup JSX — que importa para o React!** Este componente tem duas cláusulas `return` com diferentes tags JSX `<Counter />` dentro e fora do `if`:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          Use fancy styling
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Você pode esperar que o estado seja redefinido ao marcar a caixa de seleção, mas não é! Isso ocorre porque **ambas as tags `<Counter />` são renderizadas na mesma posição.** O React não sabe onde você coloca as condições em sua função. Tudo o que ele "vê" é a árvore que você retorna.

Em ambos os casos, o componente `App` retorna um `<div>` com `<Counter />` como o primeiro filho. Para o React, esses dois contadores têm o mesmo "endereço": o primeiro filho do primeiro filho da raiz. É assim que o React os associa entre a renderização anterior e a próxima, independentemente de como você estrutura sua lógica.

</Pitfall>

## Componentes diferentes na mesma posição redefinem o estado {/*different-components-at-the-same-position-reset-state*/}

Neste exemplo, marcar a caixa de seleção substituirá `<Counter>` por um `<p>`:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: