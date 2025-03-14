```js
// Guia de Estilo Universal
import { useState } from 'react';
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

```js
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

```js
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

As you can see, the event handlers no longer contain any state setting logic:

- `handleAddTask` dispatches an `'added'` action.
- `handleChangeTask` dispatches a `'changed'` action.
- `handleDeleteTask` dispatches a `'deleted'` action.

The reducer function, which you provided to `useReducer`, handles the processing of all these actions and returns the next state.

## When to use a reducer {/*when-to-use-a-reducer*/}

Generally, reducers are useful when:

*   You have complex state logic that involves multiple sub-values.
*   The next state depends on the previous state.
*   An event handler may need to update several pieces of state at once.

If your state updates are simple, using `useState` may be easier. However, a reducer can help you keep all the state update logic in the same place.

## Writing reducers well {/*writing-reducers-well*/}

Here are some tips for writing reducers well:

*   **Reducers should be pure.** Like `useState` updates, reducers should not modify objects that already exist but instead return new ones. In other words, your reducer function should be [pure](https://en.wikipedia.org/wiki/Pure_function): it should "compute" and return the next state instead of modifying the variables you pass in.
*   **Each action describes one user interaction.** The `type` of each action should describe what happened to the user, such as `'added'`, `'changed'`, or `'deleted'`.
*   **Each action contains the minimal data needed to apply the change.** For example, instead of passing the entire task object for `'changed'`, you could just pass the new values. This makes it easier to “replay” the actions to get back to the same state in the future (for example, for debugging or state persistence).
*   **Give descriptive names to your action types.** Be specific! Instead of `'add'`, consider `'add_item_to_cart'` or `'add_task'`.
*   **Avoid complex logic inside reducers.** If the logic to compute the next state is too complex, consider moving some of it outside the reducer. For example, you could set up the data needed to calculate a new state, and then pass it as the action contents.

By following these guidelines, your components will be easier to understand, debug, and test!
```js
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
      <button onClick={() => onDelete(task.id)}>Deletar</button>
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

Se quiser, você pode até mesmo mover o redutor para um arquivo diferente:

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
  {id: 1, text: 'Assistir a um show de fantoches', done: false},
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
      <button onClick={() => onDelete(task.id)}>Deletar</button>
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

A lógica do componente pode ser mais fácil de ler quando você separa as preocupações desta forma. Agora, os manipuladores de eventos só especificam _o que aconteceu_ ao despachar ações, e a função redutora determina _como o estado é atualizado_ em resposta a elas.

## Comparando `useState` e `useReducer` {/*comparing-usestate-and-usereducer*/}

Os redutores não são isentos de desvantagens! Aqui estão algumas maneiras de compará-los:

- **Tamanho do código:** Geralmente, com `useState` você tem que escrever menos código antecipadamente. Com `useReducer`, você tem que escrever uma função redutora _e_ despachar ações. No entanto, `useReducer` pode ajudar a reduzir o código se muitos manipuladores de eventos modificarem o estado de maneira semelhante.
- **Legibilidade:** `useState` é muito fácil de ler quando as atualizações de estado são simples. Quando elas ficam mais complexas, podem inflar o código do seu componente e dificultar a verificação. Nesse caso, `useReducer` permite que você separe claramente o _como_ da lógica de atualização do _o que aconteceu_ dos manipuladores de eventos.
- **Depuração:** Quando você tem um erro com `useState`, pode ser difícil dizer _onde_ o estado foi definido incorretamente e _por que_. Com `useReducer`, você pode adicionar um console log ao seu redutor para ver cada atualização de estado e _por que_ ela aconteceu (devido a qual `ação`). Se cada `ação` estiver correta, você saberá que o erro está na própria lógica do redutor. No entanto, você tem que percorrer mais código do que com `useState`.
- **Teste:** Um redutor é uma função pura que não depende do seu componente. Isso significa que você pode exportá-lo e testá-lo separadamente, isoladamente. Embora geralmente seja melhor testar componentes em um ambiente mais realista, para uma lógica complexa de atualização de estado, pode ser útil afirmar que seu redutor retorna um determinado estado para um estado inicial e ação específicos.
- **Preferência pessoal:** Algumas pessoas gostam de redutores, outras não. Tudo bem. É uma questão de preferência. Você sempre pode converter entre `useState` e `useReducer` e vice-versa: eles são equivalentes!

Recomendamos o uso de um redutor se você frequentemente encontrar erros devido a atualizações de estado incorretas em algum componente e quiser introduzir mais estrutura em seu código. Você não precisa usar redutores para tudo: sinta-se à vontade para misturar e combinar! Você pode até usar `useState` e `useReducer` no mesmo componente.

## Escrevendo redutores bem {/*writing-reducers-well*/}

Tenha estas duas dicas em mente ao escrever redutores:

- **Redutores devem ser puros.** Semelhantes às [funções de atualização de estado](/learn/queueing-a-series-of-state-updates), os redutores são executados durante a renderização! (As ações são enfileiradas até a próxima renderização.) Isso significa que os redutores [devem ser puros](/learn/keeping-components-pure)—as mesmas entradas sempre resultam na mesma saída. Eles não devem enviar solicitações, agendar timeouts ou realizar quaisquer efeitos colaterais (operações que afetam coisas fora do componente). Eles devem atualizar [objetos](/learn/updating-objects-in-state) e [arrays](/learn/updating-arrays-in-state) sem mutações.
- **Cada ação descreve uma única interação do usuário, mesmo que isso leve a várias alterações nos dados.** Por exemplo, se um usuário pressiona "Redefinir" em um formulário com cinco campos gerenciados por um redutor, faz mais sentido despachar uma ação `reset_form` em vez de cinco ações `set_field` separadas. Se você registrar cada ação em um redutor, esse registro deve ser claro o suficiente para que você reconstrua quais interações ou respostas aconteceram e em que ordem. Isso ajuda na depuração!

## Escrevendo redutores concisos com Immer {/*writing-concise-reducers-with-immer*/}

Assim como com [a atualização de objetos](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) e [arrays](/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) no estado normal, você pode usar a biblioteca Immer para tornar os redutores mais concisos. Aqui, [`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) permite que você mute o estado com atribuição `push` ou `arr[i] =`:

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
  {id: 1, text: 'Assistir a um show de fantoches', done: false},
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
      <button onClick={() => onDelete(task.id)}>Deletar</button>
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

Os redutores devem ser puros, então eles não devem mutar o estado. Mas o Immer fornece um objeto `draft` especial que é seguro para mutar. Nos bastidores, Immer criará uma cópia do seu estado com as alterações que você fez no `draft`. É por isso que os redutores gerenciados por `useImmerReducer` podem mutar seu primeiro argumento e não precisam retornar o estado.

<Recap>

- Para converter de `useState` para `useReducer`:
  1. Despache ações de manipuladores de eventos.
  2. Escreva uma função redutora que retorna o próximo estado para um determinado estado e ação.
  3. Substitua `useState` por `useReducer`.
- Os redutores exigem que você escreva um pouco mais de código, mas eles ajudam na depuração e nos testes.
- Os redutores devem ser puros.
- Cada ação descreve uma única interação do usuário.
- Use Immer se quiser escrever redutores em um estilo de mutação.

</Recap>

<Challenges>

#### Despachar ações de manipuladores de eventos {/*dispatch-actions-from-event-handlers*/}

Atualmente, os manipuladores de eventos em `ContactList.js` e `Chat.js` têm comentários `// TODO`. É por isso que digitar no campo de entrada não funciona e clicar nos botões não altera o destinatário selecionado.

Substitua esses dois `// TODO`s pelo código para `despachar` as ações correspondentes. Para ver a forma esperada e o tipo das ações, verifique o redutor em `messengerReducer.js`. O redutor já está escrito, então você não precisará alterá-lo. Você só precisa despachar as ações em `ContactList.js` e `Chat.js`.

<Hint>

A função `dispatch` já está disponível em ambos os componentes porque foi passada como uma prop. Então você precisa chamar `dispatch` com o objeto de ação correspondente.

Para verificar a forma do objeto de ação, você pode olhar para o redutor e ver quais campos `action` ele espera ver. Por exemplo, o caso `changed_selection` no redutor é assim:

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId
  };
}
```

Isso significa que seu objeto de ação deve ter um `type: 'changed_selection'`. Você também vê o `action.contactId` sendo usado, então você precisa incluir uma propriedade `contactId` em sua ação.

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
``````js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                
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
```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
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
  messages: {
    0: 'Hello, Taylor',
    1: 'Hello, Alice',
    2: 'Hello, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

```js src/MyReact.js
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```
