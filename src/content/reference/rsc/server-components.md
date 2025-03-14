---
title: Server Components
---

<RSC>

Server Components são para uso em [React Server Components](/learn/start-a-new-react-project#bleeding-edge-react-frameworks).

</RSC>

<Intro>

Server Components são um novo tipo de Componente que renderiza com antecedência, antes de empacotar, em um ambiente separado do seu aplicativo cliente ou servidor SSR.

</Intro>

Este ambiente separado é o "servidor" em React Server Components. Server Components podem rodar uma vez no tempo de construção no seu servidor CI, ou podem ser executados para cada requisição usando um servidor web.

<InlineToc />

<Note>

#### Como construo suporte para Server Components? {/*how-do-i-build-support-for-server-components*/}

Enquanto os React Server Components no React 19 são estáveis e não quebrarão entre versões secundárias, as APIs subjacentes usadas para implementar um empacotador ou framework React Server Components não seguem semver e podem quebrar entre as versões secundárias no React 19.x.

Para oferecer suporte a React Server Components como um empacotador ou framework, recomendamos fixar em uma versão específica do React ou usar o Canary release. Continuaremos trabalhando com empacotadores e frameworks para estabilizar as APIs usadas para implementar React Server Components no futuro.

</Note>

### Server Components sem um Servidor {/*server-components-without-a-server*/}
Server components podem rodar no tempo de construção para ler do sistema de arquivos ou buscar conteúdo estático, então um servidor web não é necessário. Por exemplo, você pode querer ler dados estáticos de um sistema de gerenciamento de conteúdo.

Sem Server Components, é comum buscar dados estáticos no cliente com um Effect:
```js
// bundle.js
import marked from 'marked'; // 35.9K (11.2K gzipped)
import sanitizeHtml from 'sanitize-html'; // 206K (63.3K gzipped)

function Page({page}) {
  const [content, setContent] = useState('');
  // NOTE: loads *after* first page render.
  useEffect(() => {
    fetch(`/api/content/${page}`).then((data) => {
      setContent(data.content);
    });
  }, [page]);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```
```js
// api.js
app.get(`/api/content/:page`, async (req, res) => {
  const page = req.params.page;
  const content = await file.readFile(`${page}.md`);
  res.send({content});
});
```

Este padrão significa que os usuários precisam baixar e analisar 75K (gzipped) de bibliotecas adicionais, e esperar por uma segunda requisição para buscar os dados após a página carregar, apenas para renderizar o conteúdo estático que não mudará durante a vida útil da página.

Com Server Components, você pode renderizar esses componentes uma vez no tempo de construção:

```js
import marked from 'marked'; // Not included in bundle
import sanitizeHtml from 'sanitize-html'; // Not included in bundle

async function Page({page}) {
  // NOTE: loads *during* render, when the app is built.
  const content = await file.readFile(`${page}.md`);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

A saída renderizada pode então ser renderizada no lado do servidor (SSR) para HTML e carregada em uma CDN. Quando o aplicativo carrega, o cliente não verá o componente `Page` original, nem as bibliotecas dispendiosas para renderizar o markdown. O cliente só verá a saída renderizada:

```js
<div><!-- html for markdown --></div>
```

Isso significa que o conteúdo é visível durante o carregamento da primeira página, e o bundle não inclui as bibliotecas caras necessárias para renderizar o conteúdo estático.

<Note>

Você pode notar que o Server Component acima é uma função async:

```js
async function Page({page}) {
  //...
}
```

Async Components são um novo recurso dos Server Components que permitem que você `await` no render.

Veja [Async components with Server Components](#async-components-with-server-components) abaixo.

</Note>

### Server Components com um Servidor {/*server-components-with-a-server*/}
Server Components também podem rodar em um servidor web durante uma requisição para uma página, permitindo que você acesse sua camada de dados sem ter que construir uma API. Eles são renderizados antes que seu aplicativo seja empacotado, e podem passar dados e JSX como props para Client Components.

Sem Server Components, é comum buscar dados dinâmicos no cliente em um Effect:

```js
// bundle.js
function Note({id}) {
  const [note, setNote] = useState('');
  // NOTE: loads *after* first render.
  useEffect(() => {
    fetch(`/api/notes/${id}`).then(data => {
      setNote(data.note);
    });
  }, [id]);
  
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

function Author({id}) {
  const [author, setAuthor] = useState('');
  // NOTE: loads *after* Note renders.
  // Causing an expensive client-server waterfall.
  useEffect(() => {
    fetch(`/api/authors/${id}`).then(data => {
      setAuthor(data.author);
    });
  }, [id]);

  return <span>By: {author.name}</span>;
}
```
```js
// api
import db from './database';

app.get(`/api/notes/:id`, async (req, res) => {
  const note = await db.notes.get(id);
  res.send({note});
});

app.get(`/api/authors/:id`, async (req, res) => {
  const author = await db.authors.get(id);
  res.send({author});
});
```

Com Server Components, você pode ler os dados e renderizá-los no componente:

```js
import db from './database';

async function Note({id}) {
  // NOTE: loads *during* render.
  const note = await db.notes.get(id);
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

async function Author({id}) {
  // NOTE: loads *after* Note,
  // but is fast if data is co-located.
  const author = await db.authors.get(id);
  return <span>By: {author.name}</span>;
}
```

O empacotador então combina os dados, Server Components renderizados e Client Components dinâmicos em um bundle. Opcionalmente, esse bundle pode então ser renderizado no lado do servidor (SSR) para criar o HTML inicial para a página. Quando a página carrega, o navegador não vê os componentes `Note` e `Author` originais; apenas a saída renderizada é enviada para o cliente:

```js
<div>
  <span>By: The React Team</span>
  <p>React 19 is...</p>
</div>
```

Server Components podem ser tornados dinâmicos, re-buscando-os de um servidor, onde eles podem acessar os dados e renderizar novamente. Essa nova arquitetura de aplicativo combina o simples modelo mental de “request/response” de Multi-Page Apps centrados no servidor com a interatividade perfeita de Single-Page Apps centradas no cliente, dando a você o melhor dos dois mundos.

### Adicionando interatividade a Server Components {/*adding-interactivity-to-server-components*/}

Server Components não são enviados para o navegador, então eles não podem usar APIs interativas como `useState`. Para adicionar interatividade a Server Components, você pode compô-los com Client Component usando a diretiva `"use client"`.

<Note>

#### Não há diretiva para Server Components. {/*there-is-no-directive-for-server-components*/}

Um mal-entendido comum é que Server Components são denotados por `"use server"`, mas não há diretiva para Server Components. A diretiva `"use server"` é usada para Server Functions.

Para mais informações, consulte a documentação para [Diretivas](/reference/rsc/directives).

</Note>


No exemplo a seguir, o Server Component `Notes` importa um Client Component `Expandable` que usa state para alternar seu estado `expanded`:
```js
// Server Component
import Expandable from './Expandable';

async function Notes() {
  const notes = await db.notes.getAll();
  return (
    <div>
      {notes.map(note => (
        <Expandable key={note.id}>
          <p note={note} />
        </Expandable>
      ))}
    </div>
  )
}
```
```js
// Client Component
"use client"

export default function Expandable({children}) {
  const [expanded, setExpanded] = useState(false);
  return (
    <div>
      <button
        onClick={() => setExpanded(!expanded)}
      >
        Toggle
      </button>
      {expanded && children}
    </div>
  )
}
```

Isso funciona renderizando primeiro `Notes` como um Server Component e, em seguida, instruindo o empacotador a criar um bundle para o Client Component `Expandable`. No navegador, os Client Components verão a saída dos Server Components passados como props:

```js
<head>
  <!-- the bundle for Client Components -->
  <script src="bundle.js" />
</head>
<body>
  <div>
    <Expandable key={1}>
      <p>this is the first note</p>
    </Expandable>
    <Expandable key={2}>
      <p>this is the second note</p>
    </Expandable>
    <!--...-->
  </div> 
</body>
```

### Async components com Server Components {/*async-components-with-server-components*/}

Server Components introduzem uma nova forma de escrever Components usando async/await. Quando você `await` em um componente async, o React irá suspender e esperar a promise ser resolvida antes de retomar a renderização. Isso funciona em limites de servidor/cliente com suporte a streaming para Suspense.

Você pode até mesmo criar uma promise no servidor, e esperar por ela no cliente:

```js
// Server Component
import db from './database';

async function Page({id}) {
  // Will suspend the Server Component.
  const note = await db.notes.get(id);
  
  // NOTE: not awaited, will start here and await on the client. 
  const commentsPromise = db.comments.get(note.id);
  return (
    <div>
      {note}
      <Suspense fallback={<p>Loading Comments...</p>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </div>
  );
}
```

```js
// Client Component
"use client";
import {use} from 'react';

function Comments({commentsPromise}) {
  // NOTE: this will resume the promise from the server.
  // It will suspend until the data is available.
  const comments = use(commentsPromise);
  return comments.map(commment => <p>{comment}</p>);
}
```

O conteúdo `note` é importante para a página renderizar, então nós o `await` no servidor. Os comentários estão abaixo da dobra e com prioridade inferior, então iniciamos a promise no servidor e esperamos por ela no cliente com a API `use`. Isso irá suspender no cliente, sem bloquear o conteúdo `note` de ser renderizado.

Como componentes async [não são suportados no cliente](#why-cant-i-use-async-components-on-the-client), nós esperamos pela promise com `use`.
```