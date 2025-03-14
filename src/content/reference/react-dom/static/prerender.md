---
title: prerender
---

<Intro>

`prerender` renderiza uma árvore React para uma string HTML estática usando um [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API).

```js
const {prerender} = await prerender(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API depende de [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) Para Node.js, use então [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream).

</Note>

---

## Referência {/*reference*/}

### `prerender(reactNode, options?)` {/*prerender*/}

Chame `prerender` para renderizar seu app em HTML estático.

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

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você quer renderizar em HTML. Por exemplo, um nó JSX como `<App />`. Espera-se que ele represente o documento inteiro, então o componente App deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de geração estática.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` inline.
  * **opcional** `bootstrapScripts`: Um array de URLs de string para as tags `<script>` a serem emitidas na página. Use isso para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omiti-lo se você não quiser executar o React no cliente.
  * **opcional** `bootstrapModules`: Como `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar múltiplas roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **opcional** `namespaceURI`: Uma string com o root [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) para o stream. Por padrão é HTML normal. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `onError`: Um callback que dispara sempre que há um erro de servidor, seja [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isso só chama `console.error`. Se você o substituir para [registrar relatórios de falha,](#logging-crashes-on-the-server) certifique-se de ainda chamar `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **opcional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **opcional** `signal`: Um [sinal de aborto](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) que permite [abortar a renderização do servidor](#aborting-server-rendering) e renderizar o restante no cliente.

#### Retorna {/*returns*/}

`prerender` retorna uma Promise:
- Se a renderização for bem-sucedida, a Promise resolverá em um objeto contendo:
  - `prelude`: um [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) de HTML. Você pode usar este stream para enviar uma resposta em chunks, ou você pode ler o stream inteiro em uma string.
- Se a renderização falhar, a Promise será rejeitada. [Use isso para exibir um fallback shell.](#recovering-from-errors-inside-the-shell)

<Note>

### Quando devo usar `prerender`? {/*when-to-use-prerender*/}

A API estática `prerender` é usada para geração estática do lado do servidor (SSG). Diferente de `renderToString`, `prerender` espera que todos os dados carreguem antes de resolver. Isso o torna adequado para gerar HTML estático para uma página inteira, incluindo dados que precisam ser buscados usando Suspense. Para transmitir conteúdo durante o carregamento, use uma API de renderização do lado do servidor (SSR) como [renderToReadableStream](/reference/react-dom/server/renderToReadableStream).

</Note>

---

## Uso {/*usage*/}

### Renderizando uma árvore React em um stream de HTML estático {/*rendering-a-react-tree-to-a-stream-of-static-html*/}

Chame `prerender` para renderizar sua árvore React em HTML estático em um [Readable Web Stream:](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream):

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

Junto com o <CodeStep step={1}>componente root</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos `<script>` bootstrap</CodeStep>. Seu componente root deve retornar **o documento inteiro, incluindo a tag `<html>` root.**

Por exemplo, pode se parecer com isto:

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

O React irá injetar o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas <CodeStep step={2}>tags de bootstrap `<script>`</CodeStep> no stream HTML resultante:

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

Isto irá anexar event listeners ao HTML estático gerado pelo servidor e torná-lo interativo.

<DeepDive>

#### Lendo caminhos de assets CSS e JS da saída da build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

As URLs finais dos assets (como arquivos JavaScript e CSS) são frequentemente hashadas após a build. Por exemplo, em vez de `styles.css` você pode acabar com `styles.123456.css`. Hashar os nomes de arquivos de assets estáticos garante que cada build distinto do mesmo asset terá um nome de arquivo diferente. Isto é útil porque permite que você habilite o caching de longo prazo com segurança para assets estáticos: um arquivo com um determinado nome nunca mudaria de conteúdo.

Entretanto, se você não souber as URLs dos assets até depois da build, não há como colocá-las no código-fonte. Por exemplo, codificar `"/styles.css"` no JSX como antes não funcionaria. Para mantê-los fora do seu código-fonte, seu componente root pode ler os nomes de arquivos reais de um mapa que é passado como uma prop:

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
// Você precisará obter este JSON de suas ferramentas de build, por exemplo, lê-lo da saída da build.
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

Uma vez que seu servidor agora está renderizando `<App assetMap={assetMap} />`, você precisa renderizá-lo com `assetMap` no cliente também para evitar erros de hidratação. Você pode serializar e passar `assetMap` para o cliente desta forma:

```js {9-10}
// Você precisará obter este JSON de suas ferramentas de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const {prelude} = await prerender(<App assetMap={assetMap} />, {
    // Cuidado: É seguro stringificar() isto porque esses dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No exemplo acima, a opção `bootstrapScriptContent` adiciona uma tag `<script>` inline extra que define a variável global `window.assetMap` no cliente. Isso permite que o código do cliente leia o mesmo `assetMap`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

Tanto o cliente quanto o servidor renderizam `App` com a mesma prop `assetMap`, então não há erros de hidratação.

</DeepDive>

---

### Renderizando uma árvore React em uma string de HTML estático {/*rendering-a-react-tree-to-a-string-of-static-html*/}

Chame `prerender` para renderizar seu app em uma string HTML estática:

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

Isto irá produzir a saída HTML inicial não interativa de seus componentes React. No cliente, você precisará chamar [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para *hidratar* o HTML gerado pelo servidor e torná-lo interativo.

---

### Esperando que todos os dados carreguem {/*waiting-for-all-data-to-load*/}

`prerender` espera que todos dados carreguem antes de finalizar a geração do HTML estático e resolver. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos, e uma lista de posts:

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

Imagine que o `<Posts />` precisa carregar alguns dados, o que leva algum tempo. Idealmente, você gostaria de esperar que os posts terminem para que ele seja incluído no HTML. Para fazer isso, você pode usar Suspense para suspender os dados, e `prerender` irá esperar que o conteúdo suspenso termine antes de resolver para o HTML estático.

<Note>

**Apenas fontes de dados com Suspense habilitado ativarão o componente Suspense.** Elas incluem:

- Busca de dados com frameworks com Suspense habilitado como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Carregamento lazy do código do componente com [`lazy`](/reference/react/lazy)
- Lendo o valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são buscados dentro de um Effect ou manipulador de eventos.

A maneira exata como você carregaria os dados no componente `Posts` acima depende do seu framework. Se você usar um framework com Suspense habilitado, você encontrará os detalhes na documentação de busca de dados dele.

Busca de dados com Suspense habilitado sem o uso de um framework com opinião ainda não é suportado. Os requisitos para implementação de uma fonte de dados com Suspense habilitado são instáveis e não documentados. Uma API oficial para integrar fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

## Solução de problemas {/*troubleshooting*/}

### Meu stream não começa até que o app inteiro seja renderizado {/*my-stream-doesnt-start-until-the-entire-app-is-rendered*/}

A resposta `prerender` espera que o app inteiro termine de renderizar, incluindo esperar que todas as fronteiras Suspense resolvam, antes de resolver. Ele é projetado para geração de site estático (SSG) com antecedência e não suporta a transmissão de mais conteúdo durante o carregamento.

Para transmitir conteúdo durante o carregamento, use uma API de renderização de servidor de streaming como [renderToReadableStream](/reference/react-dom/server/renderToReadableStream).