html
PART 2 OF 2:

PART 2 OF 2:

---
title: Extraindo a Lógica do Estado para um Redutor
---

<Intro>

Componentes com muitas atualizações de estado espalhadas por muitos manipuladores de eventos podem se tornar muito difíceis. Nesses casos, você pode consolidar toda a lógica de atualização do estado fora do seu componente em uma única função, chamada de _redutor._

</Intro>

<YouWillLearn>

- O que é uma função redutora
- Como refatorar `useState` para `useReducer`
- Quando usar um redutor
- Como escrever um bem

</YouWillLearn>

## Consolidar a lógica do estado com um redutor {/*consolidate-state-logic-with-a-reducer*/}

À medida que seus componentes crescem em complexidade, pode se tornar mais difícil ver rapidamente todas as diferentes maneiras pelas quais o estado de um componente é atualizado. Por exemplo, o componente `TaskApp` abaixo contém um array de `tasks` no estado e usa três manipuladores de eventos diferentes para adicionar, remover e editar tarefas:

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

Cada um de seus manipuladores de eventos chama `setTasks` para atualizar o estado. À medida que este componente cresce, também cresce a quantidade de lógica de estado espalhada por ele. Para reduzir essa complexidade e manter toda a sua lógica em um só lugar de fácil acesso, você pode mover essa lógica de estado para uma única função fora do seu componente, **chamada de "redutor".**

Redutores são uma maneira diferente de lidar com o estado. Você pode migrar de `useState` para `useReducer` em três etapas:

1.  **Mover** de definir o estado para despachar ações.
2.  **Escrever** uma função redutora.
3.  **Usar** o redutor do seu componente.

### Etapa 1: Mover de definir o estado para despachar ações {/*step-1-move-from-setting-state-to-dispatching-actions*/}

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

-   `handleAddTask(text)` é chamado quando o usuário pressiona "Add".
-   `handleChangeTask(task)` é chamado quando o usuário alterna uma tarefa ou pressiona "Salvar".
-   `handleDeleteTask(taskId)` é chamado quando o usuário pressiona "Deletar".

Gerenciar o estado com redutores é ligeiramente diferente de definir o estado diretamente. Em vez de dizer ao React "o que fazer" definindo o estado, você especifica "o que o usuário acabou de fazer" despachando "ações" de seus manipuladores de eventos. (A lógica de atualização do estado viverá em outro lugar!) Então, em vez de "definir `tasks`" por meio de um manipulador de eventos, você está despachando uma ação de "adicionado/alterado/excluído uma tarefa". Isso é mais descritivo da intenção do usuário.

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
    // "action" object:
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

É um objeto JavaScript regular. Você decide o que colocar nele, mas, em geral, ele deve conter as informações mínimas sobre _o que aconteceu_. (Você adicionará a própria função `dispatch` em uma etapa posterior.)

<Note>

Um objeto de ação pode ter qualquer formato.

Por convenção, é comum dar a ele um `type` string que descreve o que aconteceu e passar quaisquer informações adicionais em outros campos. O `type` é específico para um componente, então, neste exemplo, tanto `'added'` quanto `'added_task'` seriam adequados. Escolha um nome que diga o que aconteceu!

```js
dispatch({
  // specific to component
  type: 'what_happened',
  // other fields go here
});
```

</Note>

### Etapa 2: Escrever uma função redutora {/*step-2-write-a-reducer-function*/}

Uma função redutora é onde você colocará sua lógica de estado. Ela recebe dois argumentos, o estado atual e o objeto de ação, e retorna o próximo estado:

```js
function yourReducer(state, action) {
  // return next state for React to set
}
```

React definirá o estado para o que você retornar do redutor.

Para mover sua lógica de definição de estado de seus manipuladores de eventos para uma função redutora neste exemplo, você:

1.  Declare o estado atual (`tasks`) como o primeiro argumento.
2.  Declare o objeto `action` como o segundo argumento.
3.  Retorne o _próximo_ estado do redutor (para o qual o React definirá o estado).

Aqui está toda a lógica de definição de estado migrada para uma função redutora:

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

Como a função redutora recebe o estado (`tasks`) como argumento, você pode **declará-la fora do seu componente.** Isso diminui o nível de indentação e pode tornar seu código mais fácil de ler.

<Note>

O código acima usa instruções if/else, mas é uma convenção usar instruções [switch](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/switch) dentro dos redutores. O resultado é o mesmo, mas pode ser mais fácil ler as instruções switch de relance.

Estaremos usando-as ao longo do restante desta documentação assim:

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

Recomendamos envolver cada bloco `case` nas chaves `{` e `}` para que as variáveis declaradas dentro de diferentes `case`s não entrem em conflito entre si. Além disso, um `case` geralmente deve terminar com um `return`. Se você esquecer de `return`, o código "cairá" para o próximo `case`, o que pode levar a erros!

Se você ainda não se sente confortável com as instruções switch, usar if/else é perfeitamente aceitável.

</Note>

<DeepDive>

#### Por que os redutores são chamados assim? {/*why-are-reducers-called-this-way*/}

Embora os redutores possam "reduzir" a quantidade de código dentro do seu componente, eles são na verdade nomeados em homenagem à operação [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) que você pode realizar em arrays.

A operação `reduce()` permite que você pegue um array e "acumule" um único valor a partir de muitos:

```
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

A função que você passa para `reduce` é conhecida como um "redutor". Ele recebe o _resultado até agora_ e o _item atual_ e, em seguida, retorna o _próximo resultado_. Os redutores React são um exemplo da mesma ideia: eles recebem o _estado até agora_ e a _ação_ e retornam o _próximo estado_. Dessa forma, eles acumulam ações ao longo do tempo no estado.

Você pode até usar o método `reduce()` com um `initialState` e um array de `actions` para calcular o estado final passando sua função redutora para ele:

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

Você provavelmente não precisará fazer isso sozinho, mas isso é semelhante ao que o React faz!

</DeepDive>

### Etapa 3: Usar o redutor do seu componente {/*step-3-use-the-reducer-from-your-component*/}

Finalmente, você precisa conectar o `tasksReducer` ao seu componente. Importe o Hook `useReducer` do React:

```js
import { useReducer } from 'react';
```

Então, você pode substituir `useState`:

```js
const [tasks, setTasks] = useState(initialTasks);
```

com `useReducer` assim:

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

O Hook `useReducer` é semelhante a `useState` — você deve passar a ele um estado inicial e ele retorna um valor com estado e uma maneira de definir o estado (neste caso, a função dispatch). Mas é um pouco diferente.

O Hook `useReducer` recebe dois argumentos:

1.  Uma função redutora
2.  Um estado inicial

E ele retorna:

1.  Um valor com estado
2.  Uma função dispatch (para "despachar" ações do usuário para o redutor)

Agora está totalmente conectado! Aqui, o redutor é declarado na parte inferior do arquivo do componente:

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
```
```javascript
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
        <button onClick={() => setIsEditing(false)}>Salvar</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Editar</button>
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
      <button onClick={() => onDelete(task.id)}>Apagar</button>
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

Se quiser, você pode até mesmo mover o reducer para um arquivo diferente:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

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
      <h1>Roteiro de Praga</h1>
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
  {id: 0, text: 'Visitar o Museu Kafka', done: true},
  {id: 1, text: 'Assistir a um espetáculo de fantoches', done: false},
  {id: 2, text: 'Foto no Muro de Lennon', done: false},
];
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
      throw Error('Ação desconhecida: ' + action.type);
    }
  }
}
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Adicionar tarefa"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Adicionar
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
        <button onClick={() => setIsEditing(false)}>Salvar</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Editar</button>
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
      <button onClick={() => onDelete(task.id)}>Apagar</button>
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

A lógica do componente pode ser mais fácil de ler quando você separa as preocupações dessa forma. Agora, os manipuladores de eventos só especificam _o que aconteceu_ despachando ações, e a função do reducer determina _como o estado é atualizado_ em resposta a elas.

## Comparando `useState` e `useReducer` {/*comparing-usestate-and-usereducer*/}

Reducers não são isentos de desvantagens! Aqui estão algumas formas de compará-los:

- **Tamanho do código:** Geralmente, com `useState` você precisa escrever menos código antecipadamente. Com `useReducer`, você precisa escrever tanto uma função de reducer _quanto_ despachar ações. No entanto, `useReducer` pode ajudar a reduzir o código se muitos manipuladores de eventos modificarem o estado de forma semelhante.
- **Legibilidade:** `useState` é muito fácil de ler quando as atualizações de estado são simples. Quando elas se tornam mais complexas, podem inflar o código do seu componente e dificultar a análise. Nesse caso, `useReducer` permite que você separe de forma limpa o _como_ da lógica de atualização do _o que aconteceu_ dos manipuladores de eventos.
- **Depuração:** Quando você tem um erro com `useState`, pode ser difícil dizer _onde_ o estado foi definido incorretamente e _por quê_. Com `useReducer`, você pode adicionar um `console.log` em seu reducer para ver cada atualização de estado e _por que_ ela aconteceu (devido a qual `action`). Se cada `action` estiver correta, você saberá que o erro está na própria lógica do reducer. No entanto, você precisa percorrer mais código do que com `useState`.
- **Testes:** Um reducer é uma função pura que não depende do seu componente. Isso significa que você pode exportá-lo e testá-lo separadamente em isolamento. Embora, de modo geral, seja melhor testar os componentes em um ambiente mais realista, para uma lógica complexa de atualização de estado, pode ser útil afirmar que seu reducer retorna um estado específico para um estado inicial e ação específicos.
- **Preferência pessoal:** Algumas pessoas gostam de reducers, outras não. Tudo bem. É uma questão de preferência. Você sempre pode converter entre `useState` e `useReducer` de um lado para o outro: eles são equivalentes!

Recomendamos o uso de um reducer se você frequentemente encontrar erros devido a atualizações de estado incorretas em algum componente e quiser introduzir mais estrutura em seu código. Você não precisa usar reducers para tudo: sinta-se à vontade para misturar e combinar! Você pode até mesmo usar `useState` e `useReducer` no mesmo componente.

## Escrevendo reducers bem {/*writing-reducers-well*/}

Tenha estas duas dicas em mente ao escrever reducers:

- **Reducers devem ser puros.** Semelhante às [funções de atualização de estado](/learn/queueing-a-series-of-state-updates), os reducers são executados durante a renderização! (As ações são enfileiradas até a próxima renderização.) Isso significa que os reducers [devem ser puros](/learn/keeping-components-pure)—as mesmas entradas sempre resultam na mesma saída. Eles não devem enviar solicitações, programar timeouts ou executar quaisquer efeitos colaterais (operações que afetam coisas fora do componente). Eles devem atualizar [objetos](/learn/updating-objects-in-state) e [arrays](/learn/updating-arrays-in-state) sem mutações.
- **Cada action descreve uma única interação do usuário, mesmo que isso leve a várias alterações nos dados.** Por exemplo, se um usuário pressiona "Redefinir" em um formulário com cinco campos gerenciados por um reducer, faz mais sentido despachar uma única action `reset_form` ao invés de cinco ações `set_field` separadas. Se você registrar cada action em um reducer, esse registro deve ser claro o suficiente para você reconstruir quais interações ou respostas aconteceram e em que ordem. Isso ajuda na depuração!

## Escrevendo reducers concisos com Immer {/*writing-concise-reducers-with-immer*/}

Assim como com [a atualização de objetos](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) e [arrays](/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) no estado regular, você pode usar a biblioteca Immer para tornar os reducers mais concisos. Aqui, [`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) permite que você mute o estado com a atribuição `push` ou `arr[i] =`:

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
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Ação desconhecida: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

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
      <h1>Roteiro de Praga</h1>
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
  {id: 0, text: 'Visitar o Museu Kafka', done: true},
  {id: 1, text: 'Assistir a um espetáculo de fantoches', done: false},
  {id: 2, text: 'Foto no Muro de Lennon', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Adicionar tarefa"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Adicionar
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
        <button onClick={() => setIsEditing(false)}>Salvar</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Editar</button>
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
      <button onClick={() => onDelete(task.id)}>Apagar</button>
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

Reducers devem ser puros, então eles não devem mutar o estado. Mas o Immer fornece um objeto `draft` especial que é seguro para mutar. Por baixo dos panos, o Immer irá criar uma cópia do seu estado com as alterações que você fez no `draft`. É por isso que os reducers gerenciados por `useImmerReducer` podem mutar seu primeiro argumento e não precisam retornar o estado.

<Recap>

- Para converter de `useState` para `useReducer`:
  1. Despache actions dos manipuladores de eventos.
  2. Escreva uma função de reducer que retorna o próximo estado para um determinado estado e action.
  3. Substitua `useState` por `useReducer`.
- Reducers exigem que você escreva um pouco mais de código, mas eles ajudam com depuração e testes.
- Reducers devem ser puros.
- Cada action descreve uma única interação do usuário.
- Use Immer se quiser escrever reducers em um estilo de mutação.

</Recap>

<Challenges>

#### Disparar actions a partir dos manipuladores de eventos {/*dispatch-actions-from-event-handlers*/}

Atualmente, os manipuladores de eventos em `ContactList.js` e `Chat.js` têm comentários `// TODO`. É por isso que digitar no input não funciona e clicar nos botões não altera o destinatário selecionado.

Substitua esses dois `// TODO` pelos códigos para `dispatch` das ações correspondentes. Para ver o formato esperado e o tipo das actions, verifique o reducer em `messengerReducer.js`. O reducer já foi escrito, então você não precisará alterá-lo. Você só precisa despachar as actions em `ContactList.js` e `Chat.js`.

<Hint>

A função `dispatch` já está disponível em ambos esses componentes porque foi passada como uma prop. Então você precisa chamar `dispatch` com o objeto de action correspondente.

Para verificar o formato do objeto de action, você pode olhar o reducer e ver quais campos `action` ele espera ver. Por exemplo, o caso `changed_selection` no reducer se parece com isto:

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId
  };
}
```

Isso significa que o seu objeto de action deve ter um `type: 'changed_selection'`. Você também vê o `action.contactId` sendo usado, então você precisa incluir uma propriedade `contactId` na sua action.

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('Ação desconhecida: ' + action.type);
    }
  }
}
```
```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                // TODO: dispatch changed_selection
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Chat to ' + contact.name}
        onChange={(e) => {
          // TODO: dispatch edited_message
          // (Read the input value from e.target.value)
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Sending "${message}" to ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Send to {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
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
