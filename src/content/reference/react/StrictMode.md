```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

const initialRoomId = 'general';

export default function App() {
  const [roomId, setRoomId] = useState(initialRoomId);
  const [showChat, setShowChat] = useState(false);
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
      <button onClick={() => setShowChat(!showChat)}>
        {showChat ? 'Close chat' : 'Open chat'}
      </button>
      {showChat && <ChatRoom roomId={roomId} />}
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

Notice how the "Active connections" in the console kept increasing as you switched chat rooms. The problem is that the `useEffect` doesn't have a cleanup function. Every time `ChatRoom` re-renders with a different `roomId`, it creates a *new* connection, but it never closes the old one. This is why connections keep piling up. To fix this, specify a cleanup function:

```js {6,8}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
```

Without cleanup, you would have to keep using the app and manually check the console to see if the number of active connections keeps increasing. However, **when you use `<StrictMode>`, React will simulate this situation automatically**. Instead of mounting only one time, React will simulate mounting, immediately then cleaning-up, and then mounting again. The cleanup function is missing in the original code, so you would see the following in the console:

*   `✅ Connecting to "general" room at https://localhost:1234...`
*   `Active connections: 1`
*   `❌ Disconnected from "general" room at https://localhost:1234`
*   `Active connections: 0`
*   `✅ Connecting to "general" room at https://localhost:1234...`
*   `Active connections: 1`

The extra log message in the console immediately reveals that there's an issue. Let's wrap the original (buggy) code in `<StrictMode>`:

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

const initialRoomId = 'general';

export default function App() {
  const [roomId, setRoomId] = useState(initialRoomId);
  const [showChat, setShowChat] = useState(false);
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
      <button onClick={() => setShowChat(!showChat)}>
        {showChat ? 'Close chat' : 'Open chat'}
      </button>
      {showChat && <ChatRoom roomId={roomId} />}
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

Notice how when you open the chat in this example, you see the `connect` and `disconnect` logs immediately. You can immediately see the problem with your code by looking at the logs.

You might think that this only matters when the component unmounts or when its dependencies change. That is partially true -- you still need to write correct cleanup logic! The point is that:

1.  Strict Mode makes you write cleanup logic, because it calls it more eagerly.
2.  When you write it, it also fixes real bugs that would be very difficult to reproduce without Strict Mode.

Strict Mode helps you write components that handle changes in dependencies correctly. This type of bug usually goes unnoticed, but can be very important, especially when you are loading resources, subscriptions, and connections.

### Fixing bugs found by re-running ref callbacks in development {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

Similar to Effects, [refs](/reference/react/useRef) may also require cleanup. For example, if you use a ref to manage focus on an element, you may want to remove the focus when the component unmounts.

When Strict Mode is on, React calls your ref callbacks in the following order:

1.  `ref.current = null` (if the current ref has a value)
2.  Your ref callback with the DOM node
3.  Your ref callback with `null` (as cleanup)

If you need to perform cleanup work to prevent memory leaks, you should implement that in your refs.

**Here is an example to illustrate how re-running ref callbacks in Strict Mode helps you find bugs early.**

Consider this component that uses a ref to focus a text input:

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

```js src/App.js
import { useRef, useEffect } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);

  useEffect(() => {
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

There is an issue with this code, but it might not be immediately clear. Let's change the example to include a focus counter. It will show the number of times the input field had focus:

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

```js src/App.js
import { useRef, useEffect, useState } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);
  const [focusCount, setFocusCount] = useState(0);

  useEffect(() => {
    inputRef.current.focus();
    setFocusCount(focusCount + 1);
  }, []);

  return (
    <>
      <input ref={inputRef} />
      <p>Focus count: {focusCount}</p>
    </>
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

The `focusCount` starts at 0, but the input field is focused only one time. This behavior indicates a bug. Let's fix the same, but add the StrictMode to the example:

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

```js src/App.js
import { useRef, useEffect, useState } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);
  const [focusCount, setFocusCount] = useState(0);

  useEffect(() => {
    inputRef.current.focus();
    setFocusCount(focusCount + 1);
  }, []);

  return (
    <>
      <input ref={inputRef} />
      <p>Focus count: {focusCount}</p>
    </>
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

With Strict Mode on, you will see a different number in the focus counter, e.g. 1. The input field is focused only one time, but the counter has a number other than 0. This shows that `focusCount` has already changed before the input field is focused again.

To fix this, you need to make sure that the state update is triggered correctly:

```js {8}
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
      setFocusCount(c => c + 1);
    }
  }, []);
```

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

```js src/App.js
import { useRef, useEffect, useState } from 'react';

export default function MyForm() {
  const inputRef = useRef(null);
  const [focusCount, setFocusCount] = useState(0);

  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
      setFocusCount(c => c + 1);
    }
  }, []);

  return (
    <>
      <input ref={inputRef} />
      <p>Focus count: {focusCount}</p>
    </>
  );
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Now, the application behaves correctly, and the component is correctly handled.

### Fixing deprecation warnings enabled by Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

Strict Mode also enables some additional deprecation warnings. For example, it may warn you about:

*   Using deprecated methods like `findDOMNode`.
*   Using legacy context APIs.
*   Using string refs.

These warnings are not critical but indicate that your component may rely on legacy or soon-to-be-removed APIs. While these APIs may still work, it's generally a good idea to eliminate such warnings to make sure you're using the modern React best practices.

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

  return <h1>Bem vindo(a) ao chat {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
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
        {show ? 'Fechar chat' : 'Abrir chat'}
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
      console.log('❌ Desconectado(a) da sala "' + roomId + '" em ' + serverUrl);
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

Você notará que o número de conexões abertas continua sempre crescendo. Em um aplicativo real, isso causaria problemas de performance e de rede. O problema é que [seu Effect está faltando uma função de limpeza:](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)

```js {4}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
```

Agora que seu Effect "limpa" após si mesmo e destrói as conexões desatualizadas, o vazamento é resolvido. No entanto, observe que o problema não se tornou visível até que você tenha adicionado mais funcionalidades (a caixa de seleção).

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
  return <h1>Bem vindo(a) ao chat {roomId}!</h1>;
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
      console.log('❌ Desconectado(a) da sala "' + roomId + '" em ' + serverUrl);
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

**Com o Strict Mode, você vê imediatamente que há um problema** (o número de conexões ativas pula para 2). O Strict Mode executa um ciclo extra de setup+cleanup para cada Effect. Esse Effect não tem nenhuma lógica de limpeza, então ele cria uma conexão extra, mas não a destrói. Essa é uma dica de que você está faltando uma função de limpeza.

O Strict Mode permite que você note esses erros no início do processo. Quando você corrige seu Effect adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis erros de produção futuros, como a caixa de seleção anterior:

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

  return <h1>Bem vindo(a) ao chat {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
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
        {show ? 'Fechar chat' : 'Abrir chat'}
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
      console.log('❌ Desconectado(a) da sala "' + roomId + '" em ' + serverUrl);
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

Observe como a contagem de conexão ativa no console não continua crescendo mais.

Sem o Strict Mode, era fácil perder que seu Effect precisava de limpeza. Ao executar *setup → cleanup → setup* em vez de *setup* para seu Effect no desenvolvimento, o Strict Mode tornou a lógica de limpeza ausente mais perceptível.

[Leia mais sobre como implementar a limpeza do Effect.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---
### Consertando erros encontrados pela execução repetida dos callbacks de ref no desenvolvimento {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

O Strict Mode também pode ajudar a encontrar erros em [callbacks refs.](/learn/manipulating-the-dom-with-refs)

Cada callback `ref` tem algum código de configuração e pode ter algum código de limpeza. Normalmente, o React chama o setup quando o elemento é *criado* (é adicionado ao DOM) e chama o cleanup quando o elemento é *removido* (é removido do DOM).

Quando o Strict Mode está ativado, o React também executará **um ciclo extra de setup+cleanup no desenvolvimento para cada `ref` de callback.** Isso pode parecer surpreendente, mas ajuda a revelar erros sutis que são difíceis de detectar manualmente.

Considere este exemplo, que permite selecionar um animal e, em seguida, rolar para um deles. Observe que, ao mudar de "Cats" para "Dogs", os logs do console mostram que o número de animais na lista continua crescendo e os botões "Scroll to" param de funcionar:

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
                    // 🚩 No cleanup, this is a bug!
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

**Este é um erro de produção!** Como o callback de ref não remove animais da lista no cleanup, a lista de animais continua crescendo. Este é um vazamento de memória que pode causar problemas de performance em um aplicativo real e quebra o comportamento do aplicativo.

O problema é que o callback ref não limpa após si mesmo:

```js {6-8}
<li
  ref={node => {
    const list = itemsRef.current;
    const item = {animal, node};
    list.push(item);
    return () => {
      // 🚩 No cleanup, this is a bug!
    }
  }}
</li>
```

Agora, vamos encapsular o código original (com erro) em `<StrictMode>`:

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
                    // 🚩 No cleanup, this is a bug!
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

**Com o Strict Mode, você vê imediatamente que há um problema**. O Strict Mode executa um ciclo extra de setup+cleanup para cada ref de callback. Esse ref de callback não tem lógica de limpeza, então ele adiciona refs, mas não as remove. Essa é uma dica de que você está faltando uma função de limpeza.

O Strict Mode permite que você encontre erros em refs de callback. Quando você corrige seu callback adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis erros de produção futuros, como o erro "Scroll to" anterior:

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

Agora, na montagem inicial no StrictMode, os callbacks de ref são todos configurados, limpos e configurados novamente:

```
...
✅ Adicionando animal ao mapa. Animais no total: 10
...
❌ Removendo animal do mapa. Animais no total: 0
...
✅ Adicionando animal ao mapa. Animais no total: 10
```

**Isso é esperado.** O Strict Mode confirma que os callbacks de ref são limpos corretamente, então o tamanho nunca cresce acima da quantidade esperada. Após a correção, não há vazamentos de memória e todos os recursos funcionam como esperado.

Sem o Strict Mode, era fácil perder o erro até que você clicasse no aplicativo para notar recursos quebrados. O Strict Mode fez com que os erros aparecessem imediatamente, antes que você os enviasse para a produção.

---
### Consertando avisos de depreciação habilitados pelo Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

O React avisa se algum componente dentro de uma árvore `<StrictMode>` usa uma dessas APIs descontinuadas:

* Métodos de ciclo de vida da classe `UNSAFE_` como [`UNSAFE_componentWillMount`](/reference/react/Component#unsafe_componentwillmount). [Veja alternativas.](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#migrating-from-legacy-lifecycles)

Essas APIs são usadas principalmente em [componentes de classe](/reference/react/Component) mais antigos, então elas raramente aparecem em aplicativos modernos.
