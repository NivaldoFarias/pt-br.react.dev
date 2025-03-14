---
title: renderToPipeableStream
---

<Intro>

`renderToPipeableStream` renderiza uma árvore React a um [Stream do Node.js.](https://nodejs.org/api/stream.html)

```js
const { pipe, abort } = renderToPipeableStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API é específica para Node.js. Ambientes com [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) como Deno e runtimes de borda modernos, devem usar [`renderToReadableStream`](/reference/react-dom/server/renderToReadableStream) ao invés.

</Note>

---

## Referência {/*reference*/}

### `renderToPipeableStream(reactNode, options?)` {/*rendertopipeablestream*/}

Chame `renderToPipeableStream` para renderizar sua árvore React como HTML em um [Node.js Stream.](https://nodejs.org/api/stream.html#writable-streams)

```js
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  }
});
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você deseja renderizar em HTML. Por exemplo, um elemento JSX como `<App />`. Espera-se que ele represente o documento inteiro, então o componente `App` deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de streaming.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` inline.
  * **opcional** `bootstrapScripts`: Um array de URLs de string para as tags `<script>` a serem emitidas na página. Use isso para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omita-o se não quiser executar o React no cliente.
  * **opcional** `bootstrapModules`: Como `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar várias roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **opcional** `namespaceURI`: Uma string com o [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) da raiz para o stream. O padrão é HTML regular. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `nonce`: Uma string [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) para permitir scripts para [`script-src` Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src).
  * **opcional** `onAllReady`: Uma callback que dispara quando toda a renderização estiver completa, incluindo tanto o [shell](#specifying-what-goes-into-the-shell) quanto todo [conteúdo adicional](#streaming-more-content-as-it-loads). Você pode usar isso em vez de `onShellReady` [para rastreadores e geração estática.](#waiting-for-all-content-to-load-for-crawlers-and-static-generation) Se você iniciar o streaming aqui, não terá nenhum carregamento progressivo. O stream conterá o HTML final.
  * **opcional** `onError`: Uma callback que dispara sempre que houver um erro no servidor, seja [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isso apenas chama `console.error`. Se você substituí-lo para [registrar relatórios de falha,](#logging-crashes-on-the-server) certifique-se de ainda chamar `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **opcional** `onShellReady`: Uma callback que dispara logo após o [shell inicial](#specifying-what-goes-into-the-shell) ter sido renderizado. Você pode [definir o código de status](#setting-the-status-code) e chamar `pipe` aqui para iniciar o streaming. O React [transmitirá o conteúdo adicional](#streaming-more-content-as-it-loads) após o shell, juntamente com as tags `<script>` inline que substituem os fallbacks de carregamento HTML pelo conteúdo.
  * **opcional** `onShellError`: Uma callback que dispara se houve um erro ao renderizar o shell inicial. Ele recebe o erro como argumento. Nenhum byte foi emitido do stream ainda, e nem `onShellReady` nem `onAllReady` serão chamados, então você pode [emitir um fallback HTML shell.](#recovering-from-errors-inside-the-shell)
  * **opcional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)


#### Retorna {/*returns*/}

`renderToPipeableStream` retorna um objeto com dois métodos:

* `pipe` envia o HTML para o [Writable Node.js Stream](https://nodejs.org/api/stream.html#writable-streams) fornecido. Chame `pipe` em `onShellReady` se quiser ativar o streaming, ou em `onAllReady` para rastreadores e geração estática.
* `abort` permite que você [interrompa a renderização do servidor](#aborting-server-rendering) e renderize o restante no cliente.

---

## Uso {/*usage*/}

### Renderizando uma árvore React como HTML em um Node.js Stream {/*rendering-a-react-tree-as-html-to-a-nodejs-stream*/}

Chame `renderToPipeableStream` para renderizar sua árvore React como HTML em um [Node.js Stream:](https://nodejs.org/api/stream.html#writable-streams)

```js [[1, 5, "<App />"], [2, 6, "['/main.js']"]]
import { renderToPipeableStream } from 'react-dom/server';

// A sintaxe do manipulador de rota depende do seu framework de backend
app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
```

Junto com o <CodeStep step={1}>componente raiz</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos do `<script>` de bootstrap</CodeStep>. Seu componente raiz deve retornar **todo o documento, incluindo a tag `<html>`.**

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

O React irá injetar o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas <CodeStep step={2}>tags `<script>` de bootstrap</CodeStep> no stream HTML resultante:

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

No cliente, seu script de bootstrap deve [hidratar todo o `document` com uma chamada para `hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

Isso irá anexar event listeners ao HTML gerado pelo servidor e torná-lo interativo.

<DeepDive>

#### Lendo caminhos de assets CSS e JS da saída da build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

Os URLs finais de assets (como arquivos JavaScript e CSS) são frequentemente _hashed_ após a build. Por exemplo, em vez de `styles.css`, você pode acabar com `styles.123456.css`. Fazer o _hash_ de nomes de arquivos de ativos estáticos garante que cada build distinto do mesmo ativo terá um nome de arquivo diferente. Isso é útil porque permite que você habilite com segurança o _caching_ de longo prazo para ativos estáticos: um arquivo com um determinado nome nunca mudaria o conteúdo.

No entanto, se você não souber os URLs dos assets até depois da build, não haverá como colocá-los no código-fonte. Por exemplo, _hardcodar_ `"/styles.css"` no JSX como antes não funcionaria. Para mantê-los fora do seu código-fonte, seu componente raiz pode ler os nomes de arquivos reais de um mapa passado como um prop:

```js {1,6}
export default function App({ assetMap }) {
  return (
    <html>
      <head>
        ...
        <link rel="stylesheet" href={assetMap['styles.css']}></link>
        ...
      </head>
      ...
    </html>
  );
}
```

No servidor, renderize `<App assetMap={assetMap} />` e passe seu `assetMap` com os URLs dos assets:

```js {1-5,8,9}
// Você precisaria obter este JSON de suas ferramentas de build, por exemplo, lê-lo da saída da build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['main.js']],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
```

Como seu servidor agora está renderizando `<App assetMap={assetMap} />`, você precisa renderizá-lo com `assetMap` no cliente também para evitar erros de hidratação. Você pode serializar e passar `assetMap` para o cliente da seguinte forma:

```js {9-10}
// Você precisaria obter este JSON de suas ferramentas de build.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    // Cuidado: É seguro stringify() isto porque esses dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['main.js']],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
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

### Transmitindo mais conteúdo conforme ele carrega {/*streaming-more-content-as-it-loads*/}

O streaming permite que o usuário comece a ver o conteúdo mesmo antes que todos os dados tenham sido carregados no servidor. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos e uma lista de posts:

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

Imagine que o carregamento de dados para `<Posts />` leva algum tempo. Idealmente, você gostaria de mostrar o restante do conteúdo da página do perfil ao usuário sem esperar pelos posts. Para fazer isso, [envolva `Posts` em um limite `<Suspense>`:](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)

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

Isso diz ao React para começar a transmitir o HTML antes que `Posts` carregue seus dados. O React enviará o HTML para o fallback de carregamento (`PostsGlimmer`) primeiro e, em seguida, quando `Posts` terminar de carregar seus dados, o React enviará o HTML restante junto com uma tag `<script>` inline que substitui o fallback de carregamento por esse HTML. Da perspectiva do usuário, a página aparecerá primeiro com o `PostsGlimmer` e, posteriormente, será substituída pelos `Posts`.

Você pode ainda [aninhar limites `<Suspense>`](/reference/react/Suspense#revelando-nested-content-as-it-loads) para criar uma sequência de carregamento mais granular:

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

Neste exemplo, o React pode começar a transmitir a página ainda mais cedo. Apenas `ProfileLayout` e `ProfileCover` devem terminar de renderizar primeiro porque não estão envolvidos em nenhum limite `<Suspense>`. No entanto, se `Sidebar`, `Friends` ou `Photos` precisarem carregar alguns dados, o React enviará o HTML do fallback `BigSpinner` em vez disso. Então, à medida que mais dados se tornarem disponíveis, mais conteúdo continuará a ser revelado até que tudo se torne visível.

O streaming não precisa esperar que o próprio React carregue no navegador ou que seu aplicativo se torne interativo. O conteúdo HTML do servidor será progressivamente revelado antes que qualquer uma das tags `<script>` carregue.

[Leia mais sobre como o streaming HTML funciona.](https://github.com/reactwg/react-18/discussions/37)

<Note>

**Somente fontes de dados habilitadas para Suspense ativarão o componente Suspense.** Eles incluem:

- Busca de dados com frameworks habilitados para Suspense como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Código de componente de carregamento lento com [`lazy`](/reference/react/lazy)
- Lendo o valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são buscados dentro de um Effect ou manipulador de eventos.

A maneira exata de carregar dados no componente `Posts` acima depende do seu framework. Se você usar um framework habilitado para Suspense, encontrará os detalhes em sua documentação de busca de dados.

A busca de dados habilitada para Suspense sem o uso de um framework opinativo ainda não é suportada. Os requisitos para a implementação de uma fonte de dados habilitada para Suspense são instáveis e não documentados. Uma API oficial para a integração de fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

### Especificando o que vai para o shell {/*specifying-what-goes-into-the-shell*/}

A parte do seu aplicativo fora de qualquer limite `<Suspense>` é chamada de *shell:*

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

Ele determina o primeiro estado de carregamento que o usuário pode ver:

```js {3-5,13
<ProfileLayout>
  <ProfileCover />
  <BigSpinner />
</ProfileLayout>
```

Se você envolver todo o aplicativo em um limite `<Suspense>` na raiz, o shell conterá apenas esse spinner. No entanto, essa não é uma experiência de usuário agradável, porque ver um spinner grande na tela pode parecer mais lento e mais irritante do que esperar um pouco mais e ver o layout real. É por isso que, geralmente, você vai querer colocar os limites `<Suspense>` para que o shell pareça *mínimo, mas completo* - como um esqueleto de todo o layout da página.

A callback `onShellReady` dispara quando todo o shell foi renderizado. Normalmente, você iniciará o streaming então:

```js {3-6}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  }
});
```

Quando `onShellReady` disparar, componentes em limites `<Suspense>` aninhados ainda podem estar carregando dados.

---

### Registrar falhas no servidor {/*logging-crashes-on-the-server*/}

Por padrão, todos os erros no servidor são registrados no console. Você pode substituir esse comportamento para registrar relatórios de falhas:

```js {7-10}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se você fornecer uma implementação `onError` customizada, não se esqueça de também registrar erros no console como acima.

---

### Recuperando-se de erros dentro do shell {/*recovering-from-errors-inside-the-shell*/}

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

Se ocorrer um erro ao renderizar esses componentes, o React não terá nenhum HTML significativo para enviar ao cliente. Substitua `onShellError` para enviar um fallback HTML que não dependa da renderização do servidor como último recurso:

```js {7-11}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se houver um erro ao gerar o shell, tanto `onError` quanto `onShellError` serão disparados. Use `onError` para relatórios de erros e use `onShellError` para enviar o documento HTML de fallback. Seu HTML de fallback não precisa ser uma página de erro. Em vez disso, você pode incluir um shell alternativo que renderiza seu aplicativo apenas no cliente.

---

### Recuperando-se de erros fora do shell {/*recovering-from-errors-outside-the-shell*/}

Neste exemplo, o componente `<Posts />` está envolvido em `<Suspense>` para que *não* faça parte do shell:

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

Se um erro ocorrer no componente `Posts` ou em algum lugar dentro dele, o React [tentará se recuperar dele:](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

1. Ele emitirá o fallback de carregamento para o limite `<Suspense>` mais próximo (`PostsGlimmer`) no HTML.
2. Ele "desistirá" de tentar renderizar o conteúdo de `Posts` no servidor.
3. Quando o código JavaScript carregar no cliente, o React *tentará novamente* renderizar `Posts` no cliente.

Se tentar renderizar `Posts` no cliente *também* falhar, o React lançará o erro no cliente. Como com todos os erros lançados durante a renderização, o [limite de erro pai mais próximo](/reference/react/Component#static-getderivedstatefromerror) determina como apresentar o erro ao usuário. Na prática, isso significa que o usuário verá um indicador de carregamento até ter certeza de que o erro não é recuperável.

Se tentar renderizar `Posts` no cliente for bem-sucedido, o fallback de carregamento do servidor será substituído pela saída da renderização do cliente. O usuário não saberá que houve um erro no servidor. No entanto, o callback `onError` do servidor e os callbacks [`onRecoverableError`](/reference/react-dom/client/hydrateRoot#hydrateroot) do cliente dispararão para que você possa ser notificado sobre o erro.

---

### Definindo o código de status {/*setting-the-status-code*/}

O streaming introduz uma troca. Você deseja iniciar o streaming da página o mais cedo possível para que o usuário possa ver o conteúdo o mais rápido possível. No entanto, depois de iniciar o streaming, você não pode mais definir o código de status da resposta.

Ao [dividir seu aplicativo](#specifying-what-goes-into-the-shell) no shell (acima de todos os limites `<Suspense>`) e o restante do conteúdo, você já resolveu parte desse problema. Se o shell tiver erros, você receberá o callback `onShellError`, que permite que você defina o código de status do erro. Caso contrário, você sabe que o aplicativo pode se recuperar no cliente, então você pode enviar "OK".

```js {4}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = 200;
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se um componente *fora* do shell (ou seja, dentro de um limite `<Suspense>`) lançar um erro, o React não interromperá a renderização. Isso significa que a callback `onError` será disparada, mas você ainda receberá `onShellReady` em vez de `onShellError`. Isso ocorre porque o React tentará se recuperar desse erro no cliente, [conforme descrito acima.](#recovering-from-errors-outside-the-shell)

No entanto, se você quiser, pode usar o fato de algo ter dado erro para definir o código de status:

```js {1,6,16}
let didError = false;

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = didError ? 500 : 200;
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    didError = true;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Isso só capturará erros fora do shell que ocorreram durante a geração do conteúdo inicial do shell, portanto, não é exaustivo. Se saber se um erro ocorreu para algum conteúdo for crítico, você pode movê-lo para o shell.

---

### Manipulando diferentes erros de maneiras diferentes {/*handling-different-errors-in-different-ways*/}

Você pode [criar suas próprias subclasses `Error`](https://javascript.info/custom-errors) e usar o operador [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) para verificar qual erro é lançado. Por exemplo, você pode definir um `NotFoundError` customizado e lançá-lo de seu componente. Então suas callbacks `onError`, `onShellReady` e `onShellError` podem fazer algo diferente dependendo do tipo de erro:

```js {2,4-14,19,24,30}
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

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = getStatusCode();
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
   response.statusCode = getStatusCode();
   response.setHeader('content-type', 'text/html');
   response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    didError = true;
    caughtError = error;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Lembre-se de que, depois de emitir o shell e iniciar o streaming, você não pode alterar o código de status.

---

### Esperando que todo o conteúdo seja carregado para rastreadores e geração estática {/*waiting-for-all-content-to-load-for-crawlers-and-static-generation*/}

O streaming oferece uma melhor experiência para o usuário, porque ele pode ver o conteúdo à medida que ele se torna disponível.

No entanto, quando um rastreador visita sua página, ou se você estiver gerando as páginas no tempo da build, você pode querer deixar todo o conteúdo carregar primeiro e, em seguida, produzir a saída HTML final, em vez de revelá-lo progressivamente.

Você pode esperar que todo o conteúdo seja carregado usando a callback `onAllReady`:


```js {2,7,11,18-24}
let didError = false;
let isCrawler = // ... depende da sua estratégia de detecção de bot ...

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    if (!isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onAllReady() {
    if (isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);      
    }
  },
  onError(error) {
    didError = true;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Um visitante normal receberá um fluxo de conteúdo carregado progressivamente. Um rastreador receberá a saída HTML final depois que todos os dados forem carregados. No entanto, isso também significa que o rastreador terá que esperar por *todos* os dados, alguns dos quais podem demorar para carregar ou ter erros. Dependendo do seu aplicativo, você pode optar por enviar o shell para os rastreadores também.

---

### Interrompendo a renderização do servidor {/*aborting-server-rendering*/}

Você pode forçar a renderização do servidor a "desistir" após um tempo limite:

```js {1,5-7}
const { pipe, abort } = renderToPipeableStream(<App />, {
  // ...
});

setTimeout(() => {
  abort();
}, 10000);
```

O React esvaziará os fallbacks de carregamento restantes como HTML e tentará renderizar o restante no cliente.