---
title: prerenderToNodeStream
---

<Intro>

`prerenderToNodeStream` renderiza uma árvore React para uma string HTML estática usando um [Node.js Stream.](https://nodejs.org/api/stream.html).

```js
const {prelude} = await prerenderToNodeStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API é específica do Node.js. Ambientes com [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) como Deno e runtimes de borda modernos, devem usar [`prerender`](/reference/react-dom/static/prerender) em vez disso.

</Note>

---

## Referência {/*reference*/}

### `prerenderToNodeStream(reactNode, options?)` {/*prerender*/}

Chame `prerenderToNodeStream` para renderizar seu app para HTML estático.

```js
import { prerenderToNodeStream } from 'react-dom/static';

// A sintaxe do manipulador de rota depende da sua framework de backend
app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js'],
  });

  response.setHeader('Content-Type', 'text/plain');
  prelude.pipe(response);
});
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você deseja renderizar para HTML. Por exemplo, um nó JSX como `<App />`. Espera-se que ele represente o documento inteiro, então o componente App deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de geração estática.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` embutida.
  * **opcional** `bootstrapScripts`: Uma array de URLs de string para as tags `<script>` a serem emitidas na página. Use isso para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omita se você não quiser executar React no cliente.
  * **optional** `bootstrapModules`: Similar a `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar vários roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **opcional** `namespaceURI`: Uma string com o [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) raiz para o stream. O padrão é HTML regular. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `onError`: Um callback que é disparado sempre que há um erro de servidor, seja [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isso só chama `console.error`. Se você substituí-lo para [registrar relatórios de falha,](#logging-crashes-on-the-server) certifique-se de que você ainda chama `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **opcional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **opcional** `signal`: Um [sinal de interrupção](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) que permite que você [interrompa a renderização do servidor](#aborting-server-rendering) e renderize o restante no cliente.

#### Retorna {/*returns*/}

`prerenderToNodeStream` retorna uma Promise:
- Se a renderização for bem-sucedida, a Promise será resolvida para um objeto contendo:
  - `prelude`: um [Node.js Stream.](https://nodejs.org/api/stream.html) de HTML. Você pode usar este stream para enviar uma resposta em chunks, ou você pode ler o stream inteiro em uma string.
- Se a renderização falhar, a Promise será rejeitada. [Use isso para enviar um shell de fallback.](#recovering-from-errors-inside-the-shell)

<Note>

### Quando devo usar `prerenderToNodeStream`? {/*when-to-use-prerender*/}

A API estática `prerenderToNodeStream` é usada para geração estática do lado do servidor (SSG). Diferente de `renderToString`, `prerenderToNodeStream` espera que todos os dados sejam carregados antes de resolver. Isso o torna adequado para gerar HTML estático para uma página inteira, incluindo dados que precisam ser buscados usando Suspense. Para transmitir conteúdo conforme ele carrega, use uma API de renderização do lado do servidor (SSR) de streaming como [renderToReadableStream](/reference/react-dom/server/renderToReadableStream).

</Note>

---

## Uso {/*usage*/}

### Renderizando uma árvore React para um stream de HTML estático {/*rendering-a-react-tree-to-a-stream-of-static-html*/}

Chame `prerenderToNodeStream` para renderizar sua árvore React para HTML estático em um [Node.js Stream.](https://nodejs.org/api/stream.html):

```js [[1, 5, "<App />"], [2, 6, "['/main.js']"]]
import { prerenderToNodeStream } from 'react-dom/static';

// A sintaxe do manipulador de rota depende da sua framework de backend
app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js'],
  });
  
  response.setHeader('Content-Type', 'text/plain');
  prelude.pipe(response);
});
```

Junto com o <CodeStep step={1}>componente raiz</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos de `<script>` de bootstrap</CodeStep>. Seu componente raiz deve retornar **o documento inteiro, incluindo a tag `<html>` raiz.**

Por exemplo, pode ser assim:

```js [[1, 1, "App"]]
export default function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

React irá injetar o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas <CodeStep step={2}>tags `<script>` de bootstrap</CodeStep> no stream HTML resultante:

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... HTML dos seus componentes ... -->
</html>
<script src="/main.js" async=""></script>
```

No cliente, seu script de bootstrap deve [hidratar o `document` inteiro com uma chamada para `hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

Isso irá anexar os manipuladores de eventos ao HTML estático gerado pelo servidor e torná-lo interativo.

<DeepDive>

#### Lendo os caminhos dos assets CSS e JS da saída da build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

As URLs finais dos assets (como arquivos JavaScript e CSS) são frequentemente hash após a build. Por exemplo, em vez de `styles.css` você pode acabar com `styles.123456.css`. O hash de nomes de arquivos de assets estáticos garante que cada build distinto do mesmo asset terá um nome de arquivo diferente. Isso é útil porque permite que você habilite com segurança o cache de longo prazo para assets estáticos: um arquivo com um determinado nome nunca alteraria o conteúdo.

No entanto, se você não souber as URLs dos assets até depois da build, não haverá como colocá-las no código-fonte. Por exemplo, hardcodar `"/styles.css"` no JSX como antes não funcionaria. Para mantê-los fora do seu código-fonte, seu componente raiz pode ler os nomes de arquivos reais de um mapa passado como uma prop:

```js {1,6}
export default function App({ assetMap }) {
  return (
    <html>
      <head>
        <title>My app</title>
        <link rel="stylesheet" href={assetMap['styles.css']}></link>
      </head>
      ...
    </html>
  );
}
```

No servidor, renderize `<App assetMap={assetMap} />` e passe seu `assetMap` com as URLs dos assets:

```js {1-5,8,9}
// Você precisaria obter este JSON de sua ferramenta de build, por exemplo, lê-lo da saída da build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: [assetMap['/main.js']]
  });

  response.setHeader('Content-Type', 'text/html');
  prelude.pipe(response);
});
```

Como seu servidor agora está renderizando `<App assetMap={assetMap} />`, você precisa renderizá-lo com `assetMap` também no cliente para evitar erros de hidratação. Você pode serializar e passar `assetMap` para o cliente assim:

```js {9-10}
// Você precisaria obter este JSON de sua ferramenta de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    // Cuidado: É seguro stringify() isso porque esses dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });

  response.setHeader('Content-Type', 'text/html');
  prelude.pipe(response);
});
```

No exemplo acima, a opção `bootstrapScriptContent` adiciona uma tag `<script>` embutida extra que define a variável global `window.assetMap` no cliente. Isso permite que o código do cliente leia o mesmo `assetMap`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

Tanto o cliente quanto o servidor renderizam `App` com a mesma prop `assetMap`, então não há erros de hidratação.

</DeepDive>

---

### Renderizando uma árvore React para uma string de HTML estático {/*rendering-a-react-tree-to-a-string-of-static-html*/}

Chame `prerenderToNodeStream` para renderizar seu app para uma string HTML estática:

```js
import { prerenderToNodeStream } from 'react-dom/static';

async function renderToString() {
  const {prelude} = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  
  return new Promise((resolve, reject) => {
    let data = '';
    prelude.on('data', chunk => {
      data += chunk;
    });
    prelude.on('end', () => resolve(data));
    prelude.on('error', reject);
  });
}
```

Isso irá produzir a saída HTML inicial não interativa de seus componentes React. No cliente, você precisará chamar [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para *hidratar* o HTML gerado pelo servidor e torná-lo interativo.

---

### Esperando todos os dados carregarem {/*waiting-for-all-data-to-load*/}

`prerenderToNodeStream` espera que todos os dados carreguem antes de finalizar a geração de HTML estático e resolver. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos, e uma lista de postagens:

```js
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Imagine que `<Posts />` precise carregar alguns dados, o que leva algum tempo. Idealmente, você gostaria de esperar que as postagens terminassem para que fossem incluídas no HTML. Para fazer isso, você pode usar Suspense para suspender nos dados, e `prerenderToNodeStream` esperará que o conteúdo suspenso termine antes de resolver para o HTML estático.

<Note>

**Somente as fontes de dados habilitadas para Suspense ativarão o componente Suspense.** Elas incluem:

- Busca de dados com frameworks habilitados para Suspense como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Carregamento lento de código de componente com [`lazy`](/reference/react/lazy)
- Leitura do valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são buscados dentro de um Effect ou manipulador de eventos.

A maneira exata de carregar dados no componente `Posts` acima depende do seu framework. Se você usar um framework habilitado para Suspense, encontrará os detalhes em sua documentação de busca de dados.

A busca de dados habilitada para Suspense sem o uso de um framework opinativo ainda não é suportada. Os requisitos para implementar uma fonte de dados habilitada para Suspense são instáveis ​​e não documentados. Uma API oficial para integrar fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

## Solução de problemas {/*troubleshooting*/}

### Meu stream não começa até que todo o app seja renderizado {/*my-stream-doesnt-start-until-the-entire-app-is-rendered*/}

A resposta `prerenderToNodeStream` aguarda até que todo o aplicativo termine de renderizar, incluindo esperar que todos os limites de Suspense sejam resolvidos, antes de resolver. Ele foi projetado para geração de site estático (SSG) com antecedência e não oferece suporte ao streaming de mais conteúdo conforme ele carrega.

Para transmitir conteúdo conforme ele carrega, use uma API de renderização de servidor de streaming como [renderToPipeableStream](/reference/react-dom/server/renderToPipeableStream).