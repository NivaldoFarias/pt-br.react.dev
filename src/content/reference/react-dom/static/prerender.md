---
title: prerender
---

<Intro>

`prerender` renderiza uma árvore React para uma string HTML estática usando uma [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API).

```js
const {prelude} = await prerender(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API depende de [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) Para Node.js, use [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream) em vez disso.

</Note>

---

## Referência {/*reference*/}

### `prerender(reactNode, options?)` {/*prerender*/}

Chame `prerender` para renderizar seu app para HTML estático.

```js
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado no servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você deseja renderizar para HTML. Por exemplo, um nó JSX como `<App />`. Espera-se que ele represente  o documento inteiro, então o componente App deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de geração estática.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` embutida.
  * **opcional** `bootstrapScripts`: Uma array de URLs de string para as tags `<script>` a serem emitidas na página. Use isso para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omita-o se você não quiser executar o React no cliente.
  * **opcional** `bootstrapModules`: Como `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar várias roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **opcional** `namespaceURI`: Uma string com o [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) da root para o stream. O padrão é HTML normal. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `onError`: Um callback que dispara sempre que houver um erro no servidor, seja [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isso apenas chama `console.error`. Se você substituir isso para [registrar relatórios de falhas,](#logging-crashes-on-the-server) certifique-se de ainda chamar `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **opcional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **opcional** `signal`: Um [sinal de abortar](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) que permite que você [aborte a renderização no servidor](#aborting-server-rendering) e renderize o restante no cliente.

#### Retorna {/*returns*/}

`prerender` retorna uma Promise:
- Se a renderização for bem-sucedida, a Promise resolverá para um objeto contendo:
  - `prelude`: um [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) de HTML. Você pode usar esta stream para enviar uma resposta em chunks, ou você pode ler a stream inteira em uma string.
- Se a renderização falhar, a Promise será rejeitada. [Use isso para gerar um shell de fallback.](#recovering-from-errors-inside-the-shell)




<Note>

### Quando devo usar `prerender`? {/*when-to-use-prerender*/}

A API estática `prerender` é usada para geração estática no lado do servidor (SSG). Diferente de `renderToString`, `prerender` espera que todos os dados sejam carregados antes de resolver. Isso o torna adequado para gerar HTML estático para uma página completa, incluindo dados que precisam ser buscados usando Suspense. Para transmitir conteúdo à medida que ele carrega, use uma API de renderização no lado do servidor (SSR) por streaming, como [renderToReadableStream](/reference/react-dom/server/renderToReadableStream).

</Note>

---

## Uso {/*usage*/}

### Renderizando uma árvore React para uma stream de HTML estático {/*rendering-a-react-tree-to-a-stream-of-static-html*/}

Chame `prerender` para renderizar sua árvore React para HTML estático em um [Readable Web Stream:](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream):

```js [[1, 4, "<App />"], [2, 5, "['/main.js']"]]
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Junto com o  <CodeStep step={1}>componente root</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos de `<script>` de bootstrap</CodeStep>. Seu componente root deve retornar **todo o documento, incluindo a tag `<html>` root.**

Por exemplo, pode ser assim que fica:

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

O React injetará o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas <CodeStep step={2}>tags `<script>` de bootstrap</CodeStep> na stream HTML resultante:

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

No cliente, seu script de bootstrap deve [hidratar o `document` inteiro com uma chamada para `hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

Isso irá anexar event listeners ao HTML estático gerado no servidor e torná-lo interativo.

<DeepDive>

#### Lendo os caminhos dos assets CSS e JS do output da build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

As URLs finais dos assets (como arquivos JavaScript e CSS) geralmente são *hashed* após a build. Por exemplo, em vez de   `styles.css` você pode acabar com  `styles.123456.css`. *Hashing* de nomes de arquivos de assets estáticos garante que toda build distinta do mesmo asset terá um nome de arquivo diferente. Isso é útil porque permite que você habilite com segurança o longo prazo do caching para assets estáticos: um arquivo com um determinado nome nunca mudaria de conteúdo.

No entanto, se você não souber as URLs dos assets até depois da build, não há como colocá-las no código-fonte. Por exemplo, *hardcoding*  `"/styles.css"` em JSX como antes não funcionaria. Para mantê-los fora do seu código-fonte, seu componente root pode ler os nomes de arquivos reais de um mapa passado como uma *prop*:

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

No servidor, renderize  `<App assetMap={assetMap} />` e passe seu `assetMap` com as URLs dos assets:

```js {1-5,8,9}
// Você precisaria obter esse JSON da sua ferramenta de build, por ex. lê-lo do output de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const {prelude} = await prerender(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['/main.js']]
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Como seu servidor agora está renderizando  `<App assetMap={assetMap} />`, você também precisa renderizá-lo com  `assetMap` no cliente para evitar erros de hidratação. Você pode serializar e passar  `assetMap` para o cliente desta forma:

```js {9-10}
// Você precisaria obter esse JSON da sua ferramenta de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const {prelude} = await prerender(<App assetMap={assetMap} />, {
    // Cuidado: É seguro fazer stringify() disso porque esses dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No exemplo acima, a opção  `bootstrapScriptContent` adiciona uma tag `<script>` embutida extra que define a variável global `window.assetMap` no cliente. Isso permite que o código do cliente leia o mesmo `assetMap`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

Tanto o cliente quanto o servidor renderizam `App` com a mesma *prop* `assetMap`, então não há erros de hidratação.

</DeepDive>

---

### Renderizando uma árvore React para uma string de HTML estático {/*rendering-a-react-tree-to-a-string-of-static-html*/}

Chame `prerender` para renderizar seu app para uma string HTML estática:

```js
import { prerender } from 'react-dom/static';

async function renderToString() {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  
  const reader = prelude.getReader();
  let content = '';
  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      return content;
    }
    content += Buffer.from(value).toString('utf8');
  }
}
```

Isso produzirá o output HTML inicial não interativo de seus componentes React. No cliente, você precisará chamar [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para *hidratar* o HTML gerado no servidor e torná-lo interativo.

---

### Esperando todos dados serem carregados {/*waiting-for-all-data-to-load*/}

`prerender` espera que todos os dados terminem de carregar antes de finalizar a geração de HTML estático e resolver. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos e uma lista de posts:

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

Imagine que `<Posts />` precisa carregar alguns dados, o que leva algum tempo. Idealmente, você gostaria de esperar os posts terminarem para que ele seja incluído no HTML. Para fazer isso, você pode usar o Suspense para suspender os dados, e `prerender`  esperará o conteúdo suspenso terminar antes de resolver para o HTML estático.

<Note>

**Apenas as fontes de dados habilitadas para Suspense ativarão o componente Suspense.** Elas incluem:

- Busca de dados com frameworks habilitados para Suspense, como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Carregamento lento do código do componente com [`lazy`](/reference/react/lazy)
- Lendo o valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são buscados dentro de um Effect ou manipulador de eventos.

A maneira exata como você carregaria os dados no componente `Posts` acima depende do seu framework. Se você usa um framework habilitado para Suspense, você encontrará os detalhes em sua documentação de busca de dados.

A busca de dados habilitada para Suspense sem o uso de um framework com opiniões próprias ainda não é suportada. Os requisitos para implementar uma fonte de dados habilitada para Suspense são instáveis ​​e não documentados. Uma API oficial para integrar fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

## Solução de problemas {/*troubleshooting*/}

### Meu stream não começa até que o app inteiro seja renderizado {/*my-stream-doesnt-start-until-the-entire-app-is-rendered*/}

A resposta de `prerender` espera a renderização do app inteiro ser concluída, incluindo esperar que todas as bordas de Suspense sejam resolvidas, antes de resolver. É projetado para geração de site estático (SSG) com antecedência e não suporta transmitir mais conteúdo à medida que ele carrega.

Para transmitir conteúdo à medida que ele carrega, use uma API de renderização no servidor por streaming, como [renderToReadableStream](/reference/react-dom/server/renderToReadableStream).
```