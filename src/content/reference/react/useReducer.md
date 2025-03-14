```js
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
      const index = draft.findIndex(t => t.id === action.task.id);
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

</Recipes>

---

### Initializing state lazily {/*initializing-state-lazily*/}

You can provide a third, optional, `init` function to `useReducer`. This lets you calculate the initial state lazily.

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

React will call your `init(initialArg)` function and use its return value as the initial state.

```js {3,4}
function init(initialCount) {
  return {count: initialCount};
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
```

This can be useful if the initial state calculation is expensive:

*   It lets you defer the calculation until after the component has rendered.
*   It lets you avoid recalculating the initial state on every re-render.

In Strict Mode, React will call your `init` function **twice in development** to [help you find accidental impurities.](#my-reducer-or-initializer-function-runs-twice)

<Recipes titleText="Examples of using init" titleId="examples-init">

#### Calculating initial state from props {/*calculating-initial-state-from-props*/}

In this example, the initial state is derived from a prop.  The `init` function ensures that the expensive `expensiveFunction` is only called once.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unexpected action');
  }
}

function expensiveFunction(a) {
  console.log('Expensive calculation.  Only happens once.');
  return { count: a };
}

function init(initialCount) {
  return expensiveFunction(initialCount);
}

export default function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### Calculating the initial state from localStorage {/*calculating-the-initial-state-from-localstorage*/}

In this example, the initial state is loaded from `localStorage`.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'set': {
      return action.payload;
    }
    default:
      throw new Error('Unknown action');
  }
}

function init() {
  return {
    todos: JSON.parse(localStorage.getItem('todos')) || []
  };
}

export default function App() {
  const [state, dispatch] = useReducer(reducer, [], init);

  useState(() => {
    localStorage.setItem('todos', JSON.stringify(state.todos));
  });

  function handleAdd(text) {
    dispatch({
      type: 'set',
      payload: {
        todos: [
          ...state.todos,
          {
            text: text,
          },
        ],
      },
    });
  }


  return (
    <>
      <input onBlur={e => {
        handleAdd(e.target.value);
      }} />
      <ul>
        {state.todos.map((todo) => (
          <li>{todo.text}</li>
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

### Choosing between `useState` and `useReducer` {/*choosing-between-usestate-and-usereducer*/}

Both `useState` and `useReducer` let you manage state in your components.

*   **Choose `useState`** if:
    *   The state is a primitive (number, string, boolean) or an object.
    *   The state update logic isn't complex.
    *   The next state doesn't depend on the previous state.
*   **Choose `useReducer`** if:
    *   The state is an object or an array, and you would like to avoid writing code that updates it manually.
    *   The update logic is complex and involves multiple sub-values or depends on the previous state.
    *   You want to optimize re-renders because you can pass `dispatch` down instead of callbacks.
    *   You find it worthwhile to test the update logic outside of a component.

In general:

*   If you're unsure, start with `useState`.
*   If you find yourself struggling to update the state, switch to `useReducer`.

---

### My reducer or initializer function runs twice {/*my-reducer-or-initializer-function-runs-twice*/}

In Strict Mode, React will **call your initializer and reducer functions twice** in development to help you find accidental impurities.

```js
function reducer(state, action) {
  // This runs twice, so avoid side effects here.
  console.log('Reducer ran');
  // ...
}

function init(initialArg) {
  // This runs twice, so avoid side effects here.
  console.log('Init ran');
  return { count: initialArg };
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, 0, init);
```

This is a development-only behavior and does not affect production.  However, it means you should not put any side effects (like logging, modifying the DOM, or running asynchronous code) into your initializer or reducer function. This is because if they run more than once, that could break your application. If you need to run code as a side effect, do it inside an event handler or in an Effect.

React ignores the result of one of the calls. As long as your initializer and reducer functions are pure, this should not affect your logic.
```js
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
      throw Error('Ação desconhecida: ' + action.type);
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
  { id: 0, text: 'Visitar o Museu Kafka', done: true },
  { id: 1, text: 'Assistir a um show de marionetes', done: false },
  { id: 2, text: 'Foto no Lennon Wall', done: false },
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Adicionar tarefa"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Adicionar</button>
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
          Salvar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
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
        Deletar
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

`React` salva o estado inicial uma vez e o ignora nas próximas renderizações.

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

Embora o resultado de `createInitialState(username)` seja usado apenas para a renderização inicial, você ainda está chamando esta função em cada renderização. Isso pode ser um desperdício se estiver criando grandes arrays ou realizando cálculos caros.

Para resolver isso, você pode **passá-lo como uma função _inicializadora_** para `useReducer` como o terceiro argumento em vez disso:

```js {6}
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

Observe que você está passando `createInitialState`, que é *a própria função*, e não `createInitialState()`, que é o resultado de chamá-la. Dessa forma, o estado inicial não é recriado após a inicialização.

No exemplo acima, `createInitialState` recebe um argumento `username`. Se seu inicializador não precisar de nenhuma informação para calcular o estado inicial, você poderá passar `null` como o segundo argumento para `useReducer`.

<Recipes titleText="A diferença entre passar um inicializador e passar o estado inicial diretamente" titleId="examples-initializer">

#### Passando a função inicializadora {/*passing-the-initializer-function*/}

Este exemplo passa a função inicializadora, para que a função `createInitialState` seja executada apenas durante a inicialização. Ela não é executada quando o componente é renderizado novamente, como quando você digita na entrada.

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
  throw Error('Ação desconhecida: ' + action.type);
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
      }}>Adicionar</button>
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

Este exemplo **não** passa a função inicializadora, então a função `createInitialState` é executada em cada renderização, como quando você digita na entrada. Não há diferença observável no comportamento, mas este código é menos eficiente.

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
  throw Error('Ação desconhecida: ' + action.type);
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
      }}>Adicionar</button>
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

## Solução de problemas {/*troubleshooting*/}

### Eu despachei uma ação, mas o log me dá o valor do estado antigo {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

Chamar a função `dispatch` **não altera o estado no código em execução**:

```js {4,5,8}
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Solicitar uma nova renderização com 43
  console.log(state.age);  // Ainda 42!

  setTimeout(() => {
    console.log(state.age); // Também 42!
  }, 5000);
}
```

Isso ocorre porque [estados se comportam como um snapshot.](/learn/state-as-a-snapshot) Atualizar o estado solicita outra renderização com o novo valor de estado, mas não afeta a variável JavaScript `state` em seu manipulador de eventos já em execução.

Se você precisar adivinhar o próximo valor do estado, pode calculá-lo manualmente chamando o redutor você mesmo:

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

---

### Eu despachei uma ação, mas a tela não atualiza {/*ive-dispatched-an-action-but-the-screen-doesnt-update*/}

React **ignorará sua atualização se o próximo estado for igual ao estado anterior,** conforme determinado por uma comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Isso geralmente acontece quando você altera um objeto ou array no estado diretamente:

```js {4-5,9-10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Errado: mutando objeto existente
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 Errado: mutando objeto existente
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

### Uma parte do meu estado do redutor se torna indefinida após o despacho {/*a-part-of-my-reducer-state-becomes-undefined-after-dispatching*/}

Certifique-se de que cada ramificação `case** copie todos os campos existentes** ao retornar o novo estado:

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

Sem `...state` acima, o próximo estado retornado conteria apenas o campo `age` e nada mais.

---

### Meu estado do redutor inteiro se torna indefinido após o envio {/*my-entire-reducer-state-becomes-undefined-after-dispatching*/}

Se seu estado inesperadamente se tornar `undefined`, você provavelmente está esquecendo de `return` o estado em um dos casos ou seu tipo de ação não corresponde a nenhuma das declarações `case`. Para descobrir o porquê, lance um erro fora do `switch`:

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
  throw Error('Ação desconhecida: ' + action.type);
}
```

Você também pode usar um verificador de tipo estático como o TypeScript para detectar esses erros.

---

### Estou recebendo um erro: "Muitas re-renderizações" {/*im-getting-an-error-too-many-re-renders*/}

Você pode obter um erro que diz: `Muitas re-renderizações. React limita o número de renderizações para evitar um loop infinito.` Normalmente, isso significa que você está despachando condicionalmente uma ação *durante a renderização*, então seu componente entra em um loop: renderizar, despachar (o que causa uma renderização), renderizar, despachar (o que causa uma renderização) e assim por diante. Muitas vezes, isso é causado por um erro na especificação de um manipulador de eventos:

```js {1-2}
// 🚩 Errado: chama o manipulador durante a renderização
return <button onClick={handleClick()}>Clique em mim</button>

// ✅ Correto: passa o manipulador de eventos
return <button onClick={handleClick}>Clique em mim</button>

// ✅ Correto: passa uma função embutida
return <button onClick={(e) => handleClick(e)}>Clique em mim</button>
```

Se você não conseguir encontrar a causa desse erro, clique na seta ao lado do erro no console e examine a pilha JavaScript para encontrar a chamada de função `dispatch` específica responsável pelo erro.

---

### Meu redutor ou função inicializadora é executado duas vezes {/*my-reducer-or-initializer-function-runs-twice*/}

No [Modo Strict](/reference/react/StrictMode), o React chamará seu redutor e funções de inicialização duas vezes. Isso não deve quebrar seu código.

Este comportamento **somente para desenvolvimento** ajuda você a [manter os componentes puros.](/learn/keeping-components-pure) React usa o resultado de uma das chamadas e ignora o resultado da outra chamada. Contanto que seu componente, inicializador e funções de redutor sejam puros, isso não deve afetar sua lógica. No entanto, se eles forem acidentalmente impuros, isso o ajudará a notar os erros.

Por exemplo, esta função de redutor impura muta um array no estado:

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

Como o React chama sua função de redutor duas vezes, você verá que o todo foi adicionado duas vezes, então você saberá que há um erro. Neste exemplo, você pode corrigir o erro [substituindo o array em vez de mutá-lo](/learn/updating-arrays-in-state#adding-to-an-array):

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

Agora que esta função redutora é pura, chamá-la uma vez extra não faz diferença no comportamento. É por isso que o React chamá-lo duas vezes ajuda você a encontrar erros. **Apenas os componentes, inicializadores e funções de redutor precisam ser puros.** Os manipuladores de eventos não precisam ser puros, então o React nunca chamará seus manipuladores de eventos duas vezes.

Leia [manter os componentes puros](/learn/keeping-components-pure) para saber mais.
