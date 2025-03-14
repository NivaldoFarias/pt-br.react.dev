---
title: Enfileirando uma Série de Atualizações de Estado
---

<Intro>

Definir uma variável de estado vai enfileirar outra renderização. Mas, às vezes, você pode querer realizar várias operações no valor antes de enfileirar a próxima renderização. Para fazer isso, ajuda entender como o React faz *batch* de atualizações de estado.

</Intro>

<YouWillLearn>

* O que é "batch" e como o React o usa para processar múltiplas atualizações de estado
* Como aplicar várias atualizações à mesma variável de estado em sequência

</YouWillLearn>

## React faz *batch* de atualizações de estado {/*react-batches-state-updates*/}

Você pode esperar que clicar no botão "+3" incremente o contador três vezes porque ele chama `setNumber(number + 1)` três vezes:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

No entanto, como você pode se lembrar da seção anterior, [os valores do estado de cada renderização são fixos](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time), então o valor de `number` dentro do event handler da primeira renderização é sempre `0`, não importa quantas vezes você chamar `setNumber(1)`:

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

Mas há um outro fator em jogo aqui. **O React espera até *todo* código nos event handlers ser executado antes de processar suas atualizações de estado.** É por isso que a re-renderização só acontece *depois* de todas essas chamadas `setNumber()`.

Isso pode te lembrar de um garçom anotando um pedido no restaurante. Um garçom não corre para a cozinha na menção do seu primeiro prato! Em vez disso, eles deixam você terminar seu pedido, permitem que você faça alterações nele e até mesmo aceitam pedidos de outras pessoas na mesa.

<Illustration src="/images/docs/illustrations/i_react-batching.png"  alt="Um cursor elegante em um restaurante faz e pede várias vezes com o React, interpretando o papel de garçom. Depois que ela chama setState() várias vezes, o garçom anota a última que ela solicitou como seu pedido final." />

Isso permite que você atualize várias variáveis de estado - mesmo de vários componentes - sem acionar muitas [re-renderizações.](/learn/render-and-commit#re-renders-when-state-updates) Mas isso também significa que a UI não será atualizada até _depois_ que seu event handler, e qualquer código nele, for concluído. Esse comportamento, também conhecido como **batching,** faz com que seu aplicativo React seja executado muito mais rápido. Ele também evita lidar com renderizações confusas "inacabadas", onde apenas algumas das variáveis foram atualizadas.

**O React não faz *batch* em *vários* eventos intencionais, como cliques** — cada clique é tratado separadamente. Tenha certeza de que o React só faz *batch* quando é geralmente seguro fazê-lo. Isso garante que, por exemplo, se o primeiro clique no botão desabilitar um formulário, o segundo clique não o enviará novamente.

## Atualizando o mesmo estado várias vezes antes da próxima renderização {/*updating-the-same-state-multiple-times-before-the-next-render*/}

É um caso de uso incomum, mas se você quiser atualizar a mesma variável de estado várias vezes antes da próxima renderização, em vez de passar o *próximo valor do estado* como `setNumber(number + 1)`, você pode passar uma *função* que calcula o próximo estado com base no anterior na fila, como `setNumber(n => n + 1)`. É uma forma de dizer ao React para "fazer algo com o valor do estado" em vez de apenas substituí-lo.

Tente incrementar o contador agora:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Aqui, `n => n + 1` é chamada de **função updater.** Quando você a passa para um definidor de estado:

1.  O React enfileira essa função para ser processada depois que todo o outro código no event handler foi executado.
2.  Durante a próxima renderização, o React percorre a fila e fornece o estado final atualizado.

```js
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

Veja como o React funciona nessas linhas de código ao executar o event handler:

1.  `setNumber(n => n + 1)`: `n => n + 1` é uma função. O React a adiciona à fila.
2.  `setNumber(n => n + 1)`: `n => n + 1` é uma função. O React a adiciona à fila.
3.  `setNumber(n => n + 1)`: `n => n + 1` é uma função. O React a adiciona à fila.

Quando você chama `useState` durante a próxima renderização, o React percorre a fila.  O estado `number` anterior era `0`, então é isso que o React passa para a primeira função de atualização como o argumento `n`. Então, o React pega o valor de retorno da sua função updater anterior e o passa para a próxima atualização como `n`, e assim por diante:

| atualização enfileirada | `n` | retorna |
|--------------|---------|-----|
| `n => n + 1` | `0` | `0 + 1 = 1` |
| `n => n + 1` | `1` | `1 + 1 = 2` |
| `n => n + 1` | `2` | `2 + 1 = 3` |

O React armazena `3` como o resultado final e o retorna de `useState`.

É por isso que clicar em "+3" no exemplo acima incrementa corretamente o valor em 3.
### O que acontece se você atualizar o estado depois de substituí-lo {/*what-happens-if-you-update-state-after-replacing-it*/}

E quanto a este event handler? O que você acha que `number` será na próxima renderização?

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Veja o que esse event handler diz ao React para fazer:

1.  `setNumber(number + 5)`: `number` é `0`, então `setNumber(0 + 5)`. O React adiciona *"substituir por `5`"* à sua fila.
2.  `setNumber(n => n + 1)`: `n => n + 1` é uma função updater. O React adiciona *essa função* à sua fila.

Durante a próxima renderização, o React percorre a fila de estado:

|   atualização na fila       | `n` | retorna |
|--------------|---------|-----|
| "substituir por `5`" | `0` (não usado) | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |

O React armazena `6` como o resultado final e o retorna de `useState`.

<Note>

Você pode ter notado que `setState(5)` realmente funciona como `setState(n => 5)`, mas `n` não é usado!

</Note>

### O que acontece se você substituir o estado depois de atualizá-lo {/*what-happens-if-you-replace-state-after-updating-it*/}

Vamos tentar mais um exemplo. O que você acha que `number` será na próxima renderização?

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Veja como o React funciona nessas linhas de código ao executar este event handler:

1.  `setNumber(number + 5)`: `number` é `0`, então `setNumber(0 + 5)`. O React adiciona *"substituir por `5`"* à sua fila.
2.  `setNumber(n => n + 1)`: `n => n + 1` é uma função updater. O React adiciona *essa função* à sua fila.
3.  `setNumber(42)`: O React adiciona *"substituir por `42`"* à sua fila.

Durante a próxima renderização, o React percorre a fila de estado:

|   atualização na fila       | `n` | retorna |
|--------------|---------|-----|
| "substituir por `5`" | `0` (não usado) | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |
| "substituir por `42`" | `6` (não usado) | `42` |

Então o React armazena `42` como o resultado final e o retorna de `useState`.

Para resumir, veja como você pode pensar no que está passando para o definidor de estado `setNumber`:

*   **Uma função updater** (por exemplo, `n => n + 1`) é adicionada à fila.
*   **Qualquer outro valor** (por exemplo, o número `5`) adiciona "substituir por `5`" à fila, ignorando o que já está enfileirado.

Após a conclusão do event handler, o React acionará uma re-renderização.  Durante a re-renderização, o React processará a fila. Funções updater são executadas durante a renderização, então **funções updater devem ser [puras](/learn/keeping-components-pure)** e apenas *retornar* o resultado. Não tente definir o estado de dentro delas ou executar outros efeitos colaterais. No Modo Strict, o React executará cada função updater duas vezes (mas descartará o segundo resultado) para ajudá-lo a encontrar erros.

### Convenções de nomenclatura {/*naming-conventions*/}

É comum nomear o argumento da função updater pelas primeiras letras da variável de estado correspondente:

```js
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

Se você prefere um código mais detalhado, outra convenção comum é repetir o nome completo da variável de estado, como `setEnabled(enabled => !enabled)`, ou usar um prefixo como `setEnabled(prevEnabled => !prevEnabled)`.

<Recap>

*  Definir o estado não altera a variável na renderização existente, mas solicita uma nova renderização.
*  O React processa as atualizações de estado depois que os event handlers terminam de ser executados. Isso é chamado de batching.
*  Para atualizar algum estado várias vezes em um evento, você pode usar a função updater `setNumber(n => n + 1)`.

</Recap>

<Challenges>

#### Corrigir um contador de solicitações {/*fix-a-request-counter*/}

Você está trabalhando em um aplicativo de mercado de arte que permite ao usuário enviar vários pedidos de um item de arte ao mesmo tempo. Cada vez que o usuário pressiona o botão "Comprar", o contador "Pendente" deve aumentar em um. Após três segundos, o contador "Pendente" deve diminuir e o contador "Concluído" deve aumentar.

No entanto, o contador "Pendente" não se comporta como o esperado. Quando você pressiona "Comprar", ele diminui para `-1` (o que não deveria ser possível!). E se você clicar rápido duas vezes, ambos os contadores parecem se comportar de forma imprevisível.

Por que isso acontece? Corrija ambos os contadores.

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>
        Pending: {pending}
      </h3>
      <h3>
        Completed: {completed}
      </h3>
      <button onClick={handleClick}>
        Buy     
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

<Solution>

Dentro do event handler `handleClick`, os valores de `pending` e `completed` correspondem ao que eles eram no momento do evento de clique.  Para a primeira renderização, `pending` era `0`, então `setPending(pending - 1)` se torna `setPending(-1)`, o que está errado. Como você quer *incrementar* ou *decrementar* os contadores, em vez de defini-los com um valor concreto determinado durante o clique, você pode, em vez disso, passar as funções updater:

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(p => p + 1);
    await delay(3000);
    setPending(p => p - 1);
    setCompleted(c => c + 1);
  }

  return (
    <>
      <h3>
        Pending: {pending}
      </h3>
      <h3>
        Completed: {completed}
      </h3>
      <button onClick={handleClick}>
        Buy     
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

Isso garante que, ao incrementar ou decrementar um contador, você o faça em relação ao seu *último* estado, em vez do estado que estava no momento do clique.

</Solution>

#### Implemente a fila de estado sozinho {/*implement-the-state-queue-yourself*/}

Nesse desafio, você reimplementará uma pequena parte do React do zero! Não é tão difícil quanto parece.

Role pela visualização do sandbox. Observe que ele mostra **quatro casos de teste.** Eles correspondem aos exemplos que você viu anteriormente nesta página. Sua tarefa é implementar a função `getFinalState` para que ela retorne o resultado correto para cada um desses casos. Se você implementá-lo corretamente, todos os quatro testes devem passar.

Você receberá dois argumentos: `baseState` é o estado inicial (como `0`) e a `queue` é uma array que contém uma mistura de números (como `5`) e funções updater (como `n => n + 1`) na ordem em que foram adicionadas.

Sua tarefa é retornar o estado final, como as tabelas mostram nesta página!

<Hint>

Se você estiver com dificuldades, comece com esta estrutura de código:

```js
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // TODO: apply the updater function
    } else {
      // TODO: replace the state
    }
  }

  return finalState;
}
```

Preencha as linhas ausentes!

</Hint>

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Base state: <b>{baseState}</b></p>
      <p>Queue: <b>[{queue.join(', ')}]</b></p>
      <p>Expected result: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Your result: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'correct' :
          'wrong'
        })
      </p>
    </>
  );
}
```

</Sandpack>

<Solution>

Este é o algoritmo exato descrito nesta página que o React usa para calcular o estado final:

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // Apply the updater function.
      finalState = update(finalState);
    } else {
      // Replace the next state.
      finalState = update;
    }
  }

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Base state: <b>{baseState}</b></p>
      <p>Queue: <b>[{queue.join(', ')}]</b></p>
      <p>Expected result: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Your result: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'correct' :
          'wrong'
        })
      </p>
    </>
  );
}
```

</Sandpack>

Agora você sabe como essa parte do React funciona!

</Solution>

</Challenges>
```