---
title: <Activity>
version: canary
---

<Canary>

**A API `<Activity />` está atualmente disponível apenas nos canais Canary e Experimental do React.** 

[Saiba mais sobre os canais de lançamento do React aqui.](/community/versioning-policy#all-release-channels)

</Canary>

<Intro>

`<Activity>` permite ocultar e restaurar a UI e o estado interno de seus filhos.

```js
<Activity mode={visibility}>
  <Sidebar />
</Activity>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<Activity>` {/*activity*/}

Você pode usar Activity para ocultar parte de sua aplicação:

```js [[1, 1, "\\"hidden\\""], [2, 2, "<Sidebar />"], [3, 1, "\\"visible\\""]]
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```

Quando um limite de Activity é <CodeStep step={1}>oculto</CodeStep>, o React ocultará visualmente <CodeStep step={2}>seus filhos</CodeStep> usando a propriedade CSS `display: "none"`. Ele também destruirá seus Effects, limpando quaisquer assinaturas ativas.

Enquanto ocultos, os filhos ainda são renderizados em resposta a novas props, embora com prioridade menor do que o restante do conteúdo.

Quando o limite se torna <CodeStep step={3}>visível</CodeStep> novamente, o React revelará os filhos com seu estado anterior restaurado e recriará seus Effects.

Dessa forma, Activity pode ser considerado um mecanismo para renderizar "atividade em segundo plano". Em vez de descartar completamente o conteúdo que provavelmente se tornará visível novamente, você pode usar Activity para manter e restaurar a UI e o estado interno desse conteúdo, garantindo que seu conteúdo oculto não tenha efeitos colaterais indesejados.

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

* `children`: A UI que você pretende mostrar e ocultar.
* `mode`: Um valor de string `'visible'` ou `'hidden'`. Se omitido, o padrão é `'visible'`.

#### Ressalvas {/*caveats*/}

- Se uma Activity for renderizada dentro de uma [ViewTransition](/reference/react/ViewTransition) e se tornar visível como resultado de uma atualização causada por [startTransition](/reference/react/startTransition), ela ativará a animação `enter` da ViewTransition. Se se tornar oculta, ativará sua animação `exit`.

---

## Uso {/*usage*/}

### Restaurando o estado de componentes ocultos {/*restoring-the-state-of-hidden-components*/}

No React, quando você deseja mostrar ou ocultar um componente condicionalmente, você normalmente o monta ou desmonta com base nessa condição:

```jsx
{isShowingSidebar && (
  <Sidebar />
)}
```

Mas desmontar um componente destrói seu estado interno, o que nem sempre é o que você deseja.

Quando você oculta um componente usando um limite de Activity em vez disso, o React "salvará" seu estado para mais tarde:

```jsx
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```

Isso permite ocultar e, em seguida, restaurar componentes no estado em que estavam anteriormente.

O exemplo a seguir tem uma barra lateral com uma seção expansível. Você pode pressionar "Overview" para revelar os três subitens abaixo dela. A área principal do aplicativo também tem um botão que oculta e mostra a barra lateral.

Tente expandir a seção Overview e, em seguida, alternar a barra lateral para fechada e depois aberta:

<Sandpack>

```js src/App.js active
import { useState } from 'react';
import Sidebar from './Sidebar.js';

export default function App() {
  const [isShowingSidebar, setIsShowingSidebar] = useState(true);

  return (
    <>
      {isShowingSidebar && (
        <Sidebar />
      )}

      <main>
        <button onClick={() => setIsShowingSidebar(!isShowingSidebar)}>
          Toggle sidebar
        </button>
        <h1>Main content</h1>
      </main>
    </>
  );
}
```

```js src/Sidebar.js
import { useState } from 'react';

export default function Sidebar() {
  const [isExpanded, setIsExpanded] = useState(false)
  
  return (
    <nav>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        Overview
        <span className={`indicator ${isExpanded ? 'down' : 'right'}`}>
          &#9650;
        </span>
      </button>

      {isExpanded && (
        <ul>
          <li>Section 1</li>
          <li>Section 2</li>
          <li>Section 3</li>
        </ul>
      )}
    </nav>
  );
}
```

```css
body { height: 275px; margin: 0; }
#root {
  display: flex;
  gap: 10px;
  height: 100%;
}
nav {
  padding: 10px;
  background: #eee;
  font-size: 14px;
  height: 100%;
}
main {
  padding: 10px;
}
p {
  margin: 0;
}
h1 {
  margin-top: 10px;
}
.indicator {
  margin-left: 4px;
  display: inline-block;
  rotate: 90deg;
}
.indicator.down {
  rotate: 180deg;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
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

A seção Overview sempre começa recolhida. Como desmontamos a barra lateral quando `isShowingSidebar` muda para `false`, todo o seu estado interno é perdido.

Este é um caso de uso perfeito para Activity. Podemos preservar o estado interno de nossa barra lateral, mesmo ao ocultá-la visualmente.

Vamos substituir a renderização condicional de nossa barra lateral por um limite de Activity:

```jsx {7,9}
// Antes
{isShowingSidebar && (
  <Sidebar />
)}

// Depois
<Activity mode={isShowingSidebar ? 'visible' : 'hidden'}>
  <Sidebar />
</Activity>
```

e confira o novo comportamento:

<Sandpack>

```js src/App.js active
import { unstable_Activity as Activity, useState } from 'react';
import Sidebar from './Sidebar.js';

export default function App() {
  const [isShowingSidebar, setIsShowingSidebar] = useState(true);

  return (
    <>
      <Activity mode={isShowingSidebar ? 'visible' : 'hidden'}>
        <Sidebar />
      </Activity>

      <main>
        <button onClick={() => setIsShowingSidebar(!isShowingSidebar)}>
          Toggle sidebar
        </button>
        <h1>Main content</h1>
      </main>
    </>
  );
}
```

```js src/Sidebar.js
import { useState } from 'react';

export default function Sidebar() {
  const [isExpanded, setIsExpanded] = useState(false)
  
  return (
    <nav>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        Overview
        <span className={`indicator ${isExpanded ? 'down' : 'right'}`}>
          &#9650;
        </span>
      </button>

      {isExpanded && (
        <ul>
          <li>Section 1</li>
          <li>Section 2</li>
          <li>Section 3</li>
        </ul>
      )}
    </nav>
  );
}
```

```css
body { height: 275px; margin: 0; }
#root {
  display: flex;
  gap: 10px;
  height: 100%;
}
nav {
  padding: 10px;
  background: #eee;
  font-size: 14px;
  height: 100%;
}
main {
  padding: 10px;
}
p {
  margin: 0;
}
h1 {
  margin-top: 10px;
}
.indicator {
  margin-left: 4px;
  display: inline-block;
  rotate: 90deg;
}
.indicator.down {
  rotate: 180deg;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
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

O estado interno da nossa barra lateral agora é restaurado, sem alterações em sua implementação.

---

### Restaurando o DOM de componentes ocultos {/*restoring-the-dom-of-hidden-components*/}

Como os limites de Activity ocultam seus filhos usando `display: none`, o DOM de seus filhos também é preservado quando oculto. Isso os torna ideais para manter estado efêmero em partes da UI com as quais o usuário provavelmente interagirá novamente.

Neste exemplo, a aba Contato tem um `<textarea>` onde o usuário pode digitar uma mensagem. Se você digitar algo, mudar para a aba Home e depois voltar para a aba Contato, a mensagem rascunhada será perdida:

<Sandpack>

```js src/App.js 
import { useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Contact from './Contact.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('contact');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'contact'}
        onClick={() => setActiveTab('contact')}
      >
        Contact
      </TabButton>

      <hr />

      {activeTab === 'home' && <Home />}
      {activeTab === 'contact' && <Contact />}
    </>
  );
}
```

```js src/TabButton.js
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Contact.js active
export default function Contact() {
  return (
    <div>
      <p>Send me a message!</p>

      <textarea />

      <p>You can find me online here:</p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </div>
  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
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

Isso ocorre porque estamos desmontando completamente `Contact` em `App`. Quando a aba Contato é desmontada, o estado interno do elemento `<textarea>` é perdido.

Se usarmos um limite de Activity para mostrar e ocultar a aba ativa, poderemos preservar o estado do DOM de cada aba. Tente digitar texto e mudar de aba novamente, e você verá que a mensagem rascunhada não é mais redefinida:

<Sandpack>

```js src/App.js active
import { useState, unstable_Activity as Activity } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Contact from './Contact.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('contact');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'contact'}
        onClick={() => setActiveTab('contact')}
      >
        Contact
      </TabButton>

      <hr />

      <Activity mode={activeTab === 'home' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      <Activity mode={activeTab === 'contact' ? 'visible' : 'hidden'}>
        <Contact />
      </Activity>
    </>
  );
}
```

```js src/TabButton.js
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Contact.js 
export default function Contact() {
  return (
    <div>
      <p>Send me a message!</p>

      <textarea />

      <p>You can find me online here:</p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </div>
  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
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

Novamente, o limite de Activity nos permitiu preservar o estado interno da aba Contato sem alterar sua implementação.

---

### Pré-renderizando conteúdo que provavelmente se tornará visível {/*pre-rendering-content-thats-likely-to-become-visible*/}

Até agora, vimos como Activity pode ocultar algum conteúdo com o qual o usuário interagiu, sem descartar o estado efêmero desse conteúdo.

Mas os limites de Activity também podem ser usados para _preparar_ conteúdo que o usuário ainda não viu pela primeira vez:

```jsx [[1, 1, "\\"hidden\\""]]
<Activity mode="hidden">
  <SlowComponent />
</Activity>
```

Quando um limite de Activity está <CodeStep step={1}>oculto</CodeStep> durante sua renderização inicial, seus filhos não serão visíveis na página — mas eles _ainda serão renderizados_, embora com prioridade menor do que o conteúdo visível, e sem montar seus Effects.

Essa _pré-renderização_ permite que os filhos carreguem qualquer código ou dados de que precisem com antecedência, para que, mais tarde, quando o limite de Activity se tornar visível, os filhos possam aparecer mais rapidamente com tempos de carregamento reduzidos.

Vamos ver um exemplo.

Nesta demonstração, a aba Posts carrega alguns dados. Se você a pressionar, verá um fallback de Suspense exibido enquanto os dados estão sendo buscados:

<Sandpack>

```js src/App.js
import { useState, Suspense } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Posts from './Posts.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'posts'}