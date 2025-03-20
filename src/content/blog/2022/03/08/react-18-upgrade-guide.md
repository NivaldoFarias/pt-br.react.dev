---
title: "Como atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos na publicação de lançamento, o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, orientaremos você pelas etapas para atualizar para o React 18.
---

08 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, orientaremos você pelas etapas para atualizar para o React 18.

Por favor, [relate quaisquer problemas](https://github.com/facebook/react/issues/new/choose) que encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 depende da Nova Arquitetura do React Native para se beneficiar dos novos recursos apresentados nesta publicação do blog. Para obter mais informações, consulte a [apresentação principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

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

## Updates to Client Rendering APIs {/*updates-to-client-rendering-apis*/}

Ao instalar o React 18 pela primeira vez, você verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 apresenta uma nova API raiz que oferece melhor ergonomia para gerenciar raízes. A nova API raiz também habilita o novo renderizador concorrente, que permite que você opte por recursos concorrentes.

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

Também alteramos `unmountComponentAtNode` para `root.unmount`:

```js
// Antes
unmountComponentAtNode(container);

// Depois
root.unmount();
```

Também removemos o callback de render, pois geralmente ele não tem o resultado esperado ao usar Suspense:

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

<Note>

Não há uma substituição individual para a antiga API de callback de render — ela depende do seu caso de uso. Consulte a publicação do grupo de trabalho para [Replacing render with createRoot](https://github.com/reactwg/react-18/discussions/5) para obter mais informações.

</Note>

Finalmente, se seu aplicativo usa renderização do lado do servidor com hidratação, atualize `hydrate` para `hydrateRoot`:

```js
// Antes
import { hydrate } from 'react-dom';
const container = document.getElementById('app');
hydrate(<App tab="home" />, container);

// Depois
import { hydrateRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = hydrateRoot(container, <App tab="home" />);
// Ao contrário do createRoot, você não precisa de uma chamada root.render() separada aqui.
```

Para obter mais informações, consulte a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu aplicativo não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** [O Strict Mode ficou mais rigoroso no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Strict Mode corrigir seu aplicativo, você poderá removê-lo durante a atualização e, em seguida, adicioná-lo novamente (no topo ou para uma parte da árvore) depois de corrigir os problemas que ele está indicando.

</Note>

## Updates to Server Rendering APIs {/*updates-to-server-rendering-apis*/}

Nesta versão, estamos reformulando nossas APIs `react-dom/server` para oferecer suporte total ao Suspense no servidor e Streaming SSR. Como parte dessas alterações, estamos descontinuando a antiga API de streaming do Node, que não oferece suporte ao streaming incremental do Suspense no servidor.

Usar esta API agora avisará:

* `renderToNodeStream`: **Descontinuado ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Também estamos apresentando uma nova API para oferecer suporte ao streaming SSR com Suspense para ambientes de tempo de execução de borda modernos, como Deno e Cloudflare workers:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
* `renderToStaticNodeStream`

Para obter mais informações sobre as alterações nas APIs de renderização do servidor, consulte a publicação do grupo de trabalho sobre [Upgrading to React 18 on the server](https://github.com/reactwg/react-18/discussions/22), uma [análise aprofundada da nova Arquitetura SSR do Suspense](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering with Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Updates to TypeScript definitions {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e detectam problemas que costumavam ser ignorados pelo verificador de tipos. A alteração mais notável é que a propriedade `children` agora precisa ser listada explicitamente ao definir props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Consulte o [pull request de tipagens do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para obter uma lista completa de alterações somente de tipo. Ele vincula correções de exemplo em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código do seu aplicativo para as novas e mais seguras typings mais rapidamente.

Se você encontrar um erro nas tipagens, [crie um problema](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório do DefinitelyTyped.

## Automatic Batching {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais batching por padrão. Batching é quando o React agrupa várias atualizações de state em uma única re-renderização para melhor desempenho. Antes do React 18, nós só fazíamos batch updates dentro de event handlers do React. Updates dentro de promises, setTimeout, native event handlers ou qualquer outro evento não eram feitos em batch no React por padrão:

```js
// Antes do React 18 apenas eventos do React eram feitos em batch

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai re-renderizar apenas uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar duas vezes, uma vez para cada atualização de state (sem batching)
}, 1000);
```

A partir do React 18 com `createRoot`, todas as atualizações serão automaticamente processadas em batch, não importa de onde elas se originem. Isso significa que updates dentro de timeouts, promises, native event handlers ou qualquer outro evento terão batch da mesma forma que updates dentro de eventos do React:

```js
// Depois do React 18 updates dentro de timeouts, promises,
// native event handlers ou qualquer outro evento são feitos em batch.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai re-renderizar apenas uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai re-renderizar apenas uma vez no final (isso é batching!)
}, 1000);
```

Esta é uma mudança significativa, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para desativar o batching automático, você pode usar `flushSync`:

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

Para obter mais informações, consulte o [Automatic batching deep dive](https://github.com/reactwg/react-18/discussions/21).

## New APIs for Libraries {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os responsáveis pela manutenção de bibliotecas para criar novas APIs necessárias para oferecer suporte à renderização concorrente para casos de uso específicos em suas áreas, como estilos e stores externos. Para oferecer suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo Hook que permite que stores externos ofereçam suporte a leituras concorrentes, forçando as atualizações do store a serem síncronas. Esta nova API é recomendada para qualquer biblioteca que se integra ao state externo ao React. Para obter mais informações, consulte a postagem de visão geral [useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e os [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS resolvam problemas de desempenho de injeção de estilos na renderização. A menos que você já tenha criado uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para obter mais informações, consulte o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também apresenta novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre os quais compartilhamos mais na [publicação de lançamento](/blog/2022/03/29/react-v18).

## Updates to Strict Mode {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite que o React adicione e remova seções da interface do usuário, preservando o state. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo state do componente de antes.

Este recurso fornecerá ao React melhor desempenho imediato, mas exige que os componentes sejam resilientes a efeitos que são montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos presumem que eles são montados ou destruídos apenas uma vez.

Para ajudar a mostrar esses problemas, o React 18 apresenta uma nova verificação somente de desenvolvimento para o Strict Mode. Esta nova verificação desmontará e remontará automaticamente cada componente sempre que um componente for montado pela primeira vez, restaurando o state anterior na segunda montagem.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos são criados.
```

Com o Strict Mode no React 18, o React simulará a desmontagem e a remontagem do componente no modo de desenvolvimento:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos são criados.
* React simula a desmontagem do componente.
    * Efeitos de layout são destruídos.
    * Efeitos são destruídos.
* React simula a montagem do componente com o state anterior.
    * Código de configuração do efeito de layout é executado
    * Código de configuração do efeito é executado
```

Para obter mais informações, consulte as publicações do Grupo de Trabalho para [Adding Reusable State to StrictMode](https://github.com/reactwg/react-18/discussions/19) e [How to support Reusable State in Effects](https://github.com/reactwg/react-18/discussions/18).

## Configuring Your Testing Environment {/*configuring-your-testing-environment*/}

Ao atualizar seus testes para usar `createRoot`, você pode ver este aviso no console do teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para suportar act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// No seu arquivo de configuração de teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo da flag é informar ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir o flag como `false` para informar ao React que `act` não é necessário. Isso pode ser útil para testes ponta a ponta que simulam um ambiente completo do navegador.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão da React Testing Library tem suporte integrado ao React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e as alterações relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Dropping Support for Internet Explorer {/*dropping-support-for-internet-explorer*/}

Nesta versão, o React está abandonando o suporte ao Internet Explorer, que [estará fora de suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa alteração agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtasks, que não podem ser preenchidos adequadamente no IE.

Se você precisar oferecer suporte ao Internet Explorer, recomendamos que você permaneça com o React 17.

## Deprecations {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi descontinuado. Usá-lo irá avisar e executar seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi descontinuado. Usá-lo irá avisar e executar seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi descontinuado.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi descontinuado.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi descontinuado.

## Other Breaking Changes {/*other-breaking-changes*/}

* **Tempo consistente de useEffect**: o React agora sempre libera as funções de efeito de forma síncrona se a atualização foi acionada durante um evento de entrada do usuário discreto, como um evento de clique ou keydown. Anteriormente, o comportamento nem sempre era previsível ou consistente.
* **Erros de hidratação mais rigorosos**: incompatibilidades de hidratação devido à falta ou conteúdo de texto extra agora são tratados como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente na tentativa de corresponder à marcação do servidor, e reverterá para a renderização do cliente até o limite mais próximo de `<Suspense>` na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis falhas de privacidade e segurança que podem ser causadas por incompatibilidades de hidratação.
* **Árvores Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto ou disparará seus efeitos. Em vez disso, o React descartará a nova árvore completamente, aguardará a conclusão da operação assíncrona e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
* **Efeitos de layout com Suspense**: Quando uma árvore re-suspende e reverte para um fallback, o React agora limpará os efeitos de layout e, em seguida, os recriará quando o conteúdo dentro do limite for mostrado novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medirem corretamente o layout quando usadas com Suspense.
* **Novos requisitos de ambiente JS**: o React agora depende dos recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos modernos do navegador nativamente ou têm implementações não compatíveis, considere incluir um polyfill global em seu aplicativo empacotado.

## Other Notable Changes {/*other-notable-changes*/}

### React {/*react*/}

* **Componentes agora podem renderizar `undefined`:** React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componentes permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos usar um linter para evitar erros como esquecer uma instrução `return` antes do JSX.
* **Em testes, os avisos `act` agora são opcionais:** Se você estiver executando testes ponta a ponta, os avisos `act` são desnecessários. Apresentamos um mecanismo [opcional](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade onde eles são úteis e benéficos.
* **Sem aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chama `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas se deparam principalmente com ele em cenários em que definir o state está correto, e as soluções alternativas tornam o código pior. [Removemos](https://github.com/facebook/react/pull/22114) este aviso.
* **Sem supressão de logs do console:** Quando você usa o Strict Mode, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para facilitar a leitura dos logs. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo log serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
* **Melhor uso da memória:** O React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória irrelutantes que podem existir no código do seu aplicativo menos severo.

### React DOM Server {/*react-dom-server*/}

* **`renderToString`:** Não gerará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo e, em seguida, tentará renderizar novamente o mesmo conteúdo no cliente. Ainda é recomendável que você mude para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream`.
* **`renderToStaticMarkup`:** Não gerará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode ver o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).
``