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

### Passing the initial state with `init` {/*passing-the-initial-state-with-init*/}

You can lazily initialize your state. Pass the initial state as the second argument to `useReducer`. If you need to do some computation to determine the initial state (for example, if you need to read it from local storage), you can pass an `init` function as the third argument. It will be called with the same initial `initialArg` value, and its return value will be used as the initial state.

```js
function init(initialCount) {
  return {count: initialCount};
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
```

This is useful for when the initial state needs to be computed in some way:

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment': {
      return {count: state.count + 1};
    }
    case 'decrement': {
      return {count: state.count - 1};
    }
    default: {
      throw Error('Unknown action.');
    }
  }
}

function init(initialCount) {
  return {count: initialCount};
}

export default function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      <button
        onClick={() => dispatch({ type: 'decrement' })}
      >
        -
      </button>
      <p>Count: {state.count}</p>
      <button
        onClick={() => dispatch({ type: 'increment' })}
      >
        +
      </button>
    </>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

<Recipes titleText="useReducer with init examples" titleId="examples-init">

#### Fetching data from local storage {/*fetching-data-from-local-storage*/}

In this example, the initial state is read from `localStorage`. Passing `init` allows us to separate the logic for calculating the initial state from the logic for updating the state.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'updated': {
      return action.todos;
    }
    case 'added': {
      return [...state, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return state.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return state.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

function init(initialTodos) {
  // You could also read this from localStorage
  return initialTodos;
}

export default function TodoApp() {
  const [todos, dispatch] = useReducer(reducer, [], init);

  return (
    <>
      <input
        type="text"
        placeholder="Add a todo"
        onKeyDown={e => {
          if (e.key === 'Enter') {
            e.preventDefault();
            const text = e.target.value;
            e.target.value = '';
            dispatch({
              type: 'added',
              id: crypto.randomUUID(),
              text: text,
            });
          }
        }}
      />
      <TaskList
        todos={todos}
        onChangeTodo={todo => {
          dispatch({
            type: 'changed',
            task: todo
          });
        }}
        onDeleteTodo={id => {
          dispatch({
            type: 'deleted',
            id: id
          });
        }}
      />
    </>
  );
}

function TaskList({ todos, onChangeTodo, onDeleteTodo }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={todo.text}
          onChange={e => {
            onChange({
              ...todo,
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
        {todo.text}
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
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
input {
  margin: 5px;
}

button {
  margin: 5px;
}

li {
  list-style-type: none;
}

ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Choosing between `useState` and `useReducer` {/*choosing-between-usestate-and-usereducer*/}

Both `useState` and `useReducer` let you manage state in a component.

*   `useState` is good for simple values and when the next state doesn't depend on the previous one.
*   `useReducer` is good when the next state **does** depend on the previous one, or if you have complex state update logic.

In the example above, the logic for updating the state is extracted into a `reducer` function. This makes the code easier to understand. If the state is simple, like a single number or string, you might not need `useReducer`. But when you have a more complex object, or if you have multiple ways to update your state, `useReducer` can make your components easier to maintain for a complex application.

Here's a summary table for comparing `useState` and `useReducer`:

| Feature           | `useState`                                         | `useReducer`                                                       |
| ----------------- | -------------------------------------------------- | ------------------------------------------------------------------ |
| Initial state     | Any value.                                         | Any value, can be computed with `init`.                            |
| Updating state    | Call the `set` function.                           | Call the `dispatch` function, pass an action as the argument.     |
| Next state        | The value you pass to the `set` function.          | Computed by the `reducer` function based on `state` and `action`. |
| State logic       | Inline in event handlers.                          | Extracted into a `reducer` function.                               |
| Useful when...    | Single values, state updates don't depend on the previous state. | Complex state logic, state updates depend on the previous state. |

---

## Troubleshooting {/*troubleshooting*/}

### My reducer or initializer function runs twice {/*my-reducer-or-initializer-function-runs-twice*/}

In Strict Mode, React will **call your reducer and initializer twice** in order to [help you find accidental impurities.](/reference/react/StrictMode) This is development-only behavior and does not affect production.

```js
function reducer(state, action) {
  console.log('reducer ran'); // This will log twice in development!
  // ...
}

function init(initialValue) {
  console.log('init ran'); // This will log twice in development!
  return {count: initialValue};
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, 0, init);
  // ...
```

If your reducer and initializer are pure (as they should be), this should not affect your logic. The result from one of the calls is ignored.

If you *do* see problems, make sure that your functions are pure. For example, you should not modify any objects or arrays that you read from the state, nor should you calculate or log things as side effects within the reducer or init functions.
If the reducer and initializer are pure, this should not affect your logic.

### I've dispatched an action, but logging gives me the old state value {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

Calling the `dispatch` function **doesn't update the state immediately.** This is done to improve performance. Instead, it tells React that you want to update the state. React will then process the update later, when it's ready, and re-render your component.

That means that if you try to read the state right after calling the `dispatch` function, you'll get the "old" value.

```js
function handleClick() {
  dispatch({ type: 'incremented_age' });
  console.log(state.age); // 🔴 Will log the old age
}
```

To read the updated state, read `state` when the component re-renders:

```js
function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  function handleClick() {
    dispatch({ type: 'incremented_age' });
  }

  console.log(state.age); // ✅ Will log the current age
  return (
    <button onClick={handleClick}>
      Increment age
    </button>
  );
}
```

If you need to perform a side effect *after* your state has been updated, use an [`useEffect`](/reference/react/useEffect) Hook.

### How to dispatch multiple actions in a row {/*how-to-dispatch-multiple-actions-in-a-row*/}

You can dispatch multiple actions during a single event. React will process these updates in the order they are dispatched, though it may batch the re-renders for performance. Here's an example of dispatching multiple actions in a row:

```js
function handleChange(e) {
  const type = e.target.value;
  dispatch({ type: 'SET_TYPE', payload: type });
  dispatch({ type: 'LOAD_DATA', payload: type });
}
```
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

React salva o estado inicial uma vez e o ignora nas próximas renderizações.

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

Embora o resultado de `createInitialState(username)` seja usado apenas para a renderização inicial, você ainda está chamando essa função a cada renderização. Isso pode ser um desperdício se estiver criando grandes arrays ou executando cálculos caros.

Para resolver isso, você pode **passá-lo como uma função _inicializadora_** para `useReducer` como o terceiro argumento:

```js {6}
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

Observe que você está passando `createInitialState`, que é a *própria função*, e não `createInitialState()`, que é o resultado de chamá-la. Dessa forma, o estado inicial não é recriado após a inicialização.

No exemplo acima, `createInitialState` recebe um argumento `username`. Se seu inicializador não precisar de nenhuma informação para calcular o estado inicial, você pode passar `null` como o segundo argumento para `useReducer`.

<Recipes titleText="A diferença entre passar um inicializador e passar o estado inicial diretamente" titleId="examples-initializer">

#### Passando a função inicializadora {/*passing-the-initializer-function*/}

Este exemplo passa a função inicializadora, portanto, a função `createInitialState` só é executada durante a inicialização. Ela não é executada quando o componente é renderizado novamente, como quando você digita no campo de entrada.

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

Este exemplo **não** passa a função inicializadora, então a função `createInitialState` é executada em cada renderização, como quando você digita no campo de entrada. Não há diferença observável no comportamento, mas esse código é menos eficiente.

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

## Solução de Problemas {/*troubleshooting*/}

### Eu despachei uma ação, mas o log me dá o valor de estado antigo {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

Chamar a função `dispatch` **não altera o estado no código em execução**:

```js {4,5,8}
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Solicita uma nova renderização com 43
  console.log(state.age);  // Ainda é 42!

  setTimeout(() => {
    console.log(state.age); // Também 42!
  }, 5000);
}
```

Isso ocorre porque [estados se comportam como um snapshot.](/learn/state-as-a-snapshot) Atualizar o estado solicita outra renderização com o novo valor do estado, mas não afeta a variável JavaScript `state` no seu manipulador de eventos já em execução.

Se você precisar adivinhar o próximo valor do estado, você pode calculá-lo manualmente chamando o reducer você mesmo:

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

---

### Eu despachei uma ação, mas a tela não atualiza {/*ive-dispatched-an-action-but-the-screen-doesnt-update*/}

React vai **ignorar sua atualização se o próximo estado for igual ao estado anterior,** conforme determinado por uma comparação [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Isso geralmente acontece quando você altera um objeto ou um array no estado diretamente:

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

Você mutou um objeto `state` existente e o retornou, então o React ignorou a atualização. Para corrigir isso, você precisa garantir que você está sempre [atualizando objetos no estado](/learn/updating-objects-in-state) e [atualizando arrays no estado](/learn/updating-arrays-in-state) em vez de mutá-los:

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

### Uma parte do meu estado de reducer se torna indefinida após o envio {/*a-part-of-my-reducer-state-becomes-undefined-after-dispatching*/}

Certifique-se de que cada ramificação `case` **copia todos os campos existentes** ao retornar o novo estado:

```js {5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // Não esqueça disso!
        age: state.age + 1
      };
    }
    // ...
```

Sem `...state` acima, o próximo estado retornado só conteria o campo `age` e nada mais.

---

### Meu estado inteiro do reducer se torna indefinido após o envio {/*my-entire-reducer-state-becomes-undefined-after-dispatching*/}

Se o seu estado inesperadamente se torna `undefined`, você provavelmente está esquecendo de `return` o estado em um dos casos, ou seu tipo de ação não corresponde a nenhuma das instruções `case`. Para descobrir o porquê, lance um erro fora do `switch`:

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

Você também pode usar um verificador de tipo estático como o TypeScript para detectar tais erros.

---

### Estou recebendo um erro: "Muitas re-renderizações" {/*im-getting-an-error-too-many-re-renders*/}

Você pode receber um erro que diz: `Muitas re-renderizações. React limita o número de renderizações para evitar um loop infinito.` Normalmente, isso significa que você está despachando uma ação incondicionalmente *durante a renderização*, então seu componente entra em um loop: renderizar, despachar (o que causa uma renderização), renderizar, despachar (o que causa uma renderização) e assim por diante. Muitas vezes, isso é causado por um erro na especificação de um manipulador de eventos:

```js {1-2}
// 🚩 Errado: chama o manipulador durante a renderização
return <button onClick={handleClick()}>Clique em mim</button>

// ✅ Correto: passa o manipulador de eventos
return <button onClick={handleClick}>Clique em mim</button>

// ✅ Correto: passa uma função inline
return <button onClick={(e) => handleClick(e)}>Clique em mim</button>
```

Se você não conseguir encontrar a causa desse erro, clique na seta ao lado do erro no console e examine a pilha JavaScript para encontrar a chamada específica da função `dispatch` responsável pelo erro.

---

### Minha função reducer ou inicializadora é executada duas vezes {/*my-reducer-or-initializer-function-runs-twice*/}

No [Modo Estrito](/reference/react/StrictMode), o React chamará sua função reducer e inicializadora duas vezes. Isso não deve quebrar seu código.

Este comportamento **somente de desenvolvimento** ajuda você a [manter os componentes puros.](/learn/keeping-components-pure) React usa o resultado de uma das chamadas e ignora o resultado da outra chamada. Contanto que seus componentes, inicializadores e funções reducer sejam puros, isso não deve afetar sua lógica. No entanto, se eles forem acidentalmente impuros, isso o ajudará a notar os erros.

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

Como o React chama sua função reducer duas vezes, você verá que a tarefa foi adicionada duas vezes, então saberá que há um erro. Neste exemplo, você pode corrigir o erro [substituindo o array em vez de mutá-lo](/learn/updating-arrays-in-state#adding-to-an-array):

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

Agora que essa função reducer é pura, chamá-la uma vez a mais não faz diferença no comportamento. É por isso que o React chamá-la duas vezes ajuda você a encontrar erros. **Somente os componentes, inicializadores e funções reducer precisam ser puros.** Manipuladores de eventos não precisam ser puros, então o React nunca chamará seus manipuladores de eventos duas vezes.

Leia [manter componentes puros](/learn/keeping-components-pure) para saber mais.
