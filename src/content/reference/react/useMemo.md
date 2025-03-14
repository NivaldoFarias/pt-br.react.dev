# Guia de Estilo Universal

Este documento descreve as regras que devem ser aplicadas para **todos** os idiomas.
Quando estiver se referindo ao próprio `React`, use `o React`.

## IDs dos Títulos

Todos os títulos possuem IDs explícitos como abaixo:

```md
## Tente React {#try-react}
```

**Não** traduza estes IDs! Eles são usado para navegação e quebrarão se o documento for um link externo, como:

```md
Veja a [seção iniciando](/getting-started#try-react) para mais informações.
```

✅ FAÇA:

```md
## Tente React {#try-react}
```

❌ NÃO FAÇA:

```md
## Tente React {#tente-react}
```

Isto quebraria o link acima.

## Texto em Blocos de Código

Mantenha o texto em blocos de código sem tradução, exceto para os comentários. Você pode optar por traduzir o texto em strings, mas tenha cuidado para não traduzir strings que se refiram ao código!

Exemplo:

```js
// Example
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ FAÇA:

```js
// Exemplo
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ PERMITIDO:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

❌ NÃO FAÇA:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
// "root" se refere a um ID de elemento.
// NÃO TRADUZA
ReactDOM.render(element, document.getElementById('raiz'));
```

❌ DEFINITIVAMENTE NÃO FAÇA:

```js
// Exemplo
const elemento = <h1>Olá mundo</h1>;
ReactDOM.renderizar(elemento, documento.obterElementoPorId('raiz'));
```

## Links Externos

Se um link externo se referir a um artigo no [MDN] or [Wikipedia] e se houver uma versão traduzida em seu idioma em uma qualidade decente, opte por usar a versão traduzida.

[mdn]: https://developer.mozilla.org/pt-BR/
[wikipedia]: https://pt.wikipedia.org/wiki/Wikipédia:Página_principal

Exemplo:

```md
React elements are [immutable](https://en.wikipedia.org/wiki/Immutable_object).
```

✅ OK:

```md
Elementos React são [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável).
```

Para links que não possuem tradução (Stack Overflow, vídeos do YouTube, etc.), simplesmente use o link original.

## Traduções Comuns

Sugestões de palavras e termos:

| Palavra/Termo original | Sugestão                               |
| ---------------------- | -------------------------------------- |
| assertion              | asserção                               |
| at the top level       | na raiz                                |
| browser                | navegador                              |
| bubbling               | propagar                               |
| bug                    | erro                                   |
| caveats                | ressalvas                              |
| class component        | componente de classe                   |
| class                  | classe                                 |
| client                 | cliente                                |
| client-side            | lado do cliente                        |
| container              | contêiner                              |
| context                | contexto                               |
| controlled component   | componente controlado                  |
| debugging              | depuração                              |
| DOM node               | nó do DOM                              |
| event handler          | manipulador de eventos (event handler) |
| function component     | componente de função                   |
| handler                | manipulador                            |
| helper function        | função auxiliar                        |
| high-order components  | componente de alta-ordem               |
| key                    | chave                                  |
| library                | biblioteca                             |
| lowercase              | minúscula(s) / caixa baixa             |
| package                | pacote                                 |
| React element          | Elemento React                         |
| React fragment         | Fragmento React                        |
| render                 | renderizar (verb), renderizado (noun)  |
| server                 | servidor                               |
| server-side            | lado do servidor                       |
| siblings               | irmãos                                 |
| stateful component     | componente com estado                  |
| stateful logic         | lógica com estado                      |
| to assert              | afirmar                                |
| to wrap                | encapsular                             |
| troubleshooting        | solução de problemas                   |
| uncontrolled component | componente não controlado              |
| uppercase              | maiúscula(s) / caixa alta              |

## Conteúdo que não deve ser traduzido

- array
- arrow function
- bind
- bundle
- bundler
- callback
- camelCase
- DOM
- event listener
- framework
- hook
- log
- mock
- portal
- props
- ref
- release
- script
- single-page-apps
- state
- string
- string literal
- subscribe
- subscription
- template literal
- timestamps
- UI
- watcher
- widgets
- wrapper

---
title: useMemo
---

<Intro>

`useMemo` é um Hook do React que permite que você faça cache do resultado de um cálculo entre re-renderizações.

```js
const cachedValue = useMemo(calculateValue, dependencies)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `useMemo(calculateValue, dependencies)` {/*usememo*/}

Chame `useMemo` no nível superior do seu componente para fazer cache de um cálculo entre re-renderizações:

```js
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `calculateValue`: A função que calcula o valor que você quer fazer cache. Ela deve ser pura, não deve receber argumentos e deve retornar um valor de qualquer tipo. O React chamará sua função durante o render inicial. Nas próximas renderizações, o React retornará o mesmo valor novamente se as `dependencies` não tiverem sido alteradas desde a última renderização. Caso contrário, ele chamará `calculateValue`, retornará seu resultado e o armazenará para que possa ser reutilizado mais tarde.

* `dependencies`: A lista de todos os valores reativos referenciados dentro do código `calculateValue`. Valores reativos incluem props, state e todas as variáveis e funções declaradas diretamente dentro do corpo do seu componente. Se seu linter estiver [configurado para o React](/learn/editor-setup#linting), ele verificará se cada valor reativo é especificado corretamente como uma dependência. A lista de dependências deve ter um número constante de itens e ser escrita inline como `[dep1, dep2, dep3]`. O React comparará cada dependência com seu valor anterior usando a comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is).

#### Retorna {/*returns*/}

Na renderização inicial, `useMemo` retorna o resultado da chamada `calculateValue` sem argumentos.

Durante as próximas renderizações, ele retornará um valor já armazenado da última renderização (se as dependências não tiverem sido alteradas) ou chamará `calculateValue` novamente e retornará o resultado que `calculateValue` retornou.

#### Ressalvas {/*caveats*/}

* `useMemo` é um Hook, então você só pode chamá-lo **no nível superior do seu componente** ou dos seus próprios Hooks. Você não pode chamá-lo dentro de loops ou condições. Se você precisar disso, extraia um novo componente e mova o state para ele.
* No Modo Strict, o React vai **chamar sua função de cálculo duas vezes** para [ajudá-lo a encontrar impurezas acidentais.](#my-calculation-runs-twice-on-every-re-render) Este é um comportamento apenas para desenvolvimento e não afeta a produção. Se sua função de cálculo for pura (como deveria ser), isso não deve afetar sua lógica. O resultado de uma das chamadas será ignorado.
* O React **não vai descartar o valor em cache a menos que haja um motivo específico para isso.** Por exemplo, em desenvolvimento, o React descarta o cache quando você edita o arquivo do seu componente. Tanto em desenvolvimento quanto em produção, o React vai descartar o cache se seu componente suspender durante a montagem inicial. No futuro, o React pode adicionar mais recursos que se beneficiam de descartar o cache — por exemplo, se o React adicionar suporte integrado para listas virtualizadas no futuro, faria sentido descartar o cache para itens que saem da janela de visualização da tabela virtualizada. Isso deve ser bom se você confiar em `useMemo` apenas como uma otimização de performance. Caso contrário, uma [variável de state](/reference/react/useState#avoiding-recreating-the-initial-state) ou uma [ref](/reference/react/useRef#avoiding-recreating-the-ref-contents) pode ser mais apropriado.

<Note>

Fazer cache de valores de retorno como este também é conhecido como [*memoization*,](https://en.wikipedia.org/wiki/Memoization) razão pela qual este Hook é chamado de `useMemo`.

</Note>

---

## Uso {/*usage*/}

### Ignorando cálculos caros {/*skipping-expensive-recalculations*/}

Para fazer cache de um cálculo entre re-renderizações, encapsule-o em uma chamada para `useMemo` no nível superior do seu componente:

```js [[3, 4, "visibleTodos"], [1, 4, "() => filterTodos(todos, tab)"], [2, 4, "[todos, tab]"]]
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

Você precisa passar duas coisas para `useMemo`:

1. Uma <CodeStep step={1}>função de cálculo</CodeStep> que não recebe argumentos, como `() =>`, e retorna o que você queria calcular.
2. <CodeStep step={2}>Uma lista de dependências</CodeStep> incluindo cada valor dentro do seu componente que é usado dentro do seu cálculo.

Na renderização inicial, o <CodeStep step={3}>valor</CodeStep> que você obterá de `useMemo` será o resultado da chamada da sua <CodeStep step={1}>cálculo</CodeStep>.

Em cada renderização subsequente, o React comparará as <CodeStep step={2}>dependências</CodeStep> com as dependências que você passou durante a última renderização. Se nenhuma das dependências tiver mudado (comparada com [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), `useMemo` retornará o valor que você já calculou antes. Caso contrário, o React executará novamente seu cálculo e retornará o novo valor.

Em outras palavras, `useMemo` faz cache de um resultado de cálculo entre re-renderizações até que suas dependências mudem.

**Vamos analisar um exemplo para ver quando isso é útil.**

Por padrão, o React vai re-executar todo o corpo do seu componente toda vez que ele renderizar novamente. Por exemplo, se este `TodoList` atualizar seu state ou receber novas props de seu pai, a função `filterTodos` será re-executada:

```js {2}
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

Normalmente, isso não é um problema porque a maioria dos cálculos é muito rápida. No entanto, se você estiver filtrando ou transformando uma array grande ou fazendo algum cálculo caro, talvez queira pular a execução novamente se os dados não tiverem mudado. Se tanto `todos` quanto `tab` forem os mesmos de quando estavam durante a última renderização, encapsular o cálculo em `useMemo` como antes permite que você reutilize `visibleTodos` que você já calculou antes.

Este tipo de cache é chamado de *[memoization.](https://en.wikipedia.org/wiki/Memoization)*

<Note>

**Você só deve contar com `useMemo` como uma otimização de performance.** Se seu código não funcionar sem ele, encontre o problema subjacente e corrija-o primeiro. Então, você pode adicionar `useMemo` para melhorar a performance.

</Note>

<DeepDive>

#### Como saber se um cálculo é caro? {/*how-to-tell-if-a-calculation-is-expensive*/}

Em geral, a menos que você esteja criando ou percorrendo milhares de objetos, provavelmente não é caro. Se você quiser ter mais confiança, pode adicionar um console log para medir o tempo gasto em um trecho de código:

```js {1,3}
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

Realize a interação que você está medindo (por exemplo, digitando no input). Você verá então logs como `filter array: 0.15ms` em seu console. Se o tempo total registrado somar uma quantidade significativa (digamos, `1ms` ou mais), pode fazer sentido memorizar esse cálculo. Como um experimento, você pode então encapsular o cálculo em `useMemo` para verificar se o tempo total registrado diminuiu para essa interação ou não:

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // Ignorado se todos e guia não tiverem sido alterados
}, [todos, tab]);
console.timeEnd('filter array');
```

`useMemo` não vai deixar a *primeira* renderização mais rápida. Ele só ajuda a pular trabalho desnecessário em atualizações.

Tenha em mente que sua máquina é provavelmente mais rápida do que a dos seus usuários, por isso é uma boa ideia testar a performance com uma lentidão artificial. Por exemplo, o Chrome oferece uma opção de [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) para isso.

Observe também que medir a performance em desenvolvimento não fornecerá os resultados mais precisos. (Por exemplo, quando o [Modo Strict](/reference/react/StrictMode) estiver ativado, você verá cada componente renderizar duas vezes em vez de uma.) Para obter os tempos mais precisos, crie seu app para produção e teste-o em um dispositivo como seus usuários têm.

</DeepDive>

<DeepDive>

#### Você deve adicionar useMemo em todo lugar? {/*should-you-add-usememo-everywhere*/}

Se seu app for como este site e a maioria das interações for grosseira (como substituir uma página ou uma seção inteira), a memorização geralmente é desnecessária. Por outro lado, se seu app for mais como um editor de desenho, e a maioria das interações for granular (como mover formas), então você pode achar a memorização muito útil.

Otimizar com `useMemo` é valioso apenas em alguns casos:

- O cálculo que você está colocando em `useMemo` é notavelmente lento, e suas dependências raramente mudam.
- Você o passa como uma prop para um componente encapsulado em [`memo`.](/reference/react/memo) Você deseja pular a re-renderização se o valor não tiver sido alterado. A memorização permite que seu componente re-renderize apenas quando as dependências não são as mesmas.
- O valor que você está passando é usado posteriormente como uma dependência de algum Hook. Por exemplo, talvez outro valor de cálculo `useMemo` dependa dele. Ou talvez você esteja dependendo desse valor de [`useEffect.`](/reference/react/useEffect)

Não há benefício em encapsular um cálculo em `useMemo` em outros casos. Também não há nenhum dano significativo em fazer isso, então algumas equipes optam por não pensar em casos individuais e memorizam o máximo possível. A desvantagem dessa abordagem é que o código se torna menos legível. Além disso, nem toda memorização é eficaz: um único valor que é "sempre novo" é suficiente para quebrar a memorização de um componente inteiro.

**Na prática, você pode tornar muita memorização desnecessária seguindo alguns princípios:**

1. Quando um componente encapsula visualmente outros componentes, deixe-o [aceitar JSX como filhos.](/learn/passing-props-to-a-component#passing-jsx-as-children) Dessa forma, quando o componente wrapper atualiza seu próprio state, o React sabe que seus filhos não precisam re-renderizar.
1. Prefira o state local e não [suba o state](/learn/sharing-state-between-components) mais do que o necessário. Por exemplo, não mantenha o state transitório como formulários e se um item está em hover no topo da sua árvore ou em uma biblioteca de state global.
1. Mantenha sua [lógica de renderização pura.](/learn/keeping-components-pure) Se a re-renderização de um componente causar um problema ou produzir algum artefato visual perceptível, é um erro em seu componente! Corrija o erro em vez de adicionar memorização.
1. Evite [Effects desnecessários que atualizam o state.](/learn/you-might-not-need-an-effect) A maioria dos problemas de performance em apps React é causada por cadeias de atualizações originárias de Effects que causam a renderização repetida de seus componentes.
1. Tente [remover dependências desnecessárias de seus Effects.](/learn/removing-effect-dependencies) Por exemplo, em vez de memorização, geralmente é mais simples mover algum objeto ou uma função dentro de um Effect ou fora do componente.

Se uma interação específica ainda parecer lenta, [use o profiler do React Developer Tools](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) para ver quais componentes se beneficiariam mais da memorização e adicione a memorização onde for necessário. Esses princípios tornam seus componentes mais fáceis de depurar e entender, por isso é bom segui-los em qualquer caso. A longo prazo, estamos pesquisando [como fazer memorização granular automaticamente](https://www.youtube.com/watch?v=lGEMwh32soc) para resolver isso de uma vez por todas.

</DeepDive>

<Recipes titleText="A diferença entre useMemo e calcular um valor diretamente" titleId="examples-recalculation">

#### Ignorando o recálculo com `useMemo` {/*skipping-recalculation-with-usememo*/}

Neste exemplo, a implementação de `filterTodos` é **artificialmente lentificada** para que você possa ver o que acontece quando alguma função JavaScript que você está chamando durante a renderização é genuinamente lenta. Tente mudar as guias e alternar o tema.

Mudar as guias parece lento porque força o `filterTodos` lentificado a ser re-executado. Isso é esperado porque a `tab` foi alterada e, portanto, o cálculo inteiro *precisa* ser executado novamente. (Se você está curioso por que ele é executado duas vezes, é explicado [aqui.](#my-calculation-runs-twice-on-every-re-render))

Alterne o tema. **Graças ao `useMemo`, é rápido, apesar da lentidão artificial!** A chamada lenta `filterTodos` foi ignorada porque tanto `todos` quanto `tab` (que você passa como dependências para `useMemo`) não foram alterados desde a última renderização.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { useMemo } from 'react';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  return (
    <div className={theme}>
      <p><b>Nota: <code>filterTodos</code> é artificialmente lentificado!</b></p>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('[ARTIFICIALLY SLOW] Filtering ' + todos.length + ' todos for "' + tab + '" tab.');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Não faça nada por 500 ms para emular um código extremamente lento
  }

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### Sempre recalculando um valor {/*always-recalculating-a-value*/}

Neste exemplo, a implementação de `filterTodos` também é **artificialmente lentificada** para que você possa ver o que acontece quando alguma função JavaScript que você está chamando durante a renderização é genuinamente lenta. Tente mudar as guias e alternar o tema.

Ao contrário do exemplo anterior, alternar o tema também é lento agora! Isso ocorre porque **não há nenhuma chamada `useMemo` nesta versão,** então o `filterTodos` artificialmente lentificado é chamado em cada re-renderização. Ele é chamado mesmo se apenas `theme` foi alterado.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <ul>
        <p><b>Nota: <code>filterTodos</code> é artificialmente lentificado!</b></p>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('[ARTIFICIALLY SLOW] Filtering ' + todos.length + ' todos for "' + tab + '" tab.');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Não faça nada por 500 ms para emular um código extremamente lento
  }

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

No entanto, aqui está o mesmo código **com a lentidão artificial removida.** A falta de `useMemo` parece perceptível ou não?

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('Filtering ' + todos.length + ' todos for "' + tab + '" tab.');

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

Com bastante frequência, o código sem memorização funciona bem. Se suas interações forem rápidas o suficiente, talvez você não precise de memorização.

Você pode tentar aumentar o número de itens de to-do em `utils.js` e ver como o comportamento muda. Esse cálculo em particular não era muito caro para começar, mas se o número de todos crescer significativamente, a maior parte da sobrecarga estará na re-renderização em vez da filtragem. Continue lendo abaixo para ver como você pode otimizar a re-renderização com `useMemo`.

<Solution />

</Recipes>

---

### Ignorando a re-renderização de componentes {/*skipping-re-rendering-of-components*/}

Em alguns casos, `useMemo` também pode ajudá-lo a otimizar a performance de componentes filhos de re-renderização. Para ilustrar isso, digamos que este componente `TodoList` passe o `visibleTodos` como uma prop para o componente filho `List`:

```js {5}
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

Você notou que alternar a prop `theme` congela o app por um momento, mas se você remover `<List />` do seu JSX, ele parece rápido. Isso informa que vale a pena tentar otimizar o componente `List`.

**Por padrão, quando um componente é re-renderizado, o React re-renderiza todos os seus filhos recursivamente.** É por isso que, quando `TodoList` re-renderiza com um `theme` diferente, o componente `List` *também* re-renderiza. Isso é bom para componentes que não exigem muito cálculo para serem re-renderizados. Mas se você verificou que uma re-renderização é lenta, você pode dizer a `List` para pular a re-renderização quando suas props forem as mesmas da última renderização, encapsulando-a em [`memo`:](/reference/react/memo)

```js {3,5}
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**Com essa alteração, `List` ignorará a re-renderização se todas as suas props forem as *mesmas* da última renderização.** É aqui que fazer o cache do cálculo se torna importante! Imagine que você calculou `visibleTodos` sem `useMemo`:

```js {2-3,6-7}
export default function TodoList({ todos, tab, theme }) {
  // Toda vez que o tema mudar, isso será uma array diferente...
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ...então as props de List nunca serão as mesmas, e ele será re-renderizado toda vez */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**No exemplo acima, a função `filterTodos` sempre cria uma array *diferente,*** semelhante a como o literal de objeto `{}` sempre cria um novo objeto. Normalmente, isso não seria um problema, mas significa que as props de `List` nunca serão as mesmas, e sua otimização [`memo`](/reference/react/memo) não funcionará. É aqui que `useMemo` se torna útil:

```js {2-3,5,9-10}
export default function TodoList({ todos, tab, theme }) {
  // Diga ao React para fazer cache do seu cálculo entre re-renderizações...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...então, desde que essas dependências não mudem...
  );
  return (
    <div className={theme}>
      {/* ...List receberá as mesmas props e poderá pular a re-renderização */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**Ao encapsular o cálculo `visibleTodos` em `useMemo`, você garante que ele tenha o *mesmo* valor entre as re-renderizações** (até que as dependências mudem). Você não *precisa* encapsular um cálculo em `useMemo` a menos que você faça isso por algum motivo específico. Neste exemplo, o motivo é que você o passa para um componente encapsulado em [`memo`,](/reference/react/memo) e isso permite que ele pule a re-renderização. Existem alguns outros motivos para adicionar `useMemo`, que são descritos mais adiante nesta página.

<DeepDive>

#### Memoizando nós JSX individuais {/*memoizing-individual-jsx-nodes*/}

Em vez de encapsular `List` em [`memo`](/reference/react/memo), você pode encapsular o próprio nó JSX `<List />` em `useMemo`:

```js {3,6}
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return (
    <div className={theme}>
      {children}
    </div>
  );
}
```
``````
## Comportamento {#behavior}

O comportamento seria o mesmo. Se os `visibleTodos` não mudaram, o `List` não será renderizado novamente.

Um nó JSX como `<List items={visibleTodos} />` é um objeto como `{ type: List, props: { items: visibleTodos } }`. Criar este objeto é muito barato, mas o React não sabe se seu conteúdo é o mesmo da última vez ou não. É por isso que, por padrão, o React renderizará novamente o componente `List`.

No entanto, se o React vir o mesmo JSX exato que durante a renderização anterior, ele não tentará renderizar novamente seu componente. Isso ocorre porque os nós JSX são [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável). Um objeto de nó JSX não poderia ter mudado ao longo do tempo, então o React sabe que é seguro pular uma renderização. No entanto, para que isso funcione, o nó deve *realmente ser o mesmo objeto*, e não apenas parecer o mesmo no código. É isso que `useMemo` faz neste exemplo.

Envolver manualmente nós JSX em `useMemo` não é conveniente. Por exemplo, você não pode fazer isso condicionalmente. É geralmente por isso que você envolveria componentes com [`memo`](/reference/react/memo) em vez de envolver nós JSX.

</DeepDive>

<Recipes titleText="A diferença entre pular renderizações e sempre renderizar novamente" titleId="examples-rerendering">

#### Pulando a renderização com `useMemo` e `memo` {/*skipping-re-rendering-with-usememo-and-memo*/}

Neste exemplo, o componente `List` é **artificialmente desacelerado** para que você possa ver o que acontece quando um componente React que você está renderizando é realmente lento. Tente alternar as guias e alternar o tema.

Alternar as guias parece lento porque força a `List` desacelerada a renderizar novamente. Isso é esperado porque a `tab` mudou, e então você precisa refletir a nova escolha do usuário na tela.

Em seguida, tente alternar o tema. **Graças ao `useMemo` em conjunto com [`memo`](/reference/react/memo), é rápido apesar da lentidão artificial!** A `List` pulou a renderização novamente porque a array `visibleTodos` não mudou desde a última renderização. A array `visibleTodos` não mudou porque tanto `todos` quanto `tab` (que você passa como dependências para `useMemo`) não mudaram desde a última renderização.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        Todas
      </button>
      <button onClick={() => setTab('active')}>
        Ativas
      </button>
      <button onClick={() => setTab('completed')}>
        Completas
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Modo escuro
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import { useMemo } from 'react';
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  return (
    <div className={theme}>
      <p><b>Nota: <code>List</code> está artificialmente desacelerada!</b></p>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js src/List.js
import { memo } from 'react';

const List = memo(function List({ items }) {
  console.log('[ARTIFICIALLY SLOW] Renderizando <List /> com ' + items.length + ' itens');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Não faça nada por 500 ms para simular um código extremamente lento
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
});

export default List;
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Tarefa " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### Sempre renderizando novamente um componente {/*always-re-rendering-a-component*/}

Neste exemplo, a implementação de `List` também está **artificialmente desacelerada** para que você possa ver o que acontece quando algum componente React que você está renderizando é realmente lento. Tente alternar as guias e alternar o tema.

Ao contrário do exemplo anterior, alternar o tema também está lento agora! Isso ocorre porque **não há chamada `useMemo` nesta versão,** então o `visibleTodos` é sempre uma array diferente, e o componente `List` desacelerado não pode pular a renderização.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        Todas
      </button>
      <button onClick={() => setTab('active')}>
        Ativas
      </button>
      <button onClick={() => setTab('completed')}>
        Completas
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Modo escuro
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <p><b>Nota: <code>List</code> está artificialmente desacelerada!</b></p>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js src/List.js
import { memo } from 'react';

const List = memo(function List({ items }) {
  console.log('[ARTIFICIALLY SLOW] Renderizando <List /> com ' + items.length + ' itens');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Não faça nada por 500 ms para simular um código extremamente lento
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
});

export default List;
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Tarefa " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

No entanto, aqui está o mesmo código **com a lentidão artificial removida.** A falta de `useMemo` parece notável ou não?

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        Todas
      </button>
      <button onClick={() => setTab('active')}>
        Ativas
      </button>
      <button onClick={() => setTab('completed')}>
        Completas
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Modo escuro
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js src/List.js
import { memo } from 'react';

function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
}

export default memo(List);
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Tarefa " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

Muitas vezes, o código sem memoização funciona bem. Se suas interações forem rápidas o suficiente, você não precisa de memoização.

Tenha em mente que você precisa executar o React no modo de produção, desativar as [Ferramentas de Desenvolvimento do React](/learn/react-developer-tools) e usar dispositivos semelhantes aos que seus usuários de aplicativos têm para ter uma sensação realista do que realmente está desacelerando seu aplicativo.

<Solution />

</Recipes>

---

### Impedindo que um Effect dispare com muita frequência {/*preventing-an-effect-from-firing-too-often*/}

Às vezes, você pode querer usar um valor dentro de um [Effect:](/learn/synchronizing-with-effects)

```js {4-7,10}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: 'https://localhost:1234',
    roomId: roomId
  }

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

Isso cria um problema. [Todo valor reativo deve ser declarado como uma dependência do seu Effect.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) No entanto, se você declarar `options` como uma dependência, isso fará com que seu Effect se reconecte constantemente à sala de bate-papo:

```js {5}
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🔴 Problema: esta dependência muda a cada renderização
  // ...
```

Para resolver isso, você pode encapsular o objeto que você precisa chamar de um Effect em `useMemo`:

```js {4-9,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = useMemo(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Muda apenas quando roomId muda

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ Muda apenas quando options muda
  // ...
```

Isso garante que o objeto do `options` seja o mesmo entre as renderizações se `useMemo` retornar o objeto em cache.

No entanto, como o `useMemo` é uma otimização de desempenho, não uma garantia semântica, React pode descartar o valor em cache se [houver uma razão específica para fazer isso](#caveats). Isso também fará com que o efeito seja disparado novamente, **portanto, é ainda melhor remover a necessidade de uma dependência de função** movendo seu objeto *dentro* do Effect:

```js {5-8,13}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = { // ✅ Não precisa de useMemo ou dependências de objeto!
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    }
    
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Muda apenas quando roomId muda
  // ...
```

Agora seu código é mais simples e não precisa de `useMemo`. [Saiba mais sobre como remover dependências de Effect.](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

### Memorizando uma dependência de outro Hook {/*memoizing-a-dependency-of-another-hook*/}

Suponha que você tenha um cálculo que depende de um objeto criado diretamente no corpo do componente:

```js {2}
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 Cuidado: dependência em um objeto criado no corpo do componente
  // ...
```

Depender de um objeto como este frustra o propósito da memoização. Quando um componente é renderizado novamente, todo o código diretamente dentro do corpo do componente é executado novamente. **As linhas de código que criam o objeto `searchOptions` também serão executadas em cada renderização novamente.** Como `searchOptions` é uma dependência da sua chamada `useMemo`, e é diferente toda vez, React sabe que as dependências são diferentes e recalcula `searchItems` toda vez.

Para corrigir isso, você pode memorizar o objeto `searchOptions` *em si* antes de passá-lo como uma dependência:

```js {2-4}
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ Muda apenas quando text muda

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ Muda apenas quando allItems ou searchOptions muda
  // ...
```

No exemplo acima, se o `text` não mudou, o objeto `searchOptions` também não mudará. No entanto, uma correção ainda melhor é mover a declaração do objeto `searchOptions` *para dentro* da função de cálculo `useMemo`:

```js {3}
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ Muda apenas quando allItems ou text muda
  // ...
```

Agora seu cálculo depende diretamente do `text` (que é uma string e não pode "acidentalmente" se tornar diferente).

---

### Memorizando uma função {/*memoizing-a-function*/}

Suponha que o componente `Form` seja encapsulado em [`memo`.](/reference/react/memo) Você deseja passar uma função para ele como uma prop:

```js {2-7}
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

Assim como `{}` cria um objeto diferente, as declarações de função como `function() {}` e expressões como `() => {}` produzem uma função *diferente* a cada renderização. Por si só, criar uma nova função não é um problema. Isso não é algo a ser evitado! No entanto, se o componente `Form` é memorizado, presumivelmente você deseja pular a renderização dele novamente quando nenhuma prop tiver mudado. Uma prop que é *sempre* diferente frustraria o objetivo da memoização.

Para memorizar uma função com `useMemo`, sua função de cálculo teria que retornar outra função:

```js {2-3,8-9}
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

Isso parece desajeitado! **Memoizar funções é comum o suficiente que o React tem um Hook embutido especificamente para isso. Envolva suas funções em [`useCallback`](/reference/react/useCallback) em vez de `useMemo`** para evitar ter que escrever uma função aninhada extra:

```js {2,7}
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

Os dois exemplos acima são completamente equivalentes. O único benefício para `useCallback` é que ele permite que você evite escrever uma função aninhada extra dentro. Ele não faz mais nada. [Leia mais sobre `useCallback`.](/reference/react/useCallback)

---

## Solução de problemas {/*troubleshooting*/}

### Meu cálculo é executado duas vezes em cada renderização novamente {/*my-calculation-runs-twice-on-every-re-render*/}

No [Strict Mode](/reference/react/StrictMode), o React chamará algumas de suas funções duas vezes em vez de uma:

```js {2,5,6}
function TodoList({ todos, tab }) {
  // Esta função do componente será executada duas vezes para cada renderização.

  const visibleTodos = useMemo(() => {
    // Este cálculo será executado duas vezes se alguma das dependências mudar.
    return filterTodos(todos, tab);
  }, [todos, tab]);

  // ...
```

Isso é esperado e não deve quebrar seu código.

Este comportamento **somente para desenvolvimento** ajuda você a [manter os componentes puros.](/learn/keeping-components-pure) React usa o resultado de uma das chamadas e ignora o resultado da outra chamada. Contanto que suas funções de componente e cálculo sejam puras, isso não deve afetar sua lógica. No entanto, se eles forem acidentalmente impuros, isso o ajudará a notar e corrigir o erro.

Por exemplo, esta função de cálculo impura muta uma array que você recebeu como uma prop:

```js {2-3}
  const visibleTodos = useMemo(() => {
    // 🚩 Erro: mutando uma prop
    todos.push({ id: 'last', text: 'Faça uma caminhada!' });
    const filtered = filterTodos(todos, tab);
    return filtered;
  }, [todos, tab]);
```

React chama sua função duas vezes, então você notaria que a tarefa é adicionada duas vezes. Seu cálculo não deve alterar nenhum objeto existente, mas é aceitável alterar quaisquer objetos *novos* que você criou durante o cálculo. Por exemplo, se a função `filterTodos` sempre retorna uma array *diferente*, você pode mutar *essa* array em vez disso:

```js {3,4}
  const visibleTodos = useMemo(() => {
    const filtered = filterTodos(todos, tab);
    // ✅ Correto: mutando um objeto que você criou durante o cálculo
    filtered.push({ id: 'last', text: 'Faça uma caminhada!' });
    return filtered;
  }, [todos, tab]);
```

Leia [mantendo componentes puros](/learn/keeping-components-pure) para saber mais sobre pureza.

Além disso, verifique os guias sobre [como atualizar objetos](/learn/updating-objects-in-state) e [como atualizar arrays](/learn/updating-arrays-in-state) sem mutação.

---

### Minha chamada `useMemo` deve retornar um objeto, mas retorna indefinido {/*my-usememo-call-is-supposed-to-return-an-object-but-returns-undefined*/}

Este código não funciona:

```js {1-2,5}
  // 🔴 Você não pode retornar um objeto de uma função de seta com () => {
  const searchOptions = useMemo(() => {
    matchMode: 'whole-word',
    text: text
  }, [text]);
```

Em JavaScript, `() => {` inicia o corpo da função de seta, então a chave `{` não faz parte do seu objeto. É por isso que ele não retorna um objeto e leva a erros. Você pode corrigi-lo adicionando parênteses como `({` e `})`:

```js {1-2,5}
  // Isso funciona, mas é fácil para alguém quebrar novamente
  const searchOptions = useMemo(() => ({
    matchMode: 'whole-word',
    text: text
  }), [text]);
```

No entanto, isso ainda é confuso e muito fácil para alguém quebrar, removendo os parênteses.

Para evitar esse erro, escreva uma instrução `return` explicitamente:

```js {1-3,6-7}
  // ✅ Isso funciona e é explícito
  const searchOptions = useMemo(() => {
    return {
      matchMode: 'whole-word',
      text: text
    };
  }, [text]);
```

---

### Toda vez que meu componente renderiza, o cálculo em `useMemo` é executado novamente {/*every-time-my-component-renders-the-calculation-in-usememo-re-runs*/}

Certifique-se de ter especificado a array de dependência como um segundo argumento!

Se você esquecer a array de dependência, `useMemo` executará novamente o cálculo todas as vezes:

```js {2-3}
function TodoList({ todos, tab }) {
  // 🔴 Recalcula todas as vezes: nenhuma array de dependência
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  // ...
```

Esta é a versão corrigida passando a array de dependência como um segundo argumento:

```js {2-3}
function TodoList({ todos, tab }) {
  // ✅ Não recalcula desnecessariamente
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
```

Se isso não ajudar, então o problema é que pelo menos uma de suas dependências é diferente da renderização anterior. Você pode depurar este problema registrando manualmente suas dependências no console:

```js
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  console.log([todos, tab]);
```

Você pode então clicar com o botão direito do mouse nas arrays de diferentes renderizações novamente no console e selecionar "Armazenar como uma variável global" para ambas. Supondo que a primeira foi salva como `temp1` e a segunda foi salva como `temp2`, você pode então usar o console do navegador para verificar se cada dependência em ambas as arrays é a mesma:

```js
Object.is(temp1[0], temp2[0]); // A primeira dependência é a mesma entre as arrays?
Object.is(temp1[1], temp2[1]); // A segunda dependência é a mesma entre as arrays?
Object.is(temp1[2], temp2[2]); // ... e assim por diante para cada dependência ...
```

Quando você descobrir qual dependência quebra a memorização, encontre uma maneira de removê-la, ou [memorize também.](#memoizing-a-dependency-of-another-hook)

---

### Eu preciso chamar `useMemo` para cada item da lista em um loop, mas não é permitido {/*i-need-to-call-usememo-for-each-list-item-in-a-loop-but-its-not-allowed*/}

Suponha que o componente `Chart` esteja encapsulado em [`memo`](/reference/react/memo). Você deseja pular a renderização de cada `Chart` na lista quando o componente `ReportList` renderizar novamente. No entanto, você não pode chamar `useMemo` em um loop:

```js {5-11}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 Você não pode chamar useMemo em um loop como este:
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```

Em vez disso, extraia um componente para cada item e memorize dados para itens individuais:

```js {5,12-18}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Chame useMemo no nível superior:
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

Alternativamente, você pode remover `useMemo` e, em vez disso, encapsular o próprio `Report` em [`memo`.](/reference/react/memo) Se a prop `item` não mudar, o `Report` pulará a renderização novamente, então o `Chart` pulará a renderização novamente também:

```js {5,6,12}
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  const data = calculateReport(item);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
});