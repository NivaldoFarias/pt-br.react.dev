title: Extraindo Lógica de Estado para um Reducer
---

<Intro>

Componentes com muitas atualizações de estado espalhadas por vários manipuladores de eventos podem se tornar esmagadores. Para esses casos, você pode consolidar toda a lógica de atualização de estado fora do seu componente em uma única função, chamada _reducer_.

</Intro>

<YouWillLearn>

- O que é uma função reducer
- Como refatorar `useState` para `useReducer`
- Quando usar um reducer
- Como escrever um bom reducer

</YouWillLearn>

## Consolide a lógica de estado com um reducer {/*consolidate-state-logic-with-a-reducer*/}

À medida que seus componentes crescem em complexidade, pode se tornar mais difícil ver de relance todas as diferentes maneiras pelas quais o estado de um componente é atualizado. Por exemplo, o componente `TaskApp` abaixo mantém um array de `tasks` no estado e usa três manipuladores de eventos diferentes para adicionar, remover e editar tarefas:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
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
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Add
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Save</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Edit</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Delete</button>
    </label>
  );
}
```

```css
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

Cada um dos seus manipuladores de eventos chama `setTasks` para atualizar o estado. À medida que este componente cresce, também cresce a quantidade de lógica de estado espalhada por ele. Para reduzir essa complexidade e manter toda a sua lógica em um local de fácil acesso, você pode mover essa lógica de estado para uma única função fora do seu componente, **chamada "reducer".**

Reducers são uma maneira diferente de lidar com o estado. Você pode migrar de `useState` para `useReducer` em três etapas:

1. **Mova** de definir o estado para despachar ações.
2. **Escreva** uma função reducer.
3. **Use** o reducer do seu componente.

### Etapa 1: Mova de definir o estado para despachar ações {/*step-1-move-from-setting-state-to-dispatching-actions*/}

Seus manipuladores de eventos atualmente especificam _o que fazer_ definindo o estado:

```js
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

Remova toda a lógica de definição de estado. O que resta são três manipuladores de eventos:

- `handleAddTask(text)` é chamado quando o usuário pressiona "Add".
- `handleChangeTask(task)` é chamado quando o usuário alterna uma tarefa ou pressiona "Save".
- `handleDeleteTask(taskId)` é chamado quando o usuário pressiona "Delete".

Gerenciar o estado com reducers é ligeiramente diferente de definir o estado diretamente. Em vez de dizer ao React "o que fazer" definindo o estado, você especifica "o que o usuário acabou de fazer" despachando "ações" de seus manipuladores de eventos. (A lógica de atualização de estado ficará em outro lugar!) Portanto, em vez de "definir `tasks`" por meio de um manipulador de eventos, você está despachando uma ação de "adicionar/alterar/excluir uma tarefa". Isso é mais descritivo da intenção do usuário.

```js
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
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

O objeto que você passa para `dispatch` é chamado de "ação":

```js {3-7}
function handleDeleteTask(taskId) {
  dispatch(
    // Objeto "action":
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

É um objeto JavaScript comum. Você decide o que colocar nele, mas geralmente ele deve conter a informação mínima sobre _o que aconteceu_. (Você adicionará a própria função `dispatch` em uma etapa posterior.)

<Note>

Um objeto de ação pode ter qualquer formato.

Por convenção, é comum dar a ele um `type` de string que descreve o que aconteceu e passar quaisquer informações adicionais em outros campos. O `type` é específico para um componente, então neste exemplo, `'added'` ou `'added_task'` estariam bem. Escolha um nome que diga o que aconteceu!

```js
dispatch({
  // específico do componente
  type: 'o_que_aconteceu',
  // outros campos vão aqui
});
```

</Note>

### Etapa 2: Escreva uma função reducer {/*step-2-write-a-reducer-function*/}

Uma função reducer é onde você colocará sua lógica de estado. Ela recebe dois argumentos, o estado atual e o objeto de ação, e retorna o próximo estado:

```js
function yourReducer(state, action) {
  // retorna o próximo estado para o React definir
}
```

O React definirá o estado com o que você retornar do reducer.

Para mover a lógica de definição de estado de seus manipuladores de eventos para uma função reducer neste exemplo, você irá:

1. Declare o estado atual (`tasks`) como o primeiro argumento.
2. Declare o objeto `action` como o segundo argumento.
3. Retorne o _próximo_ estado do reducer (que o React definirá como o estado).

Aqui está toda a lógica de definição de estado migrada para uma função reducer:

```js
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

Como a função reducer recebe o estado (`tasks`) como argumento, você pode **declarar fora do seu componente.** Isso diminui o nível de indentação e pode tornar seu código mais fácil de ler.

<Note>

O código acima usa instruções if/else, mas é uma convenção usar [instruções switch](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/switch) dentro de reducers. O resultado é o mesmo, mas pode ser mais fácil ler instruções switch rapidamente.

Usaremos elas em toda a documentação restante assim:

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

Recomendamos envolver cada bloco `case` com as chaves `{` e `}` para que as variáveis declaradas dentro de diferentes `case`s não entrem em conflito umas com as outras. Além disso, um `case` geralmente deve terminar com um `return`. Se você esquecer de `return`, o código "cairá" para o próximo `case`, o que pode levar a erros!

Se você ainda não está familiarizado com instruções switch, usar if/else está completamente bem.

</Note>

<DeepDive>

#### Por que os reducers são chamados assim? {/*why-are-reducers-called-this-way*/}

Embora os reducers possam "reduzir" a quantidade de código dentro do seu componente, eles são na verdade nomeados após a operação [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) que você pode realizar em arrays.

A operação `reduce()` permite que você pegue um array e "acumule" um único valor de muitos:

```
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

A função que você passa para `reduce` é conhecida como "reducer". Ela pega o _resultado até agora_ e o _item atual,_ e então retorna o _próximo resultado._ Reducers do React são um exemplo da mesma ideia: eles pegam o _estado até agora_ e a _ação_, e retornam o _próximo estado._ Dessa forma, eles acumulam ações ao longo do tempo em estado.

Você poderia até usar o método `reduce()` com um `initialState` e um array de `actions` para calcular o estado final passando sua função reducer para ele:

<Sandpack>

```js src/index.js active
import tasksReducer from './tasksReducer.js';

let initialState = [];
let actions = [
  {type: 'added', id: 1, text: 'Visit Kafka Museum'},
  {type: 'added', id: 2, text: 'Watch a puppet show'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: 'Lennon Wall pic'},
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById('output');
output.textContent = JSON.stringify(finalState, null, 2);
```

```js src/tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

```html public/index.html
<pre id="output"></pre>
```

</Sandpack>

Provavelmente você não precisará fazer isso, mas é semelhante ao que o React faz!

</DeepDive>

### Etapa 3: Use o reducer do seu componente {/*step-3-use-the-reducer-from-your-component*/}

Finalmente, você precisa conectar o `tasksReducer` ao seu componente. Importe o Hook `useReducer` do React:

```js
import { useReducer } from 'react';
```

Então você pode substituir `useState`:

```js
const [tasks, setTasks] = useState(initialTasks);
```

por `useReducer` assim:

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

O Hook `useReducer` é semelhante ao `useState`—você deve passar um estado inicial para ele e ele retorna um valor com estado e uma maneira de definir o estado (neste caso, a função dispatch). Mas é um pouco diferente.

O Hook `useReducer` recebe dois argumentos:

1. Uma função reducer
2. Um estado inicial

E retorna:

1. Um valor com estado
2. Uma função dispatch (para "despachar" ações do usuário para o reducer)

Agora está totalmente conectado! Aqui, o reducer é declarado no final do arquivo do componente:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {