---
title: Preservando e Reiniciando o Estado
---
```
<Intro>

O estado é isolado entre os componentes. O React acompanha qual estado pertence a qual componente com base em seu lugar na árvore da interface do usuário. Você pode controlar quando preservar o estado e quando redefini-lo entre as renderizações.

</Intro>

<YouWillLearn>

* Quando o React escolhe preservar ou redefinir o estado
* Como forçar o React a redefinir o estado do componente
* Como as chaves e os tipos afetam se o estado é preservado

</YouWillLearn>

## O estado está ligado a uma posição na árvore de renderização {/*state-is-tied-to-a-position-in-the-tree*/}

O React constrói [árvores de renderização](learn/understanding-your-ui-as-a-tree#the-render-tree) para a estrutura do componente em sua interface do usuário.

Quando você dá estado a um componente, pode pensar que o estado "vive" dentro do componente. Mas o estado é realmente mantido dentro do React. O React associa cada parte do estado que está mantendo ao componente correto por onde esse componente fica na árvore de renderização.

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

Veja como eles se parecem como uma árvore:

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado como 'div' e tem dois filhos. Cada um dos filhos é rotulado como 'Counter' e ambos contêm uma bolha de estado rotulada como 'count' com o valor 0.">

Árvore React

</Diagram>

</DiagramGroup>

**Estes são dois contadores separados porque cada um é renderizado em sua própria posição na árvore.** Você normalmente não precisa pensar nessas posições para usar o React, mas pode ser útil entender como ele funciona.

No React, cada componente na tela tem estado totalmente isolado. Por exemplo, se você renderizar dois componentes `Counter` lado a lado, cada um deles terá seus próprios estados `score` e `hover` independentes.

Tente clicar em ambos os contadores e observe que eles não se afetam:

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

<Diagram name="preserving_state_increment" height={248} width={441} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado como 'div' e tem dois filhos. O filho esquerdo é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0. O filho direito é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 1. A bolha de estado do filho direito é destacada em amarelo para indicar que seu valor foi atualizado.">

Atualizando o estado

</Diagram>

</DiagramGroup>

O React manterá o estado por quanto tempo você renderizar o mesmo componente na mesma posição na árvore. Para ver isso, incremente ambos os contadores, remova o segundo componente desmarcando a caixa de seleção "Renderizar o segundo contador" e, em seguida, adicione-o novamente marcando-a novamente:

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

Observe como no momento em que você para de renderizar o segundo contador, seu estado desaparece completamente. Isso ocorre porque, quando o React remove um componente, ele destrói seu estado.

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado como 'div' e tem dois filhos. O filho esquerdo é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0. O filho direito está faltando e, em seu lugar, há uma imagem amarela de 'poof', destacando o componente sendo excluído da árvore.">

Excluindo um componente

</Diagram>

</DiagramGroup>

Quando você marca "Renderizar o segundo contador", um segundo `Counter` e seu estado são inicializados do zero (`score = 0`) e adicionados ao DOM.

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado como 'div' e tem dois filhos. O filho esquerdo é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0. O filho direito é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0. O nó filho direito inteiro é destacado em amarelo, indicando que ele foi adicionado à árvore.">

Adicionando um componente

</Diagram>

</DiagramGroup>

**O React preserva o estado de um componente enquanto ele está sendo renderizado em sua posição na árvore da interface do usuário.** Se ele for removido ou um componente diferente for renderizado na mesma posição, o React descarta seu estado.

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

Quando você marca ou desmarca a caixa de seleção, o estado do contador não é redefinido. Se `isFancy` for `true` ou `false`, você sempre terá um `<Counter />` como o primeiro filho da `div` retornada do componente `App` raiz:

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="Diagrama com duas seções separadas por uma seta que faz a transição entre elas. Cada seção contém um layout de componentes com um pai rotulado como 'App' contendo uma bolha de estado rotulada como isFancy. Este componente tem um filho rotulado como 'div', que leva a uma bolha de propriedade contendo isFancy (destacado em roxo) passado para o único filho. O último filho é rotulado como 'Counter' e contém uma bolha de estado com o rótulo 'count' e o valor 3 em ambos os diagramas. Na seção esquerda do diagrama, nada é destacado e o valor do estado pai isFancy é falso. Na seção direita do diagrama, o valor do estado pai isFancy mudou para verdadeiro e está destacado em amarelo, e a bolha de propriedades abaixo também, que também mudou seu valor isFancy para verdadeiro.">

A atualização do estado do `App` não redefine o `Counter` porque o `Counter` permanece na mesma posição

</Diagram>

</DiagramGroup>

É o mesmo componente na mesma posição, então, da perspectiva do React, é o mesmo contador.

<Pitfall>

Lembre-se de que **é a posição na árvore da interface do usuário - não na marcação JSX - que importa para o React!** Este componente tem duas cláusulas `return` com tags JSX `<Counter />` diferentes dentro e fora do `if`:

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

Em ambos os casos, o componente `App` retorna um `<div>` com `<Counter />` como o primeiro filho. Para o React, esses dois contadores têm o mesmo "endereço": o primeiro filho do primeiro filho da raiz. É assim que o React os combina entre as renderizações anterior e seguinte, independentemente de como você estrutura sua lógica.

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
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Aqui, você alterna entre _diferentes_ tipos de componentes na mesma posição. Inicialmente, o primeiro filho do `<div>` continha um `Counter`. Mas quando você trocou por um `p`, o React removeu o `Counter` da árvore da interface do usuário e destruiu seu estado.

<DiagramGroup>

<Diagram name="preserving_state_diff_pt1" height={290} width={753} alt="Diagrama com três seções, com uma seta fazendo a transição entre cada seção. A primeira seção contém um componente React rotulado como 'div' com um único filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 3. A seção do meio tem o mesmo pai 'div', mas o componente filho foi excluído, indicado por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado como 'p', destacado em amarelo.">

Quando `Counter` muda para `p`, o `Counter` é excluído e o `p` é adicionado

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_pt2" height={290} width={753} alt="Diagrama com três seções, com uma seta fazendo a transição entre cada seção. A primeira seção contém um componente React rotulado como 'p'. A seção do meio tem o mesmo pai 'div', mas o componente filho foi excluído, indicado por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 0, destacada em amarelo.">

Ao alternar de volta, o `p` é excluído e o `Counter` é adicionado

</Diagram>

</DiagramGroup>

Além disso, **quando você renderiza um componente diferente na mesma posição, ele redefine o estado de toda a sua subárvore.** Para ver como isso funciona, incremente o contador e marque a caixa de seleção:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
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

O estado do contador é redefinido quando você clica na caixa de seleção. Embora você renderize um `Counter`, o primeiro filho da `div` muda de um `section` para um `div`. Quando o filho `section` foi removido do DOM, toda a árvore abaixo dele (incluindo o `Counter` e seu estado) também foi destruída.

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Diagrama com três seções, com uma seta fazendo a transição entre cada seção. A primeira seção contém um componente React rotulado como 'div' com um único filho rotulado como 'section', que tem um único filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 3. A seção do meio tem o mesmo pai 'div', mas os componentes filhos foram excluídos, indicados por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado como 'div', destacado em amarelo, também com um novo filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 0, todos destacados em amarelo.">

Quando `section` muda para `div`, o `section` é excluído e o novo `div` é adicionado

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt2" height={350} width={794} alt="Diagrama com três seções, com uma seta fazendo a transição entre cada seção. A primeira seção contém um componente React rotulado como 'div' com um único filho rotulado como 'div', que tem um único filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 0. A seção do meio tem o mesmo pai 'div', mas os componentes filhos foram excluídos, indicados por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado como 'section', destacado em amarelo, também com um novo filho rotulado como 'Counter' contendo uma bolha de estado rotulada como 'count' com o valor 0, todos destacados em amarelo.">

Ao alternar de volta, o `div` é excluído e o novo `section` é adicionado

</Diagram>

</DiagramGroup>

Como regra geral, **se você deseja preservar o estado entre as renderizações, a estrutura de sua árvore precisa "combinar"** de uma renderização para outra. Se a estrutura for diferente, o estado será destruído porque o React destrói o estado quando remove um componente da árvore.

<Pitfall>

É por isso que você não deve aninhar definições de funções de componentes.

Aqui, a função do componente `MyTextField` é definida *dentro* de `MyComponent`:

<Sandpack>

```js
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}
```

</Sandpack>

Toda vez que você clica no botão, o estado da entrada desaparece! Isso ocorre porque uma função `MyTextField` *diferente* é criada para cada renderização de `MyComponent`. Você está renderizando um componente *diferente* na mesma posição, então o React redefine todo o estado abaixo. Isso leva a bugs e problemas de desempenho. Para evitar esse problema, **sempre declare funções de componentes no nível superior e não aninhe suas definições.**

</Pitfall>

## Redefinindo o estado na mesma posição {/*resetting-state-at-the-same-position*/}

Por padrão, o React preserva o estado de um componente enquanto ele permanece na mesma posição. Normalmente, isso é exatamente o que você deseja, então faz sentido como o comportamento padrão. Mas, às vezes, você pode querer redefinir o estado de um componente. Considere este aplicativo que permite que dois jogadores acompanhem suas pontuações durante cada rodada:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
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
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Atualmente, quando você altera o jogador, a pontuação é preservada. Os dois `Counter`s aparecem na mesma posição, então o React os vê como *o mesmo* `Counter` cuja propriedade `person` foi alterada.

Mas conceitualmente, neste aplicativo, eles devem ser dois contadores separados. Eles podem aparecer no mesmo lugar na interface do usuário, mas um é um contador para Taylor e outro é um contador para Sarah.

Existem duas maneiras de redefinir o estado ao alternar entre eles:

1. Renderizar componentes em posições diferentes
2. Dê a cada componente uma identidade explícita com `key`

### Opção 1: Renderizar um componente em posições diferentes {/*option-1-rendering-a-component-in-different-positions*/}

Se você deseja que esses dois `Counter`s sejam independentes, pode renderizá-los em duas posições diferentes:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
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
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

* Inicialmente, `isPlayerA` é `true`. Portanto, a primeira posição contém o estado `Counter` e a segunda está vazia.
* Quando você clica no botão "Próximo jogador", a primeira posição limpa, mas a segunda agora contém um `Counter`.

<DiagramGroup>

<Diagram name="preserving_state_diff_position_p1" height={375} width={504} alt="Diagrama com uma árvore de componentes React. O pai é rotulado como 'Scoreboard' com uma bolha de estado rotulada como isPlayerA com o valor 'true'. O único filho, disposto à esquerda, é rotulado como Counter com uma bolha de estado rotulada como 'count' e valor 0. Todo o filho esquerdo é destacado em amarelo, indicando que foi adicionado.">

Estado inicial

</Diagram>

<Diagram name="preserving_state_diff_position_p2" height={375} width={504} alt="Diagrama com uma árvore de componentes React. O pai é rotulado como 'Scoreboard' com uma bolha de estado rotulada como isPlayerA com o valor 'false'. A bolha de estado é destacada em amarelo, indicando que ela foi alterada. O filho esquerdo é substituído por uma imagem amarela de 'poof', indicando que foi excluído e há um novo filho à direita, destacado em amarelo, indicando que foi adicionado. O novo filho é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0.">

Clicando em "próximo"

</Diagram>

<Diagram name="preserving_state_diff_position_p3" height={375} width={504} alt="Diagrama com uma árvore de componentes React. O pai é rotulado como 'Scoreboard' com uma bolha de estado rotulada como isPlayerA com o valor 'true'. A bolha de estado é destacada em amarelo, indicando que ela foi alterada. Há um novo filho à esquerda, destacado em amarelo, indicando que foi adicionado. O novo filho é rotulado como 'Counter' e contém uma bolha de estado rotulada como 'count' com o valor 0. O filho direito é substituído por uma imagem amarela de 'poof', indicando que foi excluído.">

Clicando em "próximo" novamente

</Diagram>

</DiagramGroup>

O estado de cada `Counter` é destruído toda vez que ele é removido do DOM. É por isso que eles são redefinidos toda vez que você clica no botão.

Esta solução é conveniente quando você tem apenas alguns componentes independentes renderizados no mesmo lugar. Neste exemplo, você tem apenas dois, então não é um incômodo renderizar os dois separadamente no JSX.

### Opção 2: Redefinindo o estado com uma chave {/*option-2-resetting-state-with-a-key*/}

Há também outra maneira, mais genérica, de redefinir o estado de um componente.

Você pode ter visto `key`s ao [renderizar listas.](/learn/rendering-lists#keeping-list-items-in-order-with-key) As chaves não são apenas para listas! Você pode usar chaves para fazer o React distinguir entre quaisquer componentes. Por padrão, o React usa a ordem dentro do pai ("primeiro contador", "segundo contador") para discernir entre os componentes. Mas as chaves permitem que você diga ao React que este não é apenas um contador *primeiro* ou um contador *segundo*, mas um contador específico - por exemplo, o contador de *Taylor*. Dessa forma, o React saberá o contador de *Taylor* onde quer que ele apareça na árvore!

Neste exemplo, os dois `<Counter />`s não compartilham estado, embora apareçam no mesmo lugar no JSX:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
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
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

```css
h1 {