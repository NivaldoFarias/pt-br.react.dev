```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // 🔴 Missing: disconnect
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

The code above is missing the `disconnect` function. Whenever you select a different room, you don't disconnect from the previous room. This leads to a memory leak in your application. With the implementation above, we can only visually inspect the `console.log` output. However, the bug is easily caught when Strict Mode is enabled.

**Now let's wrap the original (buggy) code in `<StrictMode>`:**

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
  const [show, setShow] = useState(false);
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // 🔴 Missing: disconnect
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

With Strict Mode, React ran the setup and then the cleanup for every Effect, so you could immediately see that you were missing some cleanup logic.

To fix the bug, specify how to disconnect from the chat by returning a cleanup function from your Effect:

```js {5,6,7}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
```

Here is the complete example:

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
  const [show, setShow] = useState(false);
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
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

The cleanup function returned from the Effect will run before the component unmounts, and also before the Effect runs again due to a dependency change. With the extra setup+cleanup cycle that Strict Mode introduces, this bug becomes easier to catch. Strict Mode helps you find these kinds of issues before users experience them.

[Read more about Effects.](/learn/synchronizing-with-effects)

---

### Fixing bugs found by re-running ref callbacks in development {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

Strict Mode can also help find bugs in the [ref callbacks](/learn/manipulating-the-dom-with-refs).

When Strict Mode is on, React will run the ref callback **twice** in development. This may feel surprising, but it helps reveal subtle bugs that are hard to catch manually.

**Here is an example to illustrate how re-running ref callbacks in Strict Mode helps you find bugs early.**

Consider this example that uses a ref to track whether a Checkbox is checked or not:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js
import { useRef } from 'react';

export default function MyForm() {
  const checkboxRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    alert(checkboxRef.current.checked);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        <input type="checkbox" ref={checkboxRef} />
        I agree
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

</Sandpack>

When the ref gets attached to an element, React calls the ref callback as `ref(checkboxRef.current)`.

In the above implementation, you use that ref to read the current value of `checkboxRef.current.checked` when the form is submitted. This is okay and there's nothing wrong with the code above. However, there might be cases where a ref is used to store some mutable state that can change, or where the callback is used to attach an event listener, or for other side effects. Double-invoking the callback can reveal a bug in these cases. For example, if you put `console.log(checkboxRef.current)` inside the ref callback, in Strict Mode it will run twice.

**Now let's wrap this code in `<StrictMode>` and add a ref callback:**

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
  const checkboxRef = useRef(null);
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(checkboxRef.current);
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    alert(checkboxRef.current.checked);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        <input type="checkbox" ref={checkboxRef} />
        I agree
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

</Sandpack>

Although the code above does not use a ref callback, if you were to add one, the function would be triggered twice, which would give you the opportunity to catch a mistake.

Strict Mode helps you make sure that you have the opportunity to clean up any side effects that happen inside of a ref callback.

---

### Fixing deprecation warnings enabled by Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

In the future, React may deprecate certain APIs and features. Strict Mode can help you prepare for these changes ahead of time by providing **deprecation warnings**.

For example, React may warn you if you are using a deprecated `string` ref:

```js
function MyComponent() {
  const inputRef = 'myInput'; // 🚩 Deprecated: string refs
  return <input ref={inputRef} />;
}
```

If you see a deprecation warning, consult the warning message and the React documentation for how to fix the issue. This typically involves updating your code to use a non-deprecated version of the API.

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

  return <h1>Bem-vindo ao chat {roomId}!</h1>;
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
  // Uma implementação real realmente conectaria ao servidor
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

Você notará que o número de conexões abertas está sempre crescendo. Em um aplicativo real, isso causaria problemas de desempenho e de rede. O problema é que [seu Effect está faltando uma função de limpeza:](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)

```js {4}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
```

Agora que seu Effect "limpa" após si mesmo e destrói as conexões desatualizadas, o vazamento está resolvido. No entanto, observe que o problema não se tornou visível até que você adicionou mais recursos (a caixa de seleção).

**No exemplo original, o erro não era óbvio. Agora vamos encapsular o código original (com bugs) em `<StrictMode>`:**

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
  // Uma implementação real realmente conectaria ao servidor
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

**Com o Strict Mode, você vê imediatamente que há um problema** (o número de conexões ativas salta para 2). O Strict Mode executa um ciclo extra de configuração + limpeza para cada Effect. Este Effect não possui lógica de limpeza, então ele cria uma conexão extra, mas não a destrói. Esta é uma dica de que falta uma função de limpeza.

O Strict Mode permite que você note esses erros no início do processo. Quando você corrige seu Effect, adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis bugs futuros na produção, como a caixa de seleção de antes:

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

  return <h1>Bem-vindo ao chat {roomId}!</h1>;
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
  // Uma implementação real realmente conectaria ao servidor
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

Observe como a contagem de conexões ativas no console não continua crescendo.

Sem o Strict Mode, era fácil perder que o seu Effect precisava de limpeza. Ao executar *configuração → limpeza → configuração* em vez de *configuração* para seu Effect no desenvolvimento, o Strict Mode tornou a lógica de limpeza ausente mais perceptível.

[Leia mais sobre a implementação da limpeza do Effect.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---
### Corrigindo erros encontrados pela execução repetida de callbacks de ref no desenvolvimento {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

O Strict Mode também pode ajudar a encontrar erros em [refs de callback.](/learn/manipulating-the-dom-with-refs)

Cada callback `ref` tem algum código de configuração e pode ter algum código de limpeza. Normalmente, o React chama a configuração quando o elemento é *criado* (é adicionado ao DOM) e chama a limpeza quando o elemento é *removido* (é removido do DOM).

Quando o Strict Mode está ativado, o React também executará **um ciclo extra de configuração + limpeza no desenvolvimento para cada `ref` de callback.** Isso pode parecer surpreendente, mas ajuda a revelar erros sutis que são difíceis de detectar manualmente.

Considere este exemplo, que permite selecionar um animal e rolar até um deles. Observe que, ao mudar de "Cats" para "Dogs", os logs do console mostram que o número de animais na lista continua crescendo e os botões "Scroll to" param de funcionar:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ❌ Não usando StrictMode.
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
                  console.log(`✅ Adicionando animal ao mapa. Animais totais: ${list.length}`);
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

**Este é um erro de produção!** Como o ref callback não remove animais da lista na limpeza, a lista de animais continua crescendo. Este é um vazamento de memória que pode causar problemas de desempenho em um aplicativo real e interrompe o comportamento do aplicativo.

O problema é que o callback ref não limpa depois de si mesmo:

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

Agora vamos encapsular o código original (com bugs) em `<StrictMode>`:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ Usando StrictMode.
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
                  console.log(`✅ Adicionando animal ao mapa. Animais totais: ${list.length}`);
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

**Com o Strict Mode, você vê imediatamente que há um problema**. O Strict Mode executa um ciclo extra de configuração + limpeza para cada ref de callback. Este callback ref não tem lógica de limpeza, então ele adiciona refs, mas não as remove. Esta é uma dica de que falta uma função de limpeza.

O Strict Mode permite que você encontre antecipadamente erros em refs de callback. Quando você corrige seu callback adicionando uma função de limpeza no Strict Mode, você *também* corrige muitos possíveis bugs futuros de produção, como o erro "Scroll to" de antes:

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ Usando StrictMode.
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
                  console.log(`✅ Adicionando animal ao mapa. Animais totais: ${list.length}`);
                  if (list.length > 10) {
                    console.log('❌ Muitos animais na lista!');
                  }
                  return () => {
                    list.splice(list.indexOf(item));
                    console.log(`❌ Removendo animal do mapa. Animais totais: ${itemsRef.current.length}`);
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

Agora, na montagem inicial no StrictMode, os callbacks ref são todos configurados, limpos e configurados novamente:

```
...
✅ Adicionando animal ao mapa. Animais totais: 10
...
❌ Removendo animal do mapa. Animais totais: 0
...
✅ Adicionando animal ao mapa. Animais totais: 10
```

**Isso é esperado.** O Strict Mode confirma que os callbacks ref são limpos corretamente, para que o tamanho nunca cresça acima da quantidade esperada. Após a correção, não há vazamentos de memória e todos os recursos funcionam como o esperado.

Sem o Strict Mode, era fácil perder o erro até que você clicasse no aplicativo para notar recursos quebrados. O Strict Mode fez os erros aparecerem imediatamente, antes de você empurrá-los para a produção.

---
### Corrigindo avisos de descontinuação habilitados pelo Strict Mode {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

O React avisa se algum componente em qualquer lugar dentro de uma árvore `<StrictMode>` usa uma dessas APIs descontinuadas:

* Métodos de ciclo de vida de classe `UNSAFE_` como [`UNSAFE_componentWillMount`](/reference/react/Component#unsafe_componentwillmount). [Veja as alternativas.](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#migrating-from-legacy-lifecycles)

Essas APIs são usadas principalmente em [componentes de classe](/reference/react/Component) mais antigos, portanto, raramente aparecem em aplicativos modernos.
