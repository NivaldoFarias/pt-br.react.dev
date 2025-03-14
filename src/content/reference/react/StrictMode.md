js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ❌ This causes a memory leak!
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Notice that even though you pick different chat rooms, the number of active connections in the console never decreases. This is because the `createConnection` function does not provide a way to disconnect from a chat, so there is no way to cleanup your Effect.

To fix this problem, you can return a cleanup function from the Effect. When the Effect needs to re-run or when the component unmounts (is removed from the screen), React will call this cleanup function. In the cleanup function, you release any resources that were acquired by the Effect. In this case, you would call `connection.disconnect()`:

```js {5,6,7}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
```

This solves the resource leak, but you would need to run the app for a while to realize that something is wrong. Let's wrap the original (buggy) code in `<StrictMode>`:

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ❌ This causes a memory leak!
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

With Strict Mode on, React will now run your setup code, and then the cleanup code, and then the setup code again:

```
✅ Connecting to "general" room at https://localhost:1234...
Active connections: 1
❌ Disconnected from "general" room at https://localhost:1234
Active connections: 0
```

The log shows the cleanup code running immediately after the setup code, so you can see the missing cleanup code right away. When you fix your component to properly cleanup in Strict Mode, you also fix possible production bugs:

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

With the bug fixed, the log now shows both disconnect and connect calls every time you switch between chat rooms:

```
✅ Connecting to "general" room at https://localhost:1234...
Active connections: 1
❌ Disconnected from "general" room at https://localhost:1234
Active connections: 0
✅ Connecting to "travel" room at https://localhost:1234...
Active connections: 1
❌ Disconnected from "travel" room at https://localhost:1234
Active connections: 0
✅ Connecting to "music" room at https://localhost:1234...
Active connections: 1
❌ Disconnected from "music" room at https://localhost:1234
Active connections: 0
```

Without Strict Mode, it would be easy to miss the leak until you tried connecting to multiple chat rooms. Strict Mode helps you find it right away. Strict Mode helps you find bugs before you push them to your team and to your users.

[Read more about synchronizing with Effects.](/learn/synchronizing-with-effects)

---

### Fixing bugs found by re-running ref callbacks in development {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

Strict Mode can also help find bugs caused by missing cleanup in `ref` callbacks.

React calls `ref` callbacks:

*   When a component mounts
*   When a component unmounts

When Strict Mode is on, React calls `ref` callbacks:

*   When a component mounts
*   **Immediately after**, when the component mounts
*   When a component unmounts

What is the same pattern as [Effects re-running](#fixing-bugs-found-by-re-running-effects-in-development), this helps to find bugs caused by missing cleanup in `ref` callbacks.

**Here is an example to illustrate how re-running ref callbacks in Strict Mode helps you find bugs early.**

Consider this example that automatically focuses an input on the screen:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js
import { useRef, useEffect } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);

  useEffect(() => {
    // 🔴 Errado: Isto vai causar um erro no Strict Mode!
    inputRef.current.focus();
  }, []);

  return (
    <input ref={inputRef} />
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

There is an issue with this code, but it might not be immediatelly clear. In the example above the `inputRef` is set on the input element. Then, in the `useEffect`, a call is made to focus on the input. However, because of the order in which React calls ref callbacks and Effects, there is a subtle race condition:

1.  React creates the input element in the DOM.
2.  React calls the `ref` callback with the input DOM node, setting `inputRef.current` to point to it.
3.  React runs the `useEffect` after setting `inputRef.current`.
4.  If the `useEffect` happens to run *before* the browser paints the updated UI, then there is a race condition. The browser might not yet see the input HTML element, and calling `inputRef.current.focus()` would cause the browser to throw an error.

To fix this problem run the focus after the browser paints the updated UI:

```js
import { useRef, useEffect } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);

  useEffect(() => {
    // ✅ Correto: Isto não causará erros.
    inputRef.current?.focus();
  }, []);

  return (
    <input ref={inputRef} />
  );
}
```

This would fix the race condition. However, for the sake of this example we can also move the `focus()` call to the `ref` callback:

```js {5,6,7}
  const inputRef = useRef(null);

  function handleInputRef(inputNode) {
    if (inputNode !== null) {
      // ✅ Correto: Isto não causará erros.
      inputNode.focus();
    }
  }

  return (
    <input ref={handleInputRef} />
  );
```

This makes the code run at the correct time, and avoids a possible error. Let's wrap the initial code snippet in `<StrictMode>` to see the error:

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useRef, useEffect } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);

  useEffect(() => {
    // 🔴 Errado: Isto vai causar um erro no Strict Mode!
    inputRef.current?.focus();
  }, []);

  return (
    <input ref={inputRef} />
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Now, React will call the `ref` callback:

*   When the component mounts
*   **Immediately after**, when the component mounts, before `useEffect` is run
*   When the component unmounts

This will trigger the race condition, and the browser will throw an error because the input does't exist yet when you are calling `focus()`. Although, in the real world, this issue is most often found as you are running your application. Strict Mode will catch the issue right away! By fixing the component to properly cleanup the ref callback, you also fix other possible production bugs.

The corrected code is as follows:

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js active
import { useRef, useEffect } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);

  function handleInputRef(inputNode) {
    if (inputNode !== null) {
      // ✅ Correto: Isto não causará erros.
      inputNode.focus();
    }
  }

  return (
    <input ref={handleInputRef} />
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Strict Mode helps you find bugs before you do the push of your code to the team.

---

### Fixing deprecation warnings enabled by Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

In the future, Strict Mode will also warn about the usage of deprecated APIs. This will help you keep your code up-to-date as React evolves.

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Bem-vindo(a) ao chat da sala {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolher a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar bate-papo' : 'Abrir bate-papo'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
      connections++;
      console.log('Conexões ativas: ' + connections);
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
      connections--;
      console.log('Conexões ativas: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Você notará que o número de conexões abertas continua crescendo. Em um app real, isso causaria problemas de desempenho e de rede. O problema é que [seu Effect está faltando uma função de limpeza:](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)

```js {4}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
```

Agora que seu Effect "limpa" após si mesmo e destrói as conexões desatualizadas, o vazamento foi resolvido. No entanto, observe que o problema não se tornou visível até que você adicionou mais recursos (a caixa de seleção).

**No exemplo original, o erro não era óbvio. Agora, vamos encapsular o código original (com erro) em `<StrictMode>`:**

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, []);
  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
      connections++;
      console.log('Conexões ativas: ' + connections);
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
      connections--;
      console.log('Conexões ativas: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

**Com o Strict Mode, você vê imediatamente que há um problema** (o número de conexões ativas pula para 2). Strict Mode executa um ciclo extra de setup + cleanup para cada Effect. Este Effect não tem lógica de limpeza, então ele cria uma conexão extra, mas não a destrói. Esta é uma dica de que está faltando uma função de limpeza.

O Strict Mode permite que você note esses erros no início do processo. Quando você corrige seu Effect adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis erros de produção futuros, como a caixa de seleção de antes:

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolher a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar bate-papo' : 'Abrir bate-papo'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
      connections++;
      console.log('Conexões ativas: ' + connections);
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
      connections--;
      console.log('Conexões ativas: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Observe como a contagem de conexões ativas no console não continua mais crescendo.

Sem o Strict Mode, era fácil perder que seu Effect precisava de limpeza. Ao executar *setup → cleanup → setup* em vez de *setup* para seu Effect no desenvolvimento, o Strict Mode tornou a lógica de limpeza ausente mais notável.

[Leia mais sobre a implementação da limpeza do Effect.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---
### Corrigindo erros encontrados pela reexecução de retornos de chamada ref no desenvolvimento {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

O Strict Mode também pode ajudar a encontrar erros em [retornos de chamada ref.](/learn/manipulating-the-dom-with-refs)

Cada retorno de chamada `ref` tem algum código de setup e pode ter algum código de limpeza. Normalmente, o React chama o setup quando o elemento é *criado* (é adicionado ao DOM) e chama o cleanup quando o elemento é *removido* (é removido do DOM).

Quando o Strict Mode está ativado, o React também executará **um ciclo extra de setup+cleanup no desenvolvimento para cada `ref` de retorno de chamada.** Isso pode parecer surpreendente, mas ajuda a revelar erros sutis que são difíceis de detectar manualmente.

Considere este exemplo, que permite que você selecione um animal e, em seguida, role até um deles. Observe que, ao mudar de "Cats" para "Dogs", os logs do console mostram que o número de animais na lista continua crescendo e os botões "Scroll to" param de funcionar:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ❌ Not using StrictMode.
root.render(<App />);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function AnimalFriends() {
  const itemsRef = useRef([]);
  const [animalList, setAnimalList] = useState(setupAnimalList);
  const [animal, setAnimal] = useState('cat');

  function scrollToAnimal(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }
  
  const animals = animalList.filter(a => a.type === animal)
  
  return (
    <>
      <nav>
        <button onClick={() => setAnimal('cat')}>Cats</button>
        <button onClick={() => setAnimal('dog')}>Dogs</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{animals.map((animal, index) => (
          <button key={animal.src} onClick={() => scrollToAnimal(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {animals.map((animal) => (
              <li
                key={animal.src}
                ref={(node) => {
                  const list = itemsRef.current;
                  const item = {animal: animal, node}; 
                  list.push(item);
                  console.log(`✅ Adicionando animal ao mapa. Animais no total: ${list.length}`);
                  if (list.length > 10) {
                    console.log('❌ Muitos animais na lista!');
                  }
                  return () => {
                    // 🚩 Sem limpeza, isso é um erro!
                  }
                }}
              >
                <img src={animal.src} />
              </li>
            ))}
          
        </ul>
      </div>
    </>
  );
}

function setupAnimalList() {
  const animalList = [];
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'cat', src: "https://loremflickr.com/320/240/cat?lock=" + i});
  }
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'dog', src: "https://loremflickr.com/320/240/dog?lock=" + i});
  }

  return animalList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

**Este é um erro de produção!** Como o retorno de chamada ref não remove animais da lista na limpeza, a lista de animais continua crescendo. Este é um vazamento de memória que pode causar problemas de desempenho em um app real e interrompe o comportamento do app.

O problema é que o retorno de chamada ref não limpa após si mesmo:

```js {6-8}
<li
  ref={node => {
    const list = itemsRef.current;
    const item = {animal, node};
    list.push(item);
    return () => {
      // 🚩 Sem limpeza, isso é um erro!
    }
  }}
</li>
```

Agora vamos encapsular o código original (com erro) em `<StrictMode>`:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ Using StrictMode.
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function AnimalFriends() {
  const itemsRef = useRef([]);
  const [animalList, setAnimalList] = useState(setupAnimalList);
  const [animal, setAnimal] = useState('cat');

  function scrollToAnimal(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }
  
  const animals = animalList.filter(a => a.type === animal)
  
  return (
    <>
      <nav>
        <button onClick={() => setAnimal('cat')}>Cats</button>
        <button onClick={() => setAnimal('dog')}>Dogs</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{animals.map((animal, index) => (
          <button key={animal.src} onClick={() => scrollToAnimal(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {animals.map((animal) => (
              <li
                key={animal.src}
                ref={(node) => {
                  const list = itemsRef.current;
                  const item = {animal: animal, node} 
                  list.push(item);
                  console.log(`✅ Adicionando animal ao mapa. Animais no total: ${list.length}`);
                  if (list.length > 10) {
                    console.log('❌ Muitos animais na lista!');
                  }
                  return () => {
                    // 🚩 Sem limpeza, isso é um erro!
                  }
                }}
              >
                <img src={animal.src} />
              </li>
            ))}
          
        </ul>
      </div>
    </>
  );
}

function setupAnimalList() {
  const animalList = [];
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'cat', src: "https://loremflickr.com/320/240/cat?lock=" + i});
  }
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'dog', src: "https://loremflickr.com/320/240/dog?lock=" + i});
  }

  return animalList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

**Com o Strict Mode, você vê imediatamente que há um problema**. O Strict Mode executa um ciclo extra de setup+cleanup para cada ref de retorno de chamada. Este retorno de chamada ref não tem lógica de limpeza, então ele adiciona refs, mas não as remove. Esta é uma dica de que está faltando uma função de limpeza.

O Strict Mode permite que você encontre erros em refs de retorno de chamada. Quando você corrige seu retorno de chamada adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis erros de produção futuros, como o erro "Scroll to" de antes:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ Using StrictMode.
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function AnimalFriends() {
  const itemsRef = useRef([]);
  const [animalList, setAnimalList] = useState(setupAnimalList);
  const [animal, setAnimal] = useState('cat');

  function scrollToAnimal(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }
  
  const animals = animalList.filter(a => a.type === animal)
  
  return (
    <>
      <nav>
        <button onClick={() => setAnimal('cat')}>Cats</button>
        <button onClick={() => setAnimal('dog')}>Dogs</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{animals.map((animal, index) => (
          <button key={animal.src} onClick={() => scrollToAnimal(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {animals.map((animal) => (
              <li
                key={animal.src}
                ref={(node) => {
                  const list = itemsRef.current;
                  const item = {animal, node};
                  list.push({animal: animal, node});
                  console.log(`✅ Adicionando animal ao mapa. Animais no total: ${list.length}`);
                  if (list.length > 10) {
                    console.log('❌ Muitos animais na lista!');
                  }
                  return () => {
                    list.splice(list.indexOf(item));
                    console.log(`❌ Removendo animal do mapa. Animais no total: ${itemsRef.current.length}`);
                  }
                }}
              >
                <img src={animal.src} />
              </li>
            ))}
          
        </ul>
      </div>
    </>
  );
}

function setupAnimalList() {
  const animalList = [];
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'cat', src: "https://loremflickr.com/320/240/cat?lock=" + i});
  }
  for (let i = 0; i < 10; i++) {
    animalList.push({type: 'dog', src: "https://loremflickr.com/320/240/dog?lock=" + i});
  }

  return animalList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

Agora, na montagem inicial no StrictMode, os retornos de chamada ref são todos configurados, limpos e configurados novamente:

```
...
✅ Adicionando animal ao mapa. Animais no total: 10
...
❌ Removendo animal do mapa. Animais no total: 0
...
✅ Adicionando animal ao mapa. Animais no total: 10
```

**Isto é esperado.** O Strict Mode confirma que os retornos de chamada ref são limpos corretamente, então o tamanho nunca cresce acima da quantidade esperada. Após a correção, não há vazamentos de memória e todos os recursos funcionam como esperado.

Sem o Strict Mode, era fácil perder o erro até que você clicasse no aplicativo para notar recursos quebrados. O Strict Mode fez os erros aparecerem imediatamente, antes de enviá-los para a produção.

---
### Corrigindo avisos de depreciação ativados pelo Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

React adverte se algum componente em qualquer lugar dentro de uma árvore `<StrictMode>` usa uma destas APIs obsoletas:

* Métodos de ciclo de vida de classe `UNSAFE_` como [`UNSAFE_componentWillMount`](/reference/react/Component#unsafe_componentwillmount). [Veja alternativas.](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#migrating-from-legacy-lifecycles)

Essas APIs são usadas principalmente em [componentes de classe](/reference/react/Component) mais antigos, por isso raramente aparecem em apps modernos.
