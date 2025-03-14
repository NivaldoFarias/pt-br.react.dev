---
title: Componentes de Servidor
---

<RSC>

Componentes de Servidor são para uso em [Componentes de Servidor React](/learn/start-a-new-react-project#bleeding-edge-react-frameworks).

</RSC>

<Intro>

Componentes de Servidor são um novo tipo de Componente que renderiza antecipadamente, antes de empacotar, em um ambiente separado do seu aplicativo cliente ou servidor SSR.

</Intro>

Este ambiente separado é o "servidor" em Componentes de Servidor React. Componentes de Servidor podem ser executados uma vez no tempo de construção no seu servidor CI, ou podem ser executados para cada requisição usando um servidor web.

<InlineToc />

<Note>

#### Como crio suporte para Componentes de Servidor? {/*how-do-i-build-support-for-server-components*/}

Embora Componentes de Servidor React no React 19 sejam estáveis e não quebrem entre as versões secundárias, as APIs subjacentes usadas para implementar um empacotador ou framework de Componentes de Servidor React não seguem semver e podem quebrar entre menores no React 19.x.

Para oferecer suporte a Componentes de Servidor React como um empacotador ou framework, recomendamos fixar em uma versão específica do React ou usar a versão Canary. Continuaremos trabalhando com empacotadores e frameworks para estabilizar as APIs usadas para implementar Componentes de Servidor React no futuro.

</Note>

### Componentes de Servidor sem um Servidor {/*server-components-without-a-server*/}
Componentes de Servidor podem ser executados no tempo de construção para ler do sistema de arquivos ou buscar conteúdo estático, então um servidor web não é necessário. Por exemplo, você pode querer ler dados estáticos de um sistema de gerenciamento de conteúdo.

Sem Componentes de Servidor, é comum buscar dados estáticos no cliente com um Effect:
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

Este padrão significa que os usuários precisam baixar e analisar 75K (gzipped) de bibliotecas adicionais e esperar por uma segunda requisição para buscar os dados após o carregamento da página, apenas para renderizar conteúdo estático que não mudará durante o tempo de vida da página.

Com Componentes de Servidor, você pode renderizar esses componentes uma vez no tempo de construção:

```js
import marked from 'marked'; // Not included in bundle
import sanitizeHtml from 'sanitize-html'; // Not included in bundle

async function Page({page}) {
  // NOTE: loads *during* render, when the app is built.
  const content = await file.readFile(`${page}.md`);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

A saída renderizada pode então ser renderizada do lado do servidor (SSR) para HTML e carregada para uma CDN. Quando o aplicativo carrega, o cliente não verá o componente `Page` original, ou as bibliotecas caras para renderizar o markdown. O cliente só verá a saída renderizada:

```js
<div><!-- html for markdown --></div>
```

Isto significa que o conteúdo é visível durante o primeiro carregamento da página, e o bundle não inclui as bibliotecas caras necessárias para renderizar o conteúdo estático.

<Note>

Você pode notar que o Componente de Servidor acima é uma função assíncrona:

```js
async function Page({page}) {
  //...
}
```

Componentes Assíncronos são um novo recurso de Componentes de Servidor que permitem que você use `await` em renderização.

Veja [Componentes assíncronos com Componentes de Servidor](#async-components-with-server-components) abaixo.

</Note>

### Componentes de Servidor com um Servidor {/*server-components-with-a-server*/}
Componentes de Servidor também podem ser executados em um servidor web durante uma requisição de uma página, permitindo que você acesse sua camada de dados sem ter que construir uma API. Eles são renderizados antes que seu aplicativo seja empacotado, e podem passar dados e JSX como props para Componentes Cliente.

Sem Componentes de Servidor, é comum buscar dados dinâmicos no cliente em um Effect:

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

Com Componentes de Servidor, você pode ler os dados e renderizá-los no componente:

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

O empacotador então combina os dados, Componentes de Servidor renderizados e Componentes Cliente dinâmicos em um bundle. Opcionalmente, esse bundle pode então ser renderizado do lado do servidor (SSR) para criar o HTML inicial para a página. Quando a página carrega, o navegador não vê os componentes `Note` e `Author` originais; apenas a saída renderizada é enviada ao cliente:

```js
<div>
  <span>By: The React Team</span>
  <p>React 19 is...</p>
</div>
```

Componentes de Servidor podem ser tornados dinâmicos ao refetchá-los de um servidor, onde eles podem acessar os dados e renderizar novamente. Essa nova arquitetura de aplicativo combina o modelo mental simples "request/response" de aplicativos de várias páginas centrados no servidor com a interatividade perfeita de aplicativos de página única (Single-Page Apps) centrados no cliente, dando a você o melhor dos dois mundos.

### Adicionando interatividade aos Componentes de Servidor {/*adding-interactivity-to-server-components*/}

Componentes de Servidor não são enviados para o navegador, então eles não podem usar APIs interativas como `useState`. Para adicionar interatividade aos Componentes de Servidor, você pode compô-los com Componentes Cliente usando a diretiva `"use client"`.

<Note>

#### Não existe diretiva para Componentes de Servidor. {/*there-is-no-directive-for-server-components*/}

Um mal-entendido comum é que os Componentes de Servidor são denotados por `"use server"`, mas não existe diretiva para Componentes de Servidor. A diretiva `"use server"` é usada para Funções de Servidor.

Para mais informações, veja os documentos para [Diretivas](/reference/rsc/directives).

</Note>


No seguinte exemplo, o Componente de Servidor `Notes` importa um Componente Cliente `Expandable` que usa estado para alternar seu estado `expanded`:
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

Isso funciona primeiramente renderizando `Notes` como um Componente de Servidor, e então instruindo o empacotador a criar um bundle para o Componente Cliente `Expandable`. No navegador, os Componentes Cliente verão a saída dos Componentes de Servidor passados como props:

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

### Componentes assíncronos com Componentes de Servidor {/*async-components-with-server-components*/}

Componentes de Servidor introduzem uma nova forma de escrever Componentes usando async/await. Quando você usa `await` em um componente assíncrono, o React irá suspender e esperar que a promise seja resolvida antes de retomar a renderização. Isso funciona através das fronteiras servidor/cliente com suporte de streaming para Suspense.

Você pode até mesmo criar uma promise no servidor e aguardá-la no cliente:

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
      <Suspense fallback={<p>Carregando Comentários...</p>}>
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

O conteúdo `note` é dados importantes para a página renderizar, então nós o `await` no servidor. Os comentários estão abaixo da dobra e com menor prioridade, então nós iniciamos a promise no servidor e aguardamos por ela no cliente com a API `use`. Isso irá Suspender no cliente, sem bloquear o conteúdo `note` de renderizar.

Já que componentes assíncronos [não são suportados no cliente](#why-cant-i-use-async-components-on-the-client), nós aguardamos a promise com `use`.