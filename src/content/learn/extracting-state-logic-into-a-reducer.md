```js
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

</Sandpack>

>Observação: no exemplo acima, o redutor foi declarado dentro do mesmo arquivo que o componente. Em aplicações maiores, você pode querer movê-lo para um arquivo separado.

Agora, em vez de chamar `setTasks` de dentro de seus manipuladores de eventos, eles acionam ações como `dispatch({ type: 'added', id: nextId++, text: text })`. O redutor então recebe a ação e o estado atual, computa o próximo estado e React atualiza a UI. Todo o estado é consolidado em uma única função, o que torna mais fácil de entender como o estado é alterado.

## Quando usar um redutor {/*when-to-use-a-reducer*/}

Redutores são úteis, mas talvez não sejam apropriados para todos os componentes.

**Use um redutor se:**

*   Você tem uma lógica de atualização de estado complexa.
*   O próximo estado depende do estado anterior.
*   Você quer otimizar o desempenho porque uma das suas ações faz uma operação cara.
*   Você está se sentindo confuso sobre como o estado está sendo atualizado.

**Se seu estado de componentes consistir apenas em alguns pedaços de dados e não tiver um comportamento complexo, pode ser mais simples usar `useState`.**

Por exemplo, talvez você não precise de um redutor para este componente:

```js
function MyComponent() {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  // ...
```

Ao implementar um redutor, você está trocando alguma simplicidade pela flexibilidade e controle extra.

## Escrevendo um redutor bem {/*writing-a-reducer-well*/}

Aqui estão algumas dicas sobre como escrever redutores:

*   **Os redutores devem ser funções puras.** Como as funções puras, os redutores devem "calcular" o próximo estado com base no `state` e nos `action`s. Eles devem ler os valores de `state` e `action` (e seus argumentos) e não fazer mais nada. Em particular:
    *   Não modifique os objetos de `state` ou `action`—trate-os como somente leitura.
    *   Não chame APIs (por exemplo, `fetch`).
    *   Não chame funções que tenham efeitos colaterais (por exemplo, `console.log()`).
    *   Não chame funções que não sejam puras (por exemplo, `Date.now()` ou `Math.random()`).
*   **O manipulador de cada ação deve retornar o estado.** Se uma ação não for relevante para que o estado seja atualizado, retorne o estado atual sem alterações. Caso contrário, retorne o *novo* estado.
*   **O estado não precisa ser um objeto.** No exemplo acima, ele era um array. Ele pode ser um número, uma string ou qualquer tipo de dados JavaScript.
*   **Cada caso pode ter um nome de ação diferente.** Há duas abordagens comuns:
    *   Ações individuais, como `"added"`, `"changed"` e `"deleted"` no exemplo acima.
    *   Uma ação geral, como `"update"`, e informações sobre como atualizar o estado nos `action`:  Por exemplo, `dispatch({ type: 'update', payload: { id: 1, text: 'Olá' } })`.
    *   Você tem controle total sobre a forma das _ações_ que sua aplicação envia e sobre a forma do _estado_ que seu redutor retorna. Escolha o que for mais adequado para suas necessidades.

Usar um redutor pode ser um ótimo padrão para lidar com atualizações complexas de estado.
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

Se quiser, você pode até mesmo mover o **reducer** para um arquivo diferente:

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
      <h1>Itinerário de Praga</h1>
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
  {id: 2, text: 'Foto no muro de Lennon', done: false},
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
      throw Error('Unknown action: ' + action.type);
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

A lógica do componente pode ser mais fácil de ler quando você separa as preocupações dessa maneira. Agora, os manipuladores de eventos especificam apenas _o que aconteceu_ ao despachar ações, e a função redutora determina _como o estado é atualizado_ em resposta a elas.

## Comparando `useState` e `useReducer` {/*comparing-usestate-and-usereducer*/}

Redutores não são isentos de desvantagens! Aqui estão algumas maneiras de compará-los:

- **Tamanho do código:** Geralmente, com `useState`, você precisa escrever menos código antecipadamente. Com `useReducer`, você precisa escrever tanto uma função redutora _quanto_ despachar ações. No entanto, `useReducer` pode ajudar a reduzir o código se muitos manipuladores de eventos modificam o estado de maneira semelhante.
- **Legibilidade:** `useState` é muito fácil de ler quando as atualizações de estado são simples. Quando eles ficam mais complexos, eles podem inflar o código do seu componente e dificultar a verificação. Nesse caso, `useReducer` permite que você separe de forma limpa o _como_ da lógica de atualização do _o que aconteceu_ dos manipuladores de eventos.
- **Depuração:** Quando você tem um erro com `useState`, pode ser difícil dizer _onde_ o estado foi definido incorretamente e _por quê_. Com `useReducer`, você pode adicionar um log de console em seu redutor para ver cada atualização de estado e _por que_ ela aconteceu (devido a qual `ação`). Se cada `ação` estiver correta, você saberá que o erro está na própria lógica do redutor. No entanto, você precisa percorrer mais código do que com `useState`.
- **Teste:** Um redutor é uma função pura que não depende do seu componente. Isso significa que você pode exportá-lo e testá-lo separadamente em isolamento. Embora geralmente seja melhor testar componentes em um ambiente mais realista, para lógica complexa de atualização de estado, pode ser útil afirmar que seu redutor retorna um estado particular para um estado inicial e ação específicos.
- **Preferência pessoal:** Algumas pessoas gostam de redutores, outras não. Tudo bem. É uma questão de preferência. Você sempre pode converter entre `useState` e `useReducer` para frente e para trás: eles são equivalentes!

Recomendamos o uso de um redutor se você frequentemente encontrar erros devido a atualizações incorretas de estado em algum componente e quiser introduzir mais estrutura ao seu código. Você não precisa usar redutores para tudo: sinta-se à vontade para misturar e combinar! Você pode até mesmo usar `useState` e `useReducer` no mesmo componente.

## Escrevendo redutores bem {/*writing-reducers-well*/}

Tenha estas duas dicas em mente ao escrever redutores:

- **Redutores devem ser puros.** Semelhante às [funções atualizadoras de estado](/learn/queueing-a-series-of-state-updates), os redutores são executados durante a renderização! (As ações são enfileiradas até a próxima renderização.) Isso significa que os redutores [devem ser puros](/learn/keeping-components-pure) — as mesmas entradas sempre resultam na mesma saída. Eles não devem enviar solicitações, agendar timeouts ou realizar quaisquer efeitos colaterais (operações que afetam coisas fora do componente). Eles devem atualizar [objetos](/learn/updating-objects-in-state) e [arrays](/learn/updating-arrays-in-state) sem modificações.
- **Cada ação descreve uma única interação do usuário, mesmo que isso leve a múltiplas alterações nos dados.** Por exemplo, se um usuário pressiona "Redefinir" em um formulário com cinco campos gerenciados por um redutor, faz mais sentido despachar uma ação `reset_form` em vez de cinco ações `set_field` separadas. Se você registrar cada ação em um redutor, esse log deve ser claro o suficiente para que você possa reconstruir quais interações ou respostas aconteceram e em que ordem. Isso ajuda na depuração!

## Escrevendo redutores concisos com Immer {/*writing-concise-reducers-with-immer*/}

Assim como com [a atualização de objetos](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) e [arrays](/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) no estado regular, você pode usar a biblioteca Immer para tornar os redutores mais concisos. Aqui, [`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) permite que você modifique o estado com atribuição `push` ou `arr[i] =`:

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
      throw Error('Unknown action: ' + action.type);
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
      <h1>Itinerário de Praga</h1>
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
  {id: 2, text: 'Foto no muro de Lennon', done: false},
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

Redutores devem ser puros, portanto, eles não devem modificar o estado. Mas Immer fornece a você um objeto `draft` especial que é seguro para modificar. Por baixo dos panos, Immer criará uma cópia do seu estado com as alterações que você fez no `draft`. É por isso que os redutores gerenciados por `useImmerReducer` podem modificar seu primeiro argumento e não precisam retornar o estado.

<Recap>

- Para converter de `useState` para `useReducer`:
  1. Despache ações de manipuladores de eventos.
  2. Escreva uma função redutora que retorna o próximo estado para um determinado estado e ação.
  3. Substitua `useState` por `useReducer`.
- Os redutores exigem que você escreva um pouco mais de código, mas eles ajudam na depuração e no teste.
- Os redutores devem ser puros.
- Cada ação descreve uma única interação do usuário.
- Use Immer se você quiser escrever redutores em um estilo modificador.

</Recap>

<Challenges>

#### Despachar ações de manipuladores de eventos {/*dispatch-actions-from-event-handlers*/}

Atualmente, os manipuladores de eventos em `ContactList.js` e `Chat.js` têm comentários `// TODO`. É por isso que digitar na entrada não funciona e clicar nos botões não altera o destinatário selecionado.

Substitua esses dois `// TODO` com o código para `despachar` as ações correspondentes. Para ver o formato esperado e o tipo das ações, verifique o redutor em `messengerReducer.js`. O redutor já está escrito, portanto, você não precisará alterá-lo. Você só precisa despachar as ações em `ContactList.js` e `Chat.js`.

<Hint>

A função `dispatch` já está disponível em ambos os componentes porque foi passada como uma propriedade. Portanto, você precisa chamar `dispatch` com o objeto de ação correspondente.

Para verificar o formato do objeto de ação, você pode olhar para o redutor e ver quais campos `action` ele espera ver. Por exemplo, o caso `changed_selection` no redutor se parece com isto:

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId
  };
}
```

Isso significa que o seu objeto de ação deve ter um `type: 'changed_selection'`. Você também vê que `action.contactId` está sendo usado, então você precisa incluir uma propriedade `contactId` na sua ação.

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
      throw Error('Unknown action: ' + action.type);
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
