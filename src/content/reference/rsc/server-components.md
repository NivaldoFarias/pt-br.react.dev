---
title: Componentes de Servidor
---

<RSC>

Componentes de Servidor são para uso em [Componentes de Servidor React](/learn/start-a-new-react-project#bleeding-edge-react-frameworks).

</RSC>

<Intro>

Componentes de Servidor são um novo tipo de Componente que renderiza com antecedência, antes do empacotamento, em um ambiente separado do seu aplicativo cliente ou servidor SSR.

</Intro>

Este ambiente separado é o "servidor" nos Componentes de Servidor React. Componentes de Servidor podem ser executados uma vez no momento da compilação no seu servidor CI ou podem ser executados para cada solicitação usando um servidor web.

<InlineToc />

<Note>

#### Como posso criar suporte para Componentes de Servidor? {/*how-do-i-build-support-for-server-components*/}

Embora os Componentes de Servidor React no React 19 sejam estáveis e não se quebrem entre versões secundárias, as APIs subjacentes usadas para implementar um empacotador ou framework de Componentes de Servidor React não seguem semver e podem quebrar entre as versões secundárias no React 19.x.

Para dar suporte a Componentes de Servidor como um empacotador ou framework, recomendamos fixar em uma versão específica do React ou usar a versão Canary. Continuaremos trabalhando com empacotadores e frameworks para estabilizar as APIs usadas para implementar Componentes de Servidor React no futuro.

</Note>

### Componentes de Servidor sem um Servidor {/*server-components-without-a-server*/}

Componentes de Servidor podem ser executados no momento da compilação para ler no sistema de arquivos ou buscar conteúdo estático, portanto, um servidor web não é necessário. Por exemplo, você pode querer ler dados estáticos de um sistema de gerenciamento de conteúdo.

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

Este padrão significa que os usuários precisam baixar e analisar mais 75K (gzipped) de bibliotecas e aguardar uma segunda solicitação para buscar os dados após o carregamento da página, apenas para renderizar um conteúdo estático que não mudará durante todo o tempo de vida da página.

Com os Componentes de Servidor, você pode renderizar esses componentes uma vez no momento da compilação:

```js
import marked from 'marked'; // Not included in bundle
import sanitizeHtml from 'sanitize-html'; // Not included in bundle

async function Page({page}) {
  // NOTE: loads *during* render, when the app is built.
  const content = await file.readFile(`${page}.md`);
  
  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

A saída renderizada pode então ser renderizada no lado do servidor (SSR) em HTML e carregada em uma CDN. Quando o aplicativo carregar, o cliente não verá o componente `Page` original nem as bibliotecas caras para renderizar o markdown. O cliente só verá a saída renderizada:

```js
<div><!-- html for markdown --></div>
```

Isso significa que o conteúdo é visível durante o primeiro carregamento da página, e o bundle não inclui as bibliotecas caras necessárias para renderizar o conteúdo estático.

<Note>

Você pode notar que o Componente de Servidor acima é uma função assíncrona:

```js
async function Page({page}) {
  //...
}
```

Async Components são um novo recurso dos Componentes de Servidor que permite que você `await` na renderização.

Veja [Componentes assíncronos com Componentes de Servidor](#async-components-with-server-components) abaixo.

</Note>

### Componentes de Servidor com um Servidor {/*server-components-with-a-server*/}

Componentes de Servidor também podem ser executados em um servidor web durante uma solicitação por uma página, permitindo que você acesse sua camada de dados sem ter que criar uma API. Eles são renderizados antes que seu aplicativo seja empacotado e podem passar dados e JSX como props para Componentes Cliente.

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

Com os Componentes de Servidor, você pode ler os dados e renderizá-los no componente:

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

O empacotador, em seguida, combina os dados, os Componentes de Servidor renderizados e os Componentes Cliente dinâmicos em um bundle. Opcionalmente, esse bundle pode, então, ser renderizado no lado do servidor (SSR) para criar o HTML inicial para a página. Quando a página carrega, o navegador não vê os componentes `Note` e `Author` originais; apenas a saída renderizada é enviada para o cliente:

```js
<div>
  <span>By: The React Team</span>
  <p>React 19 is...</p>
</div>
```

Os Componentes de Servidor podem ser tornados dinâmicos, buscando-os novamente de um servidor, onde podem acessar os dados e renderizar novamente. Essa nova arquitetura de aplicativos combina o modelo mental simples de “request/response” dos Multi-Page Apps centrados no servidor com a interatividade perfeita dos Single-Page Apps centrados no cliente, oferecendo o melhor dos dois mundos.

### Adicionando interatividade aos Componentes de Servidor {/*adding-interactivity-to-server-components*/}

Componentes de Servidor não são enviados para o navegador, portanto, eles não podem usar APIs interativas como `useState`. Para adicionar interatividade aos Componentes de Servidor, você pode compô-los com um Componente Cliente usando a diretiva `"use client"`.

<Note>

#### Não existe diretiva para Componentes de Servidor. {/*there-is-no-directive-for-server-components*/}

Um mal-entendido comum é que os Componentes de Servidor são denotados por `"use server"`, mas não existe uma diretiva para Componentes de Servidor. A diretiva `"use server"` é usada para Funções de Servidor.

Para mais informações, consulte a documentação de [Diretivas](/reference/rsc/directives).

</Note>

No exemplo a seguir, o Componente de Servidor `Notes` importa um Componente Cliente `Expandable` que usa o estado para alternar seu estado `expanded`:
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

Isso funciona primeiro renderizando `Notes` como um Componente de Servidor e, em seguida, instruindo o empacotador a criar um bundle para o Componente Cliente `Expandable`. No navegador, os Componentes Cliente verão a saída dos Componentes de Servidor passados como props:

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

Componentes de Servidor introduzem uma nova maneira de escrever Componentes usando async/await. Quando você usa `await` em um componente assíncrono, o React irá suspender e aguardar a resolução da promise antes de retomar a renderização. Isso funciona em limites de servidor/cliente com suporte de streaming para Suspense.

Você pode até criar uma promise no servidor e aguardá-la no cliente:

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

O conteúdo da `note` é um dado importante para a página renderizar, então usamos `await` no servidor. Os comentários estão abaixo da dobra e têm prioridade mais baixa, então iniciamos a promise no servidor e a aguardamos no cliente com a API `use`. Isso irá Suspender no cliente, sem impedir que o conteúdo da `note` seja renderizado.

Como os componentes assíncronos [não são suportados no cliente](#why-cant-i-use-async-components-on-the-client), aguardamos a promise com `use`.
```