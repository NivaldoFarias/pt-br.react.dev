---
title: Preservando e Reinicializando o State
---

<Intro>

O state é isolado entre componentes. O React acompanha qual state pertence a qual componente com base em sua posição na árvore da UI. Você pode controlar quando preservar o state e quando reinicializá-lo entre as re-renderizações.

</Intro>

<YouWillLearn>

* Quando o React escolhe preservar ou reinicializar o state
* Como forçar o React a resetar o state do componente
* Como as chaves e os tipos afetam se o state é preservado

</YouWillLearn>

## State está amarrado à posição na árvore de renderização {/*state-is-tied-to-a-position-in-the-tree*/}

O React constrói [árvores de renderização](learn/understanding-your-ui-as-a-tree#the-render-tree) para a estrutura de componentes em sua UI.

Quando você dá state a um componente, você pode pensar que o state "vive" dentro do componente. Mas o state é, na verdade, mantido dentro do React. O React associa cada pedaço de state que está mantendo com o componente correto por onde esse componente se encontra na árvore de renderização.

Aqui, existe somente uma tag JSX `<Counter />`, mas ela é renderizada em duas posições diferentes:

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
        Adicionar um
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

Veja como elas se parecem como uma árvore:

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="Diagrama de uma árvore de componentes React. O nó raiz é rotulado 'div' e tem dois filhos. Cada um dos filhos é rotulado 'Counter' e ambos contêm uma bolha de state rotulada 'count' com o valor 0.">

Árvore React

</Diagram>

</DiagramGroup>

**Estes são dois contadores diferentes porque cada um é renderizado em sua própria posição na árvore.** Você geralmente não precisa pensar nessas posições para usar o React, mas pode ser útil entender como ele funciona.

No React, cada componente na tela tem state totalmente isolado. Por exemplo, se você renderizar dois componentes `Counter` lado a lado, cada um deles obterá seus próprios states `score` e `hover` independentes.

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
        Adicionar um
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

Como você pode ver, quando um contador é atualizado, apenas o state desse componente é atualizado:

<DiagramGroup>

<Diagram name="preserving_state_increment" height={248} width={441} alt="Diagrama de uma árvore de componentes do React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com o valor 0. O filho direito é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com o valor 1. A bolha de state do filho direito é destacada em amarelo para indicar que seu valor foi atualizado.">

Atualizando o state

</Diagram>

</DiagramGroup>

O React manterá o state por quanto tempo você renderizar o mesmo componente na mesma posição na árvore. Para ver isso, incremente ambos os contadores, remova o segundo componente desmarcando a checkbox "Renderizar o segundo contador" e, em seguida, adicione-o de volta marcando-a novamente:

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
        Renderizar o segundo contador
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
        Adicione um
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

Observe como no momento em que você para de renderizar o segundo contador, seu state desaparece completamente. Isso ocorre porque, quando o React remove um componente, ele destrói seu state.

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="Diagrama de uma árvore de componentes do React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com o valor 0. O filho direito está faltando, e em seu lugar está uma imagem amarela de 'poof', destacando o componente sendo deletado da árvore.">

Excluindo um componente

</Diagram>

</DiagramGroup>

Quando você marca "Renderizar o segundo contador", um segundo `Counter` e seu state são inicializados do zero (`score = 0`) e adicionados ao DOM.

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="Diagrama de uma árvore de componentes do React. O nó raiz é rotulado 'div' e tem dois filhos. O filho esquerdo é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com o valor 0. O filho direito é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com o valor 0. O nó filho direito inteiro é destacado em amarelo, indicando que ele acabou de ser adicionado à árvore.">

Adicionando um componente

</Diagram>

</DiagramGroup>

**O React preserva o state de um componente enquanto ele está sendo renderizado em sua posição na árvore da UI.** Se ele for removido, ou um componente diferente for renderizado na mesma posição, o React descarta seu state.

## Mesmo componente na mesma posição preserva o state {/*same-component-at-the-same-position-preserves-state*/}

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
        Usar estilo sofisticado
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
        Adicionar um
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

Quando você marca ou desmarca a checkbox, o state do contador não é reinicializado. Se `isFancy` é `true` ou `false`, você sempre tem um `<Counter />` como o primeiro filho do `div` retornado do componente `App` raiz:

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="Diagrama com duas seções separadas por uma seta transitando entre elas. Cada seção contém um layout de componentes com um pai rotulado 'App' contendo uma bolha state rotulada isFancy. Este componente tem um filho rotulado 'div', que leva a uma bolha de props contendo isFancy (destacado em roxo) passando para baixo para o único filho. O último filho é rotulado 'Counter' e contém uma bolha de state com o rótulo 'count' e valor 3 em ambos os diagramas. Na seção esquerda do diagrama, nada é destacado e o valor do state pai isFancy é false. Na seção direita do diagrama, o valor do state pai isFancy mudou para true e está destacado em amarelo, e também a bolha de props abaixo, que também mudou seu valor isFancy para true.">

A atualização do state do `App` não reinicia o `Counter` porque o `Counter` permanece na mesma posição

</Diagram>

</DiagramGroup>

É o mesmo componente na mesma posição, então, na perspectiva do React, é o mesmo contador.

<Pitfall>

Lembre-se que **é a posição na árvore da UI - não na marcação JSX - que importa para o React!** Este componente tem duas cláusulas `return` com tags JSX `<Counter />` diferentes dentro e fora do `if`:

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
          Usar estilo sofisticado
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
          Usar estilo sofisticado
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
        Adicionar um
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

Você pode esperar que o state seja reinicializado quando marcar a checkbox, mas não é! Isso ocorre porque **ambos os `<Counter />` tags são renderizados na mesma posição.** O React não sabe onde você coloca as condições em sua função. Tudo o que ele "vê" é a árvore que você retorna.

Em ambos os casos, o componente `App` retorna um `<div>` com `<Counter />` como o primeiro filho. Para o React, esses dois contadores têm o mesmo "endereço": o primeiro filho do primeiro filho da raiz. É assim que o React os combina entre as renderizações anteriores e as próximas, independentemente de como você estrutura sua lógica.

</Pitfall>

## Componentes diferentes na mesma posição reiniciam o state {/*different-components-at-the-same-position-reset-state*/}

Nesse exemplo, marcar a checkbox irá substituir `<Counter>` por um `<p>`:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>Até mais!</p> 
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
        Fazer uma pausa
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
        Adicionar um
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

Aqui, você alterna entre tipos de componentes _diferentes_ na mesma posição. Inicialmente, o primeiro filho do `<div>` continha um `Counter`. Mas quando você trocou por um `p`, o React removeu o `Counter` da árvore da UI e destruiu seu state.

<DiagramGroup>

<Diagram name="preserving_state_diff_pt1" height={290} width={753} alt="Diagrama com três seções, com uma seta transitando entre cada seção. A primeira seção contém um componente React rotulado 'div' com um único filho rotulado 'Counter' contendo uma bolha de state rotulada 'count' com valor 3. A seção do meio tem o mesmo pai 'div', mas o componente filho agora foi excluído, indicado por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado 'p', destacado em amarelo.">

Quando `Counter` muda para `p`, o `Counter` é excluído e o `p` é adicionado

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_pt2" height={290} width={753} alt="Diagrama com três seções, com uma seta transitando entre cada seção. A primeira seção contém um componente React rotulado 'p'. A seção do meio tem o mesmo pai 'div', mas o componente filho agora foi excluído, indicado por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado 'Counter' contendo uma bolha de state rotulada 'count' com valor 0, destacada em amarelo.">

Ao alternar de volta, o `p` é excluído e o `Counter` é adicionado

</Diagram>

</DiagramGroup>

Além disso, **quando você renderiza um componente diferente na mesma posição, ele reinicia o state de toda a sua subárvore.** Para ver como isso funciona, incremente o contador e marque a checkbox:

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
        Usar estilo sofisticado
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
        Adicionar um
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

O state do contador é reinicializado quando você clica na checkbox. Embora você renderize um `Counter`, o primeiro filho do `div` muda de um `div` para um `section`. Quando o `div` filho foi removido do DOM, toda a árvore abaixo dele (incluindo o `Counter` e seu state) também foi destruída.

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Diagrama com três seções, com uma seta transitando entre cada seção. A primeira seção contém um componente React rotulado 'div' com um único filho rotulado 'section', que tem um único filho rotulado 'Counter' contendo uma bolha de state rotulada 'count' com valor 3. A seção do meio tem o mesmo pai 'div', mas os componentes filhos agora foram excluídos, indicados por uma imagem amarela 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado 'div', destacado em amarelo, também com um novo filho rotulado 'Counter" contendo uma bolha de state rotulada 'count' com valor 0, todos destacados em amarelo.">

Quando `section` muda para `div`, o `section` é excluído e o novo `div` é adicionado

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt2" height={350} width={794} alt="Diagrama com três seções, com uma seta transitando entre cada seção. A primeira seção contém um componente React rotulado 'div' com um único filho rotulado 'div', que tem um único filho rotulado 'Counter' contendo uma bublle de state rotulada 'count' com valor 0. A seção do meio tem o mesmo pai 'div', mas os componentes filhos agora foram excluídos, indicados por uma imagem amarela de 'prova'. A terceira seção tem o mesmo pai 'div' novamente, agora com um novo filho rotulado 'section', destacado em amarelo, também com um novo filho rotulado 'Counter' contendo uma bublle de state rotulada 'count' com valor 0, todos em amarelo.">

Ao alternar de volta, o `div` é excluído e o novo `section` é adicionado

</Diagram>

</DiagramGroup>

Como regra geral, **se você deseja preservar o state entre as re-renderizações, a estrutura da sua árvore precisa "combinar"** de uma renderização para outra. Se a estrutura for diferente, o state é destruído, porque o React destrói o state quando remove um componente da árvore.

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
      }}>Clicado {counter} vezes</button>
    </>
  );
}
```

</Sandpack>

Toda vez que você clica no botão, o state da entrada desaparece! Isso ocorre porque uma função `MyTextField` *diferente* é criada para cada renderização de `MyComponent`. Você está renderizando um componente *diferente* na mesma posição, então o React reinicia todo o state abaixo. Isso leva a bugs e problemas de desempenho. Para evitar esse problema, **sempre declare as funções dos componentes no nível superior e não aninhe suas definições.**

</Pitfall>

## Reinicializando o state na mesma posição {/*resetting-state-at-the-same-position*/}

Por padrão, o React preserva o state de um componente enquanto ele permanece na mesma posição. Normalmente, isso é exatamente o que você deseja, por isso faz sentido como o comportamento padrão. Mas, às vezes, você pode querer reinicializar o state de um componente. Considere este app que permite que dois jogadores mantenham o controle de suas pontuações durante cada turno:

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
        Próximo jogador!
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
        Adicionar um
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

Atualmente, quando você altera o jogador, a pontuação é preservada. Os dois `Counter`s aparecem na mesma posição, então o React os vê como *o mesmo* `Counter` cuja prop `person` foi alterada.

Mas, conceitualmente, neste app eles deveriam ser dois contadores separados. Eles podem aparecer no mesmo lugar na UI, mas um é um contador para Taylor, e outro é um contador para Sarah.

Existem duas maneiras de reinicializar o state ao alternar entre eles:

1. Renderize componentes em posições diferentes
2. Dê a cada componente uma identidade explícita com `key`

### Opção 1: Renderizando um componente em posições diferentes {/*option-1-rendering-a-component-in-different-positions*/}

Se você quiser que esses dois `Counter`s sejam independentes, você pode renderizá-los em duas posições diferentes:

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
        Próximo jogador!
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
        Adicionar um
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

* Inicialmente, `isPlayerA` é `true`. Então, a primeira posição contém o state do `Counter`, e a segunda está vazia.
* Quando você clica no botão "Próximo jogador", a primeira posição limpa, mas a segunda agora contém um `Counter`.

<DiagramGroup>

<Diagram name="preserving_state_diff_position_p1" height={375} width={504} alt="Diagrama com uma árvore de componentes do React. O pai é rotulado 'Scoreboard' com uma bolha de state rotulada isPlayerA com o valor 'true'. O único filho, disposto à esquerda, é rotulado Counter com uma bolha de state rotulada 'count' e valor 0. Tudo do filho esquerdo está destacado em amarelo, indicando que foi adicionado.">

Estado inicial

</Diagram>

<Diagram name="preserving_state_diff_position_p2" height={375} width={504} alt="Diagrama com uma árvore de componentes React. O pai é rotulado 'Scoreboard' com uma bolha de state rotulada isPlayerA com o valor 'false'. A bolha de state está destacada em amarelo, indicando que ela foi alterada. O filho esquerdo é substituído por uma imagem amarela de 'poof', indicando que foi excluído e há um novo filho à direita, destacado em amarelo, indicando que foi adicionado. O novo filho é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com valor 0.">

Clicando em "próximo"

</Diagram>

<Diagram name="preserving_state_diff_position_p3" height={375} width={504} alt="Diagrama com uma árvore de componentes React. O pai é rotulado 'Scoreboard' com uma bolha de state rotulada isPlayerA com o valor 'true'. A bolha de state está destacada em amarelo, indicando que ela foi alterada. Há um novo filho à esquerda, destacado em amarelo, indicando que ele foi adicionado. O novo filho é rotulado 'Counter' e contém uma bolha de state rotulada 'count' com valor 0. O filho direito é substituído por uma imagem amarela de 'poof' indicando que foi excluído.">

Clicando em "próximo" novamente

</Diagram>

</DiagramGroup>

O state de cada `Counter` é destruído cada vez que ele é removido do DOM. É por isso que eles redefinem toda vez que você clica no botão.

Esta solução é conveniente quando você tem apenas alguns componentes independentes renderizados no mesmo lugar. Neste exemplo, você só tem dois, então não é uma dor de cabeça renderizá-los separadamente no JSX.

### Opção 2: Reiniciando o state com uma key {/*option-2-resetting-state-with-a-key*/}

Existe também outra forma, mais genérica, de reinicializar o state de um componente.

Você pode ter visto `key`s ao [renderizar listas.](/learn/rendering-lists#keeping-list-items-in-order-with-key) As chaves não são apenas para listas! Você pode usar chaves para fazer o React distinguir entre quaisquer componentes. Por padrão, o React usa a ordem dentro do pai ("primeiro contador", "segundo contador") para discernir entre os componentes. Mas as chaves permitem que você diga ao React que este não é apenas um contador *primeiro* ou um contador *segundo*, mas um contador específico - por exemplo, o contador de *Taylor*. Dessa forma, o React saberá o contador de *Taylor* onde quer que ele apareça na árvore!

Neste exemplo, os dois `<Counter />`s não compartilham o state, mesmo que apareçam no mesmo lugar no JSX:

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
        Próximo jogador!
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
``````javascript
```
## Resetando um formulário com uma chave {/*resetting-a-form-with-a-key*/}

Resetar o estado com uma chave é particularmente útil ao lidar com formulários.

Neste aplicativo de chat, o componente `<Chat>` contém o estado da entrada de texto:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chat to ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Send to {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Tente digitar algo na entrada e, em seguida, pressione "Alice" ou "Bob" para escolher um destinatário diferente. Você notará que o estado da entrada é preservado porque o `<Chat>` é renderizado na mesma posição na árvore.

**Em muitos aplicativos, este pode ser o comportamento desejado, mas não em um aplicativo de chat!** Você não quer permitir que o usuário envie uma mensagem que ele já digitou para uma pessoa errada devido a um clique acidental. Para corrigi-lo, adicione uma `key`:

```js
<Chat key={to.id} contact={to} />
```

Isso garante que, ao selecionar um destinatário diferente, o componente `Chat` seja recriado do zero, incluindo qualquer estado na árvore abaixo dele. React também recriará os elementos DOM em vez de reutilizá-los.

Agora, alternar o destinatário sempre limpa o campo de texto:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chat to ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Send to {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<DeepDive>

#### Preservando o estado para componentes removidos {/*preserving-state-for-removed-components*/}

Em um aplicativo de chat real, você provavelmente gostaria de recuperar o estado da entrada quando o usuário seleciona o destinatário anterior novamente. Existem algumas maneiras de manter o estado "vivo" para um componente que não está mais visível:

- Você pode renderizar _todos_ os chats em vez de apenas o atual, mas ocultar todos os outros com CSS. Os chats não seriam removidos da árvore, portanto, seu estado local seria preservado. Esta solução funciona muito bem para UIs simples. Mas pode ficar muito lento se as árvores ocultas forem grandes e contiverem muitos nós DOM.
-  Você pode [elevar o estado](/learn/sharing-state-between-components) e manter a mensagem pendente para cada destinatário no componente pai. Desta forma, quando os componentes filhos forem removidos, não importa, porque é o pai que guarda as informações importantes. Esta é a solução mais comum.
- Você também pode usar uma fonte diferente além do estado do React. Por exemplo, você provavelmente quer que um rascunho de mensagem persista mesmo que o usuário feche a página acidentalmente. Para implementar isso, você pode fazer com que o componente `Chat` inicialize seu estado lendo do [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), e salvar os rascunhos lá também.

Não importa qual estratégia você escolher, um chat _com Alice_ é conceitualmente distinto de um chat _com Bob_, então faz sentido dar uma `key` para a árvore `<Chat>` com base no destinatário atual.

</DeepDive>

<Recap>

- O React mantém o estado enquanto o mesmo componente é renderizado na mesma posição.
- O estado não é mantido em tags JSX. Ele está associado à posição da árvore em que você coloca o JSX.
- Você pode forçar uma subárvore a redefinir seu estado fornecendo a ela uma chave diferente.
- Não aninhe definições de componentes, ou você irá redefinir o estado por acidente.

</Recap>

<Challenges>

#### Corrigir o desaparecimento do texto de entrada {/*fix-disappearing-input-text*/}

Este exemplo mostra uma mensagem quando você pressiona o botão. No entanto, pressionar o botão também redefine acidentalmente a entrada. Por que isso acontece? Corrija-o para que pressionar o botão não redefina o texto de entrada.

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>Dica: Sua cidade favorita?</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>Ocultar dica</button>
      </div>
    );
  }
  return (
    <div>
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>Mostrar dica</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

O problema é que o `Form` é renderizado em posições diferentes. No ramo `if`, ele é o segundo filho do `<div>`, mas no ramo `else`, ele é o primeiro filho. Portanto, o tipo de componente em cada posição muda. A primeira posição muda entre conter um `<p>` e um `Form`, enquanto a segunda posição muda entre conter um `Form` e um `button`. React redefine o estado toda vez que o tipo do componente muda.

A solução mais fácil é unificar os ramos para que o `Form` sempre renderize na mesma posição:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  return (
    <div>
      {showHint &&
        <p><i>Dica: Sua cidade favorita?</i></p>
      }
      <Form />
      {showHint ? (
        <button onClick={() => {
          setShowHint(false);
        }}>Ocultar dica</button>
      ) : (
        <button onClick={() => {
          setShowHint(true);
        }}>Mostrar dica</button>
      )}
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

Tecnicamente, você também pode adicionar `null` antes de `<Form />` no ramo `else` para corresponder à estrutura do ramo `if`:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>Dica: Sua cidade favorita?</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>Ocultar dica</button>
      </div>
    );
  }
  return (
    <div>
      {null}
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>Mostrar dica</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

Dessa forma, `Form` é sempre o segundo filho, então ele permanece na mesma posição e mantém seu estado. Mas esta abordagem é muito menos óbvia e introduz o risco de que outra pessoa remova esse `null`.

</Solution>

#### Trocar dois campos de formulário {/*swap-two-form-fields*/}

Este formulário permite que você insira o primeiro e o último nome. Ele também tem uma caixa de seleção que controla qual campo vai primeiro. Quando você marca a caixa de seleção, o campo "Sobrenome" aparecerá antes do campo "Nome".

Quase funciona, mas há um erro. Se você preencher a entrada "Nome" e marcar a caixa de seleção, o texto permanecerá na primeira entrada (que agora é "Sobrenome"). Corrija-o para que o texto de entrada *também* se mova quando você inverter a ordem.

<Hint>

Parece que para esses campos, sua posição dentro do pai não é suficiente. Existe alguma maneira de dizer ao React como corresponder o estado entre as renderizações?

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      Inverter ordem
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field label="Sobrenome" /> 
        <Field label="Nome" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field label="Nome" /> 
        <Field label="Sobrenome" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

Forneça uma `key` para ambos os componentes `<Field>` nos ramos `if` e `else`. Isso informa ao React como "combinar" o estado correto para qualquer `<Field>` mesmo que sua ordem dentro do pai mude:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      Inverter ordem
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="Sobrenome" /> 
        <Field key="firstName" label="Nome" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key="firstName" label="Nome" /> 
        <Field key="lastName" label="Sobrenome" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

</Solution>

#### Redefinir um formulário de detalhes {/*reset-a-detail-form*/}

Esta é uma lista de contatos editável. Você pode editar os detalhes do contato selecionado e, em seguida, pressionar "Salvar" para atualizá-lo ou "Redefinir" para desfazer suas alterações.

Ao selecionar um contato diferente (por exemplo, Alice), o estado é atualizado, mas o formulário continua mostrando os detalhes do contato anterior. Corrija-o para que o formulário seja redefinido quando o contato selecionado mudar.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        Nome:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Salvar
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        Redefinir
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

Forneça `key={selectedId}` ao componente `EditContact`. Desta forma, a alternância entre diferentes contatos irá redefinir o formulário:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        key={selectedId}
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        Nome:{ ' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Salvar
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        Redefinir
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

#### Limpar uma imagem enquanto ela está carregando {/*clear-an-image-while-its-loading*/}

Quando você pressiona "Próxima", o navegador começa a carregar a próxima imagem. No entanto, como ela é exibida na mesma tag `<img>`, por padrão, você ainda verá a imagem anterior até que a próxima seja carregada. Isso pode ser indesejável se for importante que o texto sempre corresponda à imagem. Altere-o para que, no momento em que você pressionar "Próxima", a imagem anterior seja limpa imediatamente.

<Hint>

Existe alguma maneira de dizer ao React para recriar o DOM em vez de reutilizá-lo?

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        Próxima
      </button>
      <h3>
        Imagem {index + 1} de {images.length}
      </h3>
      <img src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malásia',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisboa, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Espanha',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Chile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Schwyz, Suíça',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Praga, Chéquia',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Liubliana, Eslovênia',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

<Solution>

Você pode fornecer uma `key` para a tag `<img>`. Quando essa `key` muda, o React irá recriar o nó DOM `<img>` do zero. Isso causa um breve flash quando cada imagem carrega, então não é algo que você gostaria de fazer para cada imagem em seu aplicativo. Mas faz sentido se você deseja garantir que a imagem sempre corresponda ao texto.

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        Próxima
      </button>
      <h3>
        Imagem {index + 1} de {images.length}
      </h3>
      <img key={image.src} src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malásia',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisboa, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Espanha',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Chile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Schwyz, Suíça',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Praga, Chéquia',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Liubliana, Eslovênia',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

</Solution>

#### Corrigir estado fora do lugar na lista {/*fix-misplaced-state-in-the-list*/}

Nesta lista, cada `Contact` possui um estado que determina se "Mostrar email" foi pressionado. Pressione "Mostrar email" para Alice e, em seguida, marque a caixa de seleção "Mostrar em ordem inversa". Você notará que é o email de _Taylor_ que agora está expandido, mas o de Alice - que foi movido para a parte inferior - aparece recolhido.

Corrija-o para que o estado expandido seja associado a cada contato, independentemente da ordem escolhida.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          value={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Mostrar em ordem inversa
      </label>
      <ul>
        {displayedContacts.map((contact, i) =>
          <li key={i}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? 'Ocultar' : 'Mostrar'} email
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

O problema é que este exemplo estava usando o índice como uma `key`:

```js
{displayedContacts.map((contact, i) =>
  <li key={i}>
```

No entanto, você deseja que o estado seja associado a _cada contato em particular_.

Usar o ID do contato como uma `key` em vez disso corrige o problema:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          value={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Mostrar em ordem inversa
      </label>
      <ul>
        {displayedContacts.map(contact =>
          <li key={contact.id}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? 'Ocultar' : 'Mostrar'} email
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

O estado está associado à posição da árvore. Uma `key` permite que você especifique uma posição nomeada em vez de depender da ordem.

</Solution>

</Challenges>
```