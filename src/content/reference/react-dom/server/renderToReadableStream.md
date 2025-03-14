---
title: renderToReadableStream
---

<Intro>

`renderToReadableStream` renderiza uma árvore React para um [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
const stream = await renderToReadableStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API depende de [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) Para Node.js, use [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream) em vez disso.

</Note>

---

## Referência {/*reference*/}

### `renderToReadableStream(reactNode, options?)` {/*rendertoreadablestream*/}

Chame `renderToReadableStream` para renderizar sua árvore React como HTML em um [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
import { renderToReadableStream } from 'react-dom/server';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você deseja renderizar em HTML. Por exemplo, um elemento JSX como `<App />`. Espera-se que ele represente o documento inteiro, portanto, o componente `App` deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de streaming.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` embutida.
  * **opcional** `bootstrapScripts`: Um array de URLs de strings para as tags `<script>` a serem emitidas na página. Use isto para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omita isto se você não quiser executar React no cliente.
  * **opcional** `bootstrapModules`: Assim como `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar vários roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **optional** `namespaceURI`: Uma stringcom o root [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) para o stream. O padrão é HTML normal. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `nonce`: Uma string [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) para permitir scripts para [`script-src` Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src).
  * **optional** `onError`: Um callback que é executado sempre que há um erro do servidor, seja [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isso apenas chama `console.error`. Se você substituí-lo para [registrar relatórios de falhas,](#logging-crashes-on-the-server) certifique-se de ainda chamar `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **optional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **optional** `signal`: Um [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) que permite que você [abort server rendering](#aborting-server-rendering) e renderize o restante no cliente.

#### Retorna {/*returns*/}

`renderToReadableStream` retorna uma Promise:

- Se a renderização do [shell](#specifying-what-goes-into-the-shell) for bem-sucedida, essa Promise resolverá para um [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)
- Se a renderização do shell falhar, a Promise será rejeitada. [Use isso para gerar um shell fallback.](#recovering-from-errors-inside-the-shell)

O stream retornado tem uma propriedade adicional:

* `allReady`: Uma Promise que é resolvida quando toda a renderização é concluída, incluindo tanto o [shell](#specifying-what-goes-into-the-shell) quanto o [conteúdo](#streaming-more-content-as-it-loads) adicional. Você pode `await stream.allReady` antes de retornar uma resposta [para rastreadores e geração estática.](#waiting-for-all-content-to-load-for-crawlers-and-static-generation) Se você fizer isso, não obterá nenhum carregamento progressivo. O stream conterá o HTML final.

---

## Uso {/*usage*/}

### Renderizando uma árvore React como HTML para um Readable Web Stream {/*rendering-a-react-tree-as-html-to-a-readable-web-stream*/}

Chame `renderToReadableStream` para renderizar sua árvore React como HTML em um [Readable Web Stream:](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js [[1, 4, "<App />"], [2, 5, "['/main.js']"]]
import { renderToReadableStream } from 'react-dom/server';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Juntamente com o <CodeStep step={1}>componente root</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos do `<script>` de bootstrap</CodeStep>. Seu componente root deve retornar **o documento inteiro, incluindo a tag `<html>` root.**

Por exemplo, pode ter este aspeto:

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

O React injetará o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas <CodeStep step={2}>tags `<script>` de bootstrap</CodeStep> no fluxo de HTML resultante:

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

Isto anexará event listeners ao HTML gerado pelo servidor e torná-lo-á interativo.

<DeepDive>

#### Lendo os caminhos dos assets CSS e JS da saída da build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

As URLs finais dos assets (como arquivos JavaScript e CSS) são frequentemente hash após a build. Por exemplo, em vez de `styles.css`, você pode acabar com `styles.123456.css`. Fazer hash dos nomes de arquivos de assets estáticos garante que cada build distinto do mesmo asset terá um nome de arquivo diferente. Isso é útil porque permite que você habilite com segurança o cache de longo prazo para assets estáticos: um arquivo com um determinado nome nunca mudaria o conteúdo.

No entanto, se você não souber as URLs dos assets até depois da build, não há como colocá-las no código-fonte. Por exemplo, codificar `"/styles.css"` em JSX, como antes, não funcionaria. Para mantê-los fora do seu código-fonte, seu componente root pode ler os nomes de arquivos reais de um mapa passado como uma prop:

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
// Você precisará obter este JSON da sua ferramenta de build, por exemplo, lê-lo da saída da build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const stream = await renderToReadableStream(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['/main.js']]
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Como seu servidor está renderizando `<App assetMap={assetMap} />`, agora, você precisa renderizá-lo com `assetMap` no cliente também para evitar erros de hidratação. Você pode serializar e passar `assetMap` para o cliente assim:

```js {9-10}
// Você precisará obter este JSON da sua ferramenta de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const stream = await renderToReadableStream(<App assetMap={assetMap} />, {
    // Cuidado: É seguro fazer stringify() disto porque estes dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No exemplo acima, a opção `bootstrapScriptContent` adiciona uma tag extra `<script>` embutida que define a variável global `window.assetMap` no cliente. Isso permite que o código do cliente leia o mesmo `assetMap`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

Tanto o cliente quanto o servidor renderizam `App` com a mesma prop `assetMap`, portanto, não há erros de hidratação.

</DeepDive>

---

### Streaming de mais conteúdo à medida que ele carrega {/*streaming-more-content-as-it-loads*/}

O streaming permite que o usuário comece a ver o conteúdo mesmo antes que todos os dados tenham carregado no servidor. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos e uma lista de posts:

```js
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Posts />
    </ProfileLayout>
  );
}
```

Imagine que carregar dados para `<Posts />` leva algum tempo. Idealmente, você gostaria de mostrar o restante do conteúdo da página de perfil para o usuário sem esperar pelos posts. Para fazer isso, [envolva `Posts` em uma boundary `<Suspense>`:](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)

```js {9,11}
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

Isso diz ao React para começar a transmitir o HTML antes que `Posts` carregue seus dados. O React enviará o HTML para o fallback de carregamento (`PostsGlimmer`) primeiro e, em seguida, quando `Posts` terminar de carregar seus dados, o React enviará o HTML restante junto com uma tag `<script>` embutida que substitui o fallback de carregamento por esse HTML. Da perspectiva do usuário, a página aparecerá primeiro com o `PostsGlimmer` e, posteriormente, será substituída pelos `Posts`.

Você pode ainda [anexar boundaries `<Suspense>`](/reference/react/Suspense#revealing-nested-content-as-it-loads) para criar uma sequência de carregamento mais granular:

```js {5,13}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

Neste exemplo, o React pode começar a transmitir a página ainda mais cedo. Apenas `ProfileLayout` e `ProfileCover` devem terminar a renderização primeiro porque não estão envoltos em nenhuma boundary `<Suspense>`. No entanto, se `Sidebar`, `Friends` ou `Photos` precisarem carregar alguns dados, o React enviará o HTML para o fallback `BigSpinner` em vez disso. Em seguida, à medida que mais dados se tornam disponíveis, mais conteúdo continuará a ser revelado até que tudo se torne visível.

O streaming não precisa esperar que o React carregue no navegador, ou que seu app se torne interativo. O conteúdo HTML do servidor será revelado progressivamente antes que qualquer uma das tags `<script>` carregue.

[Leia mais sobre como o streaming de HTML funciona.](https://github.com/reactwg/react-18/discussions/37)

<Note>

**Somente fontes de dados habilitadas para Suspense ativarão o componente Suspense.** Elas incluem:

- Obtendo dados com frameworks habilitados para Suspense como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Carregamento lento (lazy-loading) do código do componente com [`lazy`](/reference/react/lazy)
- Lendo o valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são obtidos dentro de um Effect ou manipulador de eventos.

A forma exata como você carregaria dados no componente `Posts` acima depende do seu framework. Se você usa um framework habilitado para Suspense, você encontrará os detalhes na sua documentação de obtenção de dados.

A obtenção de dados habilitada para Suspense sem o uso de um framework opinativo ainda não é suportada. Os requisitos para implementar uma fonte de dados habilitada para Suspense são instáveis e não documentados. Uma API oficial para integrar fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

### Especificando o que vai para o shell {/*specifying-what-goes-into-the-shell*/}

A parte do seu app fora de qualquer boundary `<Suspense>` é chamado de *shell:*

```js {3-5,13,14}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

Ele determina o estado de carregamento mais adiantado que o usuário pode ver:

```js {3-5,13
<ProfileLayout>
  <ProfileCover />
  <BigSpinner />
</ProfileLayout>
```

Se você envolver o aplicativo inteiro em uma boundary `<Suspense>` na raiz, o shell conterá apenas esse spinner. No entanto, essa não é uma experiência do usuário agradável porque ver um spinner grande na tela pode parecer mais lento e mais irritante do que esperar um pouco mais e ver o layout real. É por isso que, normalmente, você vai querer colocar os limites de `<Suspense>` para que o shell pareça *mínimo, mas completo*, como um esqueleto de todo o layout da página.

A chamada async para `renderToReadableStream` vai resolver para um `stream` assim que a shell inteira for renderizada. Normalmente, você vai começar a fazer streaming então criando e retornando uma resposta com esse `stream`:

```js {5}
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

No momento em que o `stream` é retornado, os componentes em limites `<Suspense>` aninhados ainda podem estar carregando dados.

---

### Registrando falhas no servidor {/*logging-crashes-on-the-server*/}

Por padrão, todos os erros no servidor são registrados no console. Você pode substituir este comportamento para registrar relatórios de falhas:

```js {4-7}
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onError(error) {
      console.error(error);
      logServerCrashReport(error);
    }
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Se você fornecer uma implementação `onError` personalizada, não se esqueça de também registrar erros no console, como acima.

---

### Recuperando de erros dentro do shell {/*recovering-from-errors-inside-the-shell*/}

Neste exemplo, o shell contém `ProfileLayout`, `ProfileCover` e `PostsGlimmer`:

```js {3-5,7-8}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Se ocorrer um erro durante a renderização desses componentes, o React não terá nenhum HTML significativo para enviar para o cliente. Envolva sua chamada `renderToReadableStream` em um `try...catch` para enviar um HTML de fallback que não dependa da renderização do servidor como último recurso:

```js {2,13-18}
async function handler(request) {
  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>Something went wrong</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

Se houver um erro ao gerar o shell, tanto o bloco `onError` quanto o seu bloco `catch` serão executados. Use o `onError` para relatórios de erros e use o bloco `catch` para enviar o documento HTML de fallback. Seu HTML de fallback não precisa ser uma página de erro. Em vez disso, você pode incluir um shell alternativo que renderiza seu aplicativo apenas no cliente.

---

### Recuperando de erros fora do shell {/*recovering-from-errors-outside-the-shell*/}

Neste exemplo, o componente `<Posts />` é envolvido em `<Suspense>` para que *não* faça parte do shell:

```js {6}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Se ocorrer um erro no componente `Posts` ou em algum lugar dentro dele, o React [tentará se recuperar dele:](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

1. Ele gerará o fallback de carregamento para o limite `<Suspense>` mais próximo (`PostsGlimmer`) no HTML.
2. Ele "desistirá" de tentar renderizar o conteúdo de `Posts` no servidor.
3. Quando o código JavaScript carrega no cliente, o React *tentará* renderizar `Posts` no cliente.

Se a nova tentativa de renderização de `Posts` no cliente *também* falhar, o React lançará o erro no cliente. Como com todos os erros lançados durante a renderização, o [limite do erro pai mais próximo](/reference/react/Component#static-getderivedstatefromerror) determina como apresentar o erro ao usuário. Na prática, isso significa que o usuário verá um indicador de carregamento até que se tenha certeza de que o erro não é recuperável.

Se a nova tentativa de renderização de `Posts` no cliente for bem-sucedida, o fallback de carregamento do servidor será substituído pela saída de renderização do cliente. O usuário não saberá que houve um erro no servidor. No entanto, os callbacks `onError` do servidor e os callbacks [`onRecoverableError`](/reference/react-dom/client/hydrateRoot#hydrateroot) do cliente serão executados para que você possa ser notificado sobre o erro.

---

### Configurando o código de status {/*setting-the-status-code*/}

O streaming introduz uma troca. Você quer começar a transmitir a página o mais cedo possível para que o usuário possa ver o conteúdo o mais rápido possível. No entanto, assim que você começar a fazer streaming, não poderá mais configurar o código de status da resposta.

Ao [dividir seu aplicativo](#specifying-what-goes-into-the-shell) em shell (acima de todos os limites `<Suspense>`) e o restante do conteúdo, você já resolveu parte desse problema. Se o shell apresentar erros, seu bloco `catch` será executado, o que permite que você defina o código de erro de status. Caso contrário, você sabe que o aplicativo pode se recuperar no cliente, então você pode enviar "OK".

```js {11}
async function handler(request) {
  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>Something went wrong</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

Se um componente *fora* do shell (ou seja, dentro de uma boundary `<Suspense>`) lançar um erro, o React não interromperá a renderização. Isso significa que o callback `onError` será executado, mas seu código continuará sendo executado sem entrar no bloco `catch`. Isso ocorre porque o React tentará se recuperar desse erro no cliente, [conforme descrito acima.](#recovering-from-errors-outside-the-shell)

No entanto, se quiser, você pode usar o fato de que algo deu errado para definir o código de status:

```js {3,7,13}
async function handler(request) {
  try {
    let didError = false;
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: didError ? 500 : 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>Something went wrong</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

Isso só capturará os erros fora do shell que ocorreram ao gerar o conteúdo inicial do shell, portanto, não é exaustivo. Se saber se um erro ocorreu para algum conteúdo é fundamental, você pode movê-lo para o shell.

---

### Lidar com diferentes erros de diferentes maneiras {/*handling-different-errors-in-different-ways*/}

Você pode [criar suas próprias subclasses `Error`](https://javascript.info/custom-errors) e usar o operador [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) para verificar qual erro é lançado. Por exemplo, você pode definir um `NotFoundError` personalizado e lançá-lo do seu componente. Em seguida, você pode salvar o erro em `onError` e fazer algo diferente antes de retornar a resposta, dependendo do tipo de erro:

```js {2-3,5-15,22,28,33}
async function handler(request) {
  let didError = false;
  let caughtError = null;

  function getStatusCode() {
    if (didError) {
      if (caughtError instanceof NotFoundError) {
        return 404;
      } else {
        return 500;
      }
    } else {
      return 200;
    }
  }

  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        caughtError = error;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: getStatusCode(),
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>Something went wrong</h1>', {
      status: getStatusCode(),
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

Tenha em mente que uma vez que você emite o shell e começa a fazer streaming, você não pode mudar o código de status.

---

### Esperando que todo o conteúdo seja carregado para rastreadores e geração estática {/*waiting-for-all-content-to-load-for-crawlers-and-static-generation*/}

O streaming oferece uma melhor experiência do usuário porque o usuário pode ver o conteúdo à medida que ele fica disponível.

No entanto, quando um rastreador visita sua página, ou se você estiver gerando as páginas no tempo de build, você pode querer deixar todo o conteúdo carregar primeiro e, em seguida, produzir a saída HTML final em vez de revelá-la progressivamente.

Você pode esperar que todo o conteúdo seja carregado esperando pela Promise `stream.allReady`:

```js {12-15}
async function handler(request) {
  try {
    let didError = false;
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    let isCrawler = // ... depende da sua estratégia de detecção de bot ...
    if (isCrawler) {
      await stream.allReady;
    }
    return new Response(stream, {
      status: didError ? 500 : 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>Something went wrong</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

Um visitante normal receberá um stream de conteúdo carregado progressivamente. Um rastreador receberá a saída HTML final depois que todos os dados forem carregados. No entanto, isso também significa que o rastreador terá que esperar por *todos* os dados, alguns dos quais podem demorar para carregar ou apresentar erros. Dependendo do seu aplicativo, você pode optar por enviar o shell para os rastreadores também.

---

### Abortando a renderização do servidor {/*aborting-server-rendering*/}

Você pode forçar a renderização do servidor a "desistir" após um tempo limite:

```js {3,4-6,9}
async function handler(request) {
  try {
    const controller = new AbortController();
    setTimeout(() => {
      controller.abort();
    }, 10000);

    const stream = await renderToReadableStream(<App />, {
      signal: controller.signal,
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    // ...
```

O React irá exibir os fallbacks de carregamento restantes como HTML e tentará renderizar o restante no cliente.
```