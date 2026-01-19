---
title: useReducer
---

<Intro>

`useReducer` é um Hook do React que permite adicionar um [reducer](/learn/extracting-state-logic-into-a-reducer) ao seu componente.

```js
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `useReducer(reducer, initialArg, init?)` {/*usereducer*/}

Chame `useReducer` no nível superior do seu componente para gerenciar seu estado com um [reducer.](/learn/extracting-state-logic-into-a-reducer)

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reducer`: A função reducer que especifica como o estado é atualizado. Ela deve ser pura, deve receber o estado e a ação como argumentos e deve retornar o próximo estado. Estado e ação podem ser de quaisquer tipos.
* `initialArg`: O valor a partir do qual o estado inicial é calculado. Pode ser um valor de qualquer tipo. Como o estado inicial é calculado a partir dele depende do próximo argumento `init`.
* **opcional** `init`: A função inicializadora que deve retornar o estado inicial. Se não for especificada, o estado inicial é definido como `initialArg`. Caso contrário, o estado inicial é definido como o resultado da chamada de `init(initialArg)`.

#### Retorna {/*returns*/}

`useReducer` retorna um array com exatamente dois valores:

1. O estado atual. Durante a primeira renderização, ele é definido como `init(initialArg)` ou `initialArg` (se não houver `init`).
2. A [`dispatch` function](#dispatch) que permite atualizar o estado para um valor diferente e acionar uma nova renderização.

#### Ressalvas {/*caveats*/}

* `useReducer` é um Hook, portanto, você só pode chamá-lo **no nível superior do seu componente** ou em seus próprios Hooks. Você não pode chamá-lo dentro de loops ou condições. Se precisar disso, extraia um novo componente e mova o estado para ele.
* A função `dispatch` tem uma identidade estável, então você frequentemente a verá omitida das dependências do Effect, mas incluí-la não fará com que o Effect seja acionado. Se o linter permitir que você omita uma dependência sem erros, é seguro fazê-lo. [Saiba mais sobre como remover dependências de Effect.](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)
* Em Strict Mode, o React **chamará seu reducer e inicializador duas vezes** para [ajudá-lo a encontrar impurezas acidentais.](#my-reducer-or-initializer-function-runs-twice) Este é um comportamento apenas de desenvolvimento e não afeta a produção. Se seu reducer e inicializador forem puros (como deveriam ser), isso não deve afetar sua lógica. O resultado de uma das chamadas é ignorado.

---

### `dispatch` function {/*dispatch*/}

A função `dispatch` retornada por `useReducer` permite que você atualize o estado para um valor diferente e acione uma nova renderização. Você precisa passar a ação como o único argumento para a função `dispatch`:

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

O React definirá o próximo estado como o resultado da chamada da função `reducer` que você forneceu com o `state` atual e a ação que você passou para `dispatch`.

#### Parâmetros {/*dispatch-parameters*/}

* `action`: A ação realizada pelo usuário. Pode ser um valor de qualquer tipo. Por convenção, uma ação é geralmente um objeto com uma propriedade `type` que a identifica e, opcionalmente, outras propriedades com informações adicionais.

#### Retorna {/*dispatch-returns*/}

Funções `dispatch` não têm valor de retorno.

#### Ressalvas {/*setstate-caveats*/}

* A função `dispatch` **apenas atualiza a variável de estado para a *próxima* renderização**. Se você ler a variável de estado após chamar a função `dispatch`, [você ainda obterá o valor antigo](#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value) que estava na tela antes da sua chamada.

* Se o novo valor fornecido for idêntico ao `state` atual, conforme determinado por uma comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is), o React **pulará a re-renderização do componente e seus filhos.** Esta é uma otimização. O React ainda pode precisar chamar seu componente antes de ignorar o resultado, mas isso não deve afetar seu código.

* O React [agrupa atualizações de estado.](/learn/queueing-a-series-of-state-updates) Ele atualiza a tela **após todos os manipuladores de eventos terem sido executados** e terem chamado suas funções `set`. Isso evita múltiplas re-renderizações durante um único evento. No caso raro em que você precise forçar o React a atualizar a tela mais cedo, por exemplo, para acessar o DOM, você pode usar [`flushSync`.](/reference/react-dom/flushSync)

---

## Uso {/*usage*/}

### Adicionando um reducer a um componente {/*adding-a-reducer-to-a-component*/}

Chame `useReducer` no nível superior do seu componente para gerenciar o estado com um [reducer.](/learn/extracting-state-logic-into-a-reducer)

```js [[1, 8, "state"], [2, 8, "dispatch"], [4, 8, "reducer"], [3, 8, "{ age: 42 }"]]
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer` retorna um array com exatamente dois itens:

1. O <CodeStep step={1}>estado atual</CodeStep> desta variável de estado, inicialmente definido como o <CodeStep step={3}>estado inicial</CodeStep> que você forneceu.
2. A <CodeStep step={2}>função `dispatch`</CodeStep> que permite alterá-lo em resposta a uma interação.

Para atualizar o que está na tela, chame <CodeStep step={2}>`dispatch`</CodeStep> com um objeto representando o que o usuário fez, chamado de *ação*:

```js [[2, 2, "dispatch"]]
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

O React passará o estado atual e a ação para sua <CodeStep step={4}>função reducer</CodeStep>. Seu reducer calculará e retornará o próximo estado. O React armazenará esse próximo estado, renderizará seu componente com ele e atualizará a interface do usuário.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

`useReducer` é muito semelhante a [`useState`](/reference/react/useState), mas permite que você mova a lógica de atualização de estado dos manipuladores de eventos para uma única função fora do seu componente. Leia mais sobre [escolher entre `useState` e `useReducer`.](/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer)

---

### Escrevendo a função reducer {/*writing-the-reducer-function*/}

Uma função reducer é declarada assim:

```js
function reducer(state, action) {
  // ...
}
```

Em seguida, você precisa preencher o código que calculará e retornará o próximo estado. Por convenção, é comum escrevê-la como uma [`switch` statement.](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/switch) Para cada `case` no `switch`, calcule e retorne um próximo estado.

```js {4-7,10-13}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

As actions podem ter qualquer formato. Por convenção, é comum passar objetos com uma propriedade `type` que identifica a action. Ela deve incluir as informações mínimas necessárias para que o reducer possa computar o próximo estado.

```js {5,9-12}
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

Os nomes dos tipos de action são locais ao seu componente. [Cada action descreve uma única interação, mesmo que isso resulte em múltiplas alterações nos dados.](/learn/extracting-state-logic-into-a-reducer#writing-reducers-well) O formato do estado é arbitrário, mas geralmente será um objeto ou um array.

Leia [extraindo a lógica de estado para um reducer](/learn/extracting-state-logic-into-a-reducer) para saber mais.

<Pitfall>

O estado é somente leitura. Não modifique nenhum objeto ou array no estado:

```js {4,5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Não modifique um objeto no estado assim:
      state.age = state.age + 1;
      return state;
    }
```

Em vez disso, sempre retorne novos objetos do seu reducer:

```js {4-8}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ Em vez disso, retorne um novo objeto
      return {
        ...state,
        age: state.age + 1
      };
    }
```

Leia [atualizando objetos no estado](/learn/updating-objects-in-state) e [atualizando arrays no estado](/learn/updating-arrays-in-state) para saber mais.

</Pitfall>

<Recipes titleText="Exemplos básicos de useReducer" titleId="examples-basic">

#### Formulário (objeto) {/*form-object*/}

Neste exemplo, o reducer gerencia um objeto de estado com dois campos: `name` e `age`.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

const initialState = { name: 'Taylor', age: 42 };

export default function Form() {
  const [state, dispatch] = useReducer(reducer, initialState);

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    }); 
  }

  return (
    <>
      <input
        value={state.name}
        onChange={handleInputChange}
      />
      <button onClick={handleButtonClick}>
        Increment age
      </button>
      <p>Hello, {state.name}. You are {state.age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

#### Lista de tarefas (array) {/*todo-list-array*/}

Neste exemplo, o reducer gerencia um array de tarefas. O array precisa ser atualizado [sem mutação.](/learn/updating-arrays-in-state)

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution />

#### Escrevendo lógica de atualização concisa com Immer {/*writing-concise-update-logic-with-immer*/}

Se atualizar arrays e objetos sem mutação parecer tedioso, você pode usar uma biblioteca como [Immer](https://github.com/immerjs/use-immer#useimmerreducer) para reduzir o código repetitivo. Immer permite que você escreva código conciso como se estivesse mutando objetos, mas nos bastidores ele realiza atualizações imutáveis:

<Sandpack>

```js src/App.js
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Evitando recriar o estado inicial {/*avoiding-recreating-the-initial-state*/}

O React salva o estado inicial uma vez e o ignora nas renderizações subsequentes.

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

Embora o resultado de `createInitialState(username)` seja usado apenas para a renderização inicial, você ainda está chamando essa função em cada renderização. Isso pode ser ineficiente se estiver criando grandes arrays ou realizando cálculos caros.

Para resolver isso, você pode **passá-la como uma função _inicializadora_** para `useReducer` como terceiro argumento:

```js {6}
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

Observe que você está passando `createInitialState`, que é a *própria função*, e não `createInitialState()`, que é o resultado de chamá-la. Dessa forma, o estado inicial não é recriado após a inicialização.

No exemplo acima, `createInitialState` recebe um argumento `username`. Se o seu inicializador não precisar de nenhuma informação para calcular o estado inicial, você pode passar `null` como segundo argumento para `useReducer`.

<Recipes titleText="A diferença entre passar um inicializador e passar o estado inicial diretamente" titleId="examples-initializer">

#### Passando a função inicializadora {/*passing-the-initializer-function*/}

Este exemplo passa a função inicializadora, então a função `createInitialState` é executada apenas durante a inicialização. Ela não é executada quando o componente é re-renderizado, como ao digitar no campo de entrada.

<Sandpack>

```js src/App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js src/TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    username,
    createInitialState
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>Add</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

#### Passando o estado inicial diretamente {/*passing-the-initial-state-directly*/}

Este exemplo **não** passa a função inicializadora, então a função `createInitialState` é executada em cada renderização, como ao digitar no campo de entrada. Não há diferença observável no comportamento, mas este código é menos eficiente.

<Sandpack>

```js src/App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js src/TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    createInitialState(username)
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>Add</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

## Solução de Problemas {/*troubleshooting*/}

### Despachei uma ação, mas o log me mostra o valor do estado antigo {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

Chamar a função `dispatch` **não altera o estado no código em execução**:

```js {4,5,8}
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Solicita uma nova renderização com 43
  console.log(state.age);  // Ainda 42!

  setTimeout(() => {
    console.log(state.age); // Também 42!
  }, 5000);
}
```

Isso ocorre porque [o estado se comporta como um snapshot.](/learn/state-as-a-snapshot) Atualizar o estado solicita outra renderização com o novo valor do estado, mas não afeta a variável JavaScript `state` em seu manipulador de eventos já em execução.

Se você precisar adivinhar o próximo valor do estado, poderá calculá-lo manualmente chamando o reducer:

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

---

### Despachei uma ação, mas a tela não atualiza {/*ive-dispatched-an-action-but-the-screen-doesnt-update*/}

O React **ignora sua atualização se o próximo estado for igual ao estado anterior**, conforme determinado por uma comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Isso geralmente acontece quando você altera um objeto ou um array no estado diretamente:

```js {4-5,9-10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Errado: mutando o objeto existente
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 Errado: mutando o objeto existente
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

Você mutou um objeto `state` existente e o retornou, então o React ignorou a atualização. Para corrigir isso, você precisa garantir que está sempre [atualizando objetos no estado](/learn/updating-objects-in-state) e [atualizando arrays no estado](/learn/updating-arrays-in-state) em vez de mutá-los:

```js {4-8,11-15}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ Correto: criando um novo objeto
      return {
        ...state,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      // ✅ Correto: criando um novo objeto
      return {
        ...state,
        name: action.nextName
      };
    }
    // ...
  }
}
```

---

### Uma parte do estado do meu reducer se torna `undefined` após o dispatch {/*a-part-of-my-reducer-state-becomes-undefined-after-dispatching*/}

Certifique-se de que cada branch `case` **copie todos os campos existentes** ao retornar o novo estado:

```js {5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // Não se esqueça disso!
        age: state.age + 1
      };
    }
    // ...
```

Sem o `...state` acima, o próximo estado retornado conteria apenas o campo `age` e nada mais.

---

### Todo o estado do meu reducer se torna `undefined` após o dispatch {/*my-entire-reducer-state-becomes-undefined-after-dispatching*/}

Se o seu estado inesperadamente se tornar `undefined`, você provavelmente está esquecendo de `return` o estado em um dos casos, ou o tipo da sua ação não corresponde a nenhuma das instruções `case`. Para descobrir o porquê, lance um erro fora do `switch`:

```js {10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ...
    }
    case 'edited_name': {
      // ...
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

Você também pode usar um verificador de tipo estático como TypeScript para capturar tais erros.

---

### Estou recebendo um erro: "Too many re-renders" {/*im-getting-an-error-too-many-re-renders*/}

Você pode receber um erro que diz: `Too many re-renders. React limits the number of renders to prevent an infinite loop.` Geralmente, isso significa que você está despachando incondicionalmente uma ação *durante a renderização*, então seu componente entra em um loop: renderizar, despachar (o que causa uma renderização), renderizar, despachar (o que causa uma renderização) e assim por diante. Muito frequentemente, isso é causado por um erro na especificação de um manipulador de eventos:

```js {1-2}
// 🚩 Errado: chama o manipulador durante a renderização
return <button onClick={handleClick()}>Click me</button>

// ✅ Correto: passa o manipulador de eventos
return <button onClick={handleClick}>Click me</button>

// ✅ Correto: passa uma função inline
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

Se você não conseguir encontrar a causa desse erro, clique na seta ao lado do erro no console e examine a pilha JavaScript para encontrar a chamada de função `dispatch` específica responsável pelo erro.

---

### Meu reducer ou função inicializadora é executado duas vezes {/*my-reducer-or-initializer-function-runs-twice*/}

No [Modo Estrito](/reference/react/StrictMode), o React chamará suas funções reducer e inicializadora duas vezes. Isso não deve quebrar seu código.

Esse comportamento, **apenas em desenvolvimento**, ajuda você a [manter os componentes puros.](/learn/keeping-components-pure) O React usa o resultado de uma das chamadas e ignora o resultado da outra. Contanto que suas funções de componente, inicializadora e reducer sejam puras, isso não afetará sua lógica. No entanto, se elas forem acidentalmente impuras, isso o ajudará a notar os erros.

Por exemplo, esta função reducer impura muta um array no estado:

```js {4-6}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // 🚩 Erro: mutando o estado
      state.todos.push({ id: nextId++, text: action.text });
      return state;
    }
    // ...
  }
}
```

Como o React chama sua função reducer duas vezes, você verá que o todo foi adicionado duas vezes, então saberá que há um erro. Neste exemplo, você pode corrigir o erro [substituindo o array em vez de mutá-lo](/learn/updating-arrays-in-state#adding-to-an-array):

```js {4-11}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // ✅ Correto: substituindo com novo estado
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: nextId++, text: action.text }
        ]
      };
    }
    // ...
  }
}
```

Agora que esta função reducer é pura, chamá-la uma vez extra não faz diferença no comportamento. É por isso que o React chamá-la duas vezes ajuda você a encontrar erros. **Apenas as funções de componente, inicializadora e reducer precisam ser puras.** Manipuladores de eventos não precisam ser puros, então o React nunca chamará seus manipuladores de eventos duas vezes.

Leia [mantendo componentes puros](/learn/keeping-components-pure) para saber mais.