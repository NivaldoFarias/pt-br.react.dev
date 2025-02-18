# Como Fazer o Upgrade para o React 18

Este documento descreve as regras que devem ser aplicadas para **todos** os idiomas.
Quando estiver se referindo ao próprio `React`, use `o React`.

8 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 apresenta recursos com tecnologia de nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, iremos guiá-lo pelas etapas para fazer o upgrade para o React 18.

Por favor, [relate quaisquer problemas](https://github.com/facebook/react/issues/new/choose) que você encontrar ao fazer o upgrade para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 se baseia na Nova Arquitetura do React Native para se beneficiar dos novos recursos apresentados neste post do blog. Para obter mais informações, consulte o [discurso principal da React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

---

## Instalando {/*installing*/}

Para instalar a versão mais recente do React:

```bash
npm install react react-dom
```

Ou se você estiver usando yarn:

```bash
yarn add react react-dom
```

## Atualizações para APIs de Renderização do Cliente {/*updates-to-client-rendering-apis*/}

Quando você instala o React 18 pela primeira vez, verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais compatível com o React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 apresenta uma nova API raiz que oferece melhor ergonomia para o gerenciamento de raízes. A nova API raiz também habilita o novo renderizador concorrente, que permite que você opte por recursos concorrentes.

```js
// Before
import { render } from 'react-dom';
const container = document.getElementById('app');
render(<App tab="home" />, container);

// After
import { createRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = createRoot(container); // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />);
```

Também mudamos `unmountComponentAtNode` para `root.unmount`:

```js
// Before
unmountComponentAtNode(container);

// After
root.unmount();
```

Também removemos o callback do render, pois geralmente não tem o resultado esperado ao usar Suspense:

```js
// Before
const container = document.getElementById('app');
render(<App tab="home" />, container, () => {
  console.log('rendered');
});

// After
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log('rendered');
  });

  return <App tab="home" />
}

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

<Note>

Não existe uma substituição um-para-um para a antiga API de callback de renderização - ela depende do seu caso de uso. Consulte a publicação do grupo de trabalho para [Substituindo render com createRoot](https://github.com/reactwg/react-18/discussions/5) para obter mais informações.

</Note>

Finalmente, se seu aplicativo usa renderização do lado do servidor com hidratação, atualize `hydrate` para `hydrateRoot`:

```js
// Before
import { hydrate } from 'react-dom';
const container = document.getElementById('app');
hydrate(<App tab="home" />, container);

// After
import { hydrateRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = hydrateRoot(container, <App tab="home" />);
// Unlike with createRoot, you don't need a separate root.render() call here.
```

Para obter mais informações, consulte a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu aplicativo não funcionar após a atualização, verifique se ele está envolvido em `<StrictMode>`.** [O Strict Mode ficou mais rigoroso no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se a remoção do Strict Mode corrigir seu aplicativo, você pode removê-lo durante a atualização e, em seguida, adicioná-lo novamente (no topo ou para uma parte da árvore) depois de corrigir os problemas que ele está apontando.

</Note>

## Atualizações para APIs de Renderização do Servidor {/*updates-to-server-rendering-apis*/}

Neste lançamento, estamos renovando nossas APIs `react-dom/server` para oferecer suporte total ao Suspense no servidor e ao Streaming SSR. Como parte dessas alterações, estamos descontinuando a antiga API de streaming do Node, que não oferece suporte ao streaming incremental do Suspense no servidor.

O uso desta API agora avisará:

*   `renderToNodeStream`: **Descontinuado ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
*   `renderToPipeableStream`: **Novo ✨**

Também estamos introduzindo uma nova API para oferecer suporte a streaming SSR com Suspense para ambientes de tempo de execução de borda modernos, como Deno e trabalhadores do Cloudflare:
*   `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
*   `renderToString`: **Limitado** ⚠️
*   `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
*   `renderToStaticNodeStream`

Para obter mais informações sobre as alterações nas APIs de renderização do servidor, consulte a publicação do grupo de trabalho em [Atualizando para o React 18 no servidor](https://github.com/reactwg/react-18/discussions/22), um [mergulho profundo na nova Arquitetura SSR do Suspense](https://github.com/reactwg/react-18/discussões/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Atualizações para definições TypeScript {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e detectam problemas que costumavam ser ignorados pelo verificador de tipos. A alteração mais notável é que a prop `children` agora precisa ser listada explicitamente ao definir props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Consulte a [solicitação pull do React 18 typings](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para obter uma lista completa de alterações somente de tipo. Ele se conecta a correções de exemplo em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código do seu aplicativo para as novas e seguras typings mais rapidamente.

Se você encontrar um bug nas typings, por favor, [registre um problema](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Batching Automático {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais batching por padrão. Batching é quando o React agrupa várias atualizações de estado em uma única rerenderização para melhor desempenho. Antes do React 18, nós só fazíamos batching de atualizações dentro de manipuladores de eventos do React. As atualizações dentro de promises, setTimeout, manipuladores de eventos nativos ou qualquer outro evento não eram batched no React por padrão:

```js
// Before React 18 only React events were batched

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will render twice, once for each state update (no batching)
}, 1000);
```

A partir do React 18 com `createRoot`, todas as atualizações serão automaticamente batching, não importa de onde elas se originem. Isso significa que as atualizações dentro de timeouts, promises, manipuladores de eventos nativos ou qualquer outro evento agruparão da mesma forma que as atualizações dentro dos eventos React:

```js
// After React 18 updates inside of timeouts, promises,
// native event handlers or any other event are batched.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

Esta é uma alteração de última hora, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para desabilitar o batching automático, você pode usar `flushSync`:

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```

Para obter mais informações, consulte o [mergulho profundo no Batching automático](https://github.com/reactwg/react-18/discussões/21).

## Novas APIs para Bibliotecas {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os responsáveis pelas bibliotecas para criar novas APIs necessárias para oferecer suporte à renderização concorrente para casos de uso específicos para o seu caso de uso em áreas como estilos e lojas externas. Para oferecer suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

*   `useSyncExternalStore` é um novo Hook que permite que lojas externas ofereçam suporte a leituras simultâneas, forçando atualizações na loja a serem síncronas. Esta nova API é recomendada para qualquer biblioteca que se integre com estado externo ao React. Para obter mais informações, consulte a [postagem de visão geral useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
*   `useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após o DOM ter sido mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para obter mais informações, consulte o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também apresenta novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, das quais compartilhamos mais no [post de lançamento](/blog/2022/03/29/react-v18).

## Atualizações para o Modo Estrito {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite ao React adicionar e remover seções da interface do usuário, preservando o estado. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado do componente de antes.

Esse recurso dará ao React um melhor desempenho pronto para uso, mas exige que os componentes sejam resilientes a efeitos montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos assumem que eles são montados ou destruídos apenas uma vez.

Para ajudar a apresentar esses problemas, o React 18 apresenta uma nova verificação de Modo Estrito apenas para desenvolvimento. Essa nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:

```
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
```

Com o Modo Estrito no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
* React simulates unmounting the component.
    * Layout effects are destroyed.
    * Effects are destroyed.
* React simulates mounting the component with the previous state.
    * Layout effect setup code runs
    * Effect setup code runs
```

Para obter mais informações, consulte as postagens do Grupo de Trabalho para [Adicionando Estado Reutilizável ao StrictMode](https://github.com/reactwg/react-18/discussions/19) e [Como oferecer suporte ao Estado Reutilizável em Efeitos](https://github.com/reactwg/react-18/discussions/18).

## Configurando Seu Ambiente de Teste {/*configuring-your-testing-environment*/}

Quando você atualiza seus testes pela primeira vez para usar `createRoot`, pode ver este aviso no console do teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para oferecer suporte a act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// In your test setup file
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo da flag é dizer ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir a flag como `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes de ponta a ponta que simulam um ambiente completo do navegador.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão da React Testing Library tem suporte integrado para React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e as alterações relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Parando o Suporte ao Internet Explorer {/*dropping-support-for-internet-explorer*/}

Neste lançamento, o React está parando o suporte ao Internet Explorer, que está [saindo do suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa alteração agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtarefas, que não podem ser adequadamente polifilladas no IE.

Se você precisar oferecer suporte ao Internet Explorer, recomendamos que você fique com o React 17.

## Depreciações {/*deprecations*/}

*   `react-dom`: `ReactDOM.render` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
*   `react-dom`: `ReactDOM.hydrate` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
*   `react-dom`: `ReactDOM.unmountComponentAtNode` foi descontinuado.
*   `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi descontinuado.
*   `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi descontinuado.

## Outras Mudanças Disruptivas {/*other-breaking-changes*/}

*   **Temporização consistente do useEffect**: o React agora sempre libera funções de efeito de forma síncrona se a atualização foi acionada durante um evento de entrada do usuário discreto, como um evento de clique ou keydown. Anteriormente, o comportamento nem sempre era previsível ou consistente.
*   **Erros de hidratação mais rigorosos**: incompatibilidades de hidratação devido a conteúdo de texto ausente ou extra agora são tratados como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente em uma tentativa de corresponder a marcação do servidor e reverterá para a renderização do cliente até o limite`<Suspense>`mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis buracos de privacidade e segurança que podem ser causados ​​por incompatibilidades de hidratação.
*   **Árvores Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto ou acionará seus efeitos. Em vez disso, o React descartará a nova árvore completamente, aguardará a conclusão da operação assíncrona e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
*   **Layout Effects com Suspense**: Quando uma árvore re-suspende e reverte para um fallback, o React agora limpa os efeitos de layout e, em seguida, os recria quando o conteúdo dentro do limite é mostrado novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o layout quando usavam Suspense.
*   **Novos Requisitos de Ambiente JS**: o React agora depende de recursos de navegadores modernos, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos de navegador modernos nativamente ou têm implementações não compatíveis, considere incluir um polyfill global em seu aplicativo agrupado.

## Outras Mudanças Notáveis ​​ {/*other-notable-changes*/}

### React {/*react*/}

*   **Componentes agora podem renderizar `undefined`:** o React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno do componente permitidos consistentes com os valores permitidos no meio de uma árvore de componentes. Sugerimos usar um linter para evitar erros como esquecer uma instrução `return` antes de JSX.
*   **Em testes, os avisos `act` agora são opcionais:** Se você estiver executando testes de ponta a ponta, os avisos `act` são desnecessários. Apresentamos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade, onde eles são úteis e benéficos.
*   **Nenhum aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chama `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas principalmente o encontram em cenários em que a configuração do estado é boa, e soluções alternativas tornam o código pior. Nós [removemos](https://github.com/facebook/react/pull/22114) este aviso.
*   **Nenhuma supressão de logs do console:** Quando você usa o Modo Estrito, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para facilitar a leitura dos logs. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo log serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
*   **Uso de memória aprimorado:** o React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos severo.

### React DOM Server {/*react-dom-server*/}

*   **`renderToString`:** Não irá mais apresentar erros ao suspender no servidor. Em vez disso, ele emitirá o HTML fallback para o limite `<Suspense>` mais próximo e, em seguida, tentará renderizar o mesmo conteúdo no cliente. Ainda é recomendado que você mude para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream` em vez disso.
*   **`renderToStaticMarkup`:** Não irá mais apresentar erros ao suspender no servidor. Em vez disso, ele emitirá o HTML fallback para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode ver o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).

```bash
npm install react react-dom
```

```bash
npm install react react-dom
```

```bash
yarn add react react-dom
```

```bash
yarn add react react-dom
```

```js
// Before
import { render } from 'react-dom';
const container = document.getElementById('app');
render(<App tab="home" />, container);

// After
import { createRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = createRoot(container); // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />);
```

```js
// Antes
import { render } from 'react-dom';
const container = document.getElementById('app');
render(<App tab="home" />, container);

// Depois
import { createRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = createRoot(container); // createRoot(container!) se você usar TypeScript
root.render(<App tab="home" />);
```

```js
// Before
unmountComponentAtNode(container);

// After
root.unmount();
```

```js
// Antes
unmountComponentAtNode(container);

// Depois
root.unmount();
```

```js
// Before
const container = document.getElementById('app');
render(<App tab="home" />, container, () => {
  console.log('rendered');
});

// After
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log('rendered');
  });

  return <App tab="home" />
}

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

```js
// Antes
const container = document.getElementById('app');
render(<App tab="home" />, container, () => {
  console.log('rendered');
});

// Depois
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log('rendered');
  });

  return <App tab="home" />
}

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

```js
// Before
import { hydrate } from 'react-dom';
const container = document.getElementById('app');
hydrate(<App tab="home" />, container);

// After
import { hydrateRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = hydrateRoot(container, <App tab="home" />);
// Unlike with createRoot, you don't need a separate root.render() call here.
```

```js
// Antes
import { hydrate } from 'react-dom';
const container = document.getElementById('app');
hydrate(<App tab="home" />, container);

// Depois
import { hydrateRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = hydrateRoot(container, <App tab="home" />);
// Diferente do createRoot, você não precisa de uma chamada separada root.render() aqui.
```

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

```js
// Before React 18 only React events were batched

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will render twice, once for each state update (no batching)
}, 1000);
```

```js
// Antes do React 18 apenas os eventos React eram batching

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // O React só vai re-renderizar uma vez por fim (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // O React vai renderizar duas vezes, uma para cada atualização de estado (não há batching)
}, 1000);
```

```js
// After React 18 updates inside of timeouts, promises,
// native event handlers or any other event are batched.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

```js
// Depois do React 18 atualizações dentro de timeouts, promises,
// manipuladores de eventos nativos ou qualquer outro evento são batching.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar uma vez só no fim (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar uma vez só no fim (isso é batching!)
}, 1000);
```

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React atualizou o DOM agora
  flushSync(() => {
    setFlag(f => !f);
  });
  // React atualizou o DOM agora
}
```

```
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
```

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
```

```
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
* React simulates unmounting the component.
    * Layout effects are destroyed.
    * Effects are destroyed.
* React simulates mounting the component with the previous state.
    * Layout effect setup code runs
    * Effect setup code runs
```

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
* React simula a desmontagem do componente.
    * Efeitos de layout são destruídos.
    * Efeitos são destruídos.
* React simula a montagem do componente com o estado anterior.
    * Executa o código de configuração do efeito de layout
    * Executa o código de configuração do efeito
```

```js
// In your test setup file
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

```js
// No seu arquivo de configuração de teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```