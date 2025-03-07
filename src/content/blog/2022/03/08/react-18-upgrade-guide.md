---
title: "Como atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos no post do lançamento, o React 18 introduz recursos impulsionados pelo nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicações existentes. Neste post, iremos guiá-lo pelos passos para atualizar para o React 18.
---

08 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos no [post do lançamento](/blog/2022/03/29/react-v18), o React 18 introduz recursos impulsionados pelo nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicações existentes. Neste post, iremos guiá-lo pelos passos para atualizar para o React 18.

Por favor, [reporte quaisquer erros](https://github.com/facebook/react/issues/new/choose) que você encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso porque o React 18 depende da Nova Arquitetura do React Native para se beneficiar das novas funcionalidades apresentadas neste post de blog. Para mais informações, veja a [keynote do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

---

## Instalando {/*installing*/}

Para instalar a última versão do React:

```bash
npm install react react-dom
```

Ou se você estiver usando yarn:

```bash
yarn add react react-dom
```

## Updates to Client Rendering APIs {/*updates-to-client-rendering-apis*/}

Quando você instala a primeira vez o React 18, você verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu app se comportará como se estivesse executando o React 17. Aprenda mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 introduz uma nova API root que fornece melhor ergonomia para gerenciar roots. A nova API root também habilita o novo renderizador concorrente, o que permite que você opte por entrar em funcionalidades concorrentes.

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

Nós também mudamos `unmountComponentAtNode` para `root.unmount`:

```js
// Before
unmountComponentAtNode(container);

// After
root.unmount();
```

Nós também removemos o callback de render, já que normalmente não tem o resultado esperado quando usando Suspense:

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

Não há substituição one-to-one para a antiga API de callback de render — isso depende do seu caso de uso. Veja o post do grupo de trabalho para [Substituindo render com createRoot](https://github.com/reactwg/react-18/discussions/5) para mais informação.

</Note>

Finalmente, se seu app usa server-side rendering com hydration, atualize `hydrate` para `hydrateRoot`:

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

Para mais informação, veja a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu app não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** [O Strict Mode ficou mais rigoroso no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resistentes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Strict Mode corrigir seu app, você pode removê-lo durante a atualização e, em seguida, adicioná-lo de volta (seja no topo ou para uma parte da árvore) depois de corrigir os problemas apontados por ele.

</Note>

## Updates to Server Rendering APIs {/*updates-to-server-rendering-apis*/}

Nesta release, estamos reformulando nossas APIs `react-dom/server` para dar suporte total ao Suspense no servidor e Streaming SSR. Como parte dessas mudanças, estamos depreciando a antiga API de streaming Node, que não oferece suporte ao streaming incremental de Suspense no servidor.

Usar essa API agora avisará:

* `renderToNodeStream`: **Depreciado ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Nós também estamos introduzindo uma nova API para suportar streaming SSR com Suspense para ambientes de tempo de execução de ponta modernos, como Deno e Cloudflare workers:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
* `renderToStaticNodeStream`

Para mais informações sobre as mudanças nas APIs de renderização no servidor, veja o post do grupo de trabalho sobre [Atualizando para o React 18 no servidor](https://github.com/reactwg/react-18/discussions/22), um [mergulho profundo na nova Arquitetura de SSR com Suspense](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Updates to TypeScript definitions {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e detectam problemas que costumavam ser ignorados pelo verificador de tipo. A mudança mais notável é que a prop `children` agora precisa ser listada explicitamente ao definir as props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Veja a [pull request de typings do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para uma lista completa de mudanças somente de tipo. Ele vincula a exemplos de correções em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código do seu aplicativo para os novos e mais seguros typings mais rapidamente.

Se você encontrar um erro nos typings, por favor, [crie uma issue](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Automatic Batching {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais batching por padrão. Batching é quando o React agrupa múltiplas atualizações de state em uma única re-renderização para melhor desempenho. Antes do React 18, nós só fazíamos batch de atualizações dentro de event handlers do React. Atualizações dentro de promises, setTimeout, native event handlers, ou qualquer outro evento, não eram batchadas no React por padrão:

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

Começando no React 18 com `createRoot`, todas as atualizações serão automaticamente batchadas, não importa de onde elas se originem. Isso significa que as atualizações dentro de timeouts, promises, native event handlers ou qualquer outro evento serão batchadas da mesma forma que as atualizações dentro de eventos do React:

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

Esta é uma mudança breaking change, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para optar por não usar o batching automático, você pode usar `flushSync`:

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

Para mais informações, veja o [mergulho profundo em Automatic batching](https://github.com/reactwg/react-18/discussions/21).

## New APIs for Libraries {/*new-apis-for-libraries*/}

No React 18 Working Group, nós trabalhamos com os mantenedores de bibliotecas para criar novas APIs necessárias para suportar renderização concorrente para casos de uso específicos para seus casos de uso em áreas como styles e external stores. Para dar suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo Hook que permite que external stores suportem leituras concorrentes, forçando atualizações no store a serem síncronas. Essa nova API é recomendada para qualquer biblioteca que se integra com state externo ao React. Para mais informações, veja o [post de visão geral do useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de styles em render. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Esse Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para mais informações, veja o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também introduz novas APIs para renderização concorrente como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais no [post de lançamento](/blog/2022/03/29/react-v18).

## Updates to Strict Mode {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite ao React adicionar e remover seções da UI, preservando o state. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo component state de antes.

Esta funcionalidade dará ao React melhor desempenho imediato, mas exige que os componentes sejam resilientes à efeitos sendo montados e destruídos múltiplas vezes. A maioria dos efeitos funcionará sem nenhuma mudança, mas alguns efeitos assumem que eles só são montados ou destruídos uma vez.

Para ajudar a mostrar esses problemas, o React 18 introduz uma nova verificação somente para desenvolvimento no Strict Mode. Essa nova verificação irá automaticamente desmontar e remontar cada componente, sempre que um componente for montado pela primeira vez, restaurando o state anterior na segunda montagem.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
```

Com o Strict Mode no React 18, o React irá simular a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
* React simula a desmontagem do componente.
    * Efeitos de layout são destruídos.
    * Efeitos são destruídos.
* React simula a montagem do componente com o state anterior.
    * O código de configuração do efeito de layout roda
    * O código de configuração de efeito roda
```

Para mais informação, veja os posts do Working Group para [Adicionando State Reutilizável ao StrictMode](https://github.com/reactwg/react-18/discussions/19) e [Como dar suporte a State Reutilizável em Efeitos](https://github.com/reactwg/react-18/discussions/18).

## Configuring Your Testing Environment {/*configuring-your-testing-environment*/}

Quando você for atualizar seus testes para usar `createRoot`, você pode ver essa aviso no console do seu test:

<ConsoleBlock level="error">

O ambiente atual de testes não está configurado para dar suporte a act(...)

</ConsoleBlock>

Para consertar isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` para `true` antes de rodar seu teste:

```js
// In your test setup file
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O propósito da flag é dizer ao React que ele está rodando em um ambiente parecido com um teste de unidade. React irá logar avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir a flag para `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes end-to-end que simulam um ambiente de navegador completo.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão do React Testing Library tem suporte integrado ao React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem qualquer configuração adicional.

[Mais informações sobre a API de teste `act` e mudanças relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Dropping Support for Internet Explorer {/*dropping-support-for-internet-explorer*/}

Nesta release, o React está abandonando o suporte ao Internet Explorer, que [está saindo do suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa mudança agora porque novos recursos introduzidos no React 18 são construídos usando recursos de navegador modernos, como microtasks, que não podem ter polyfill adequado no IE.

Se você precisar suportar o Internet Explorer, recomendamos que você fique com o React 17.

## Deprecations {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi depreciado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi depreciado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi depreciado.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi depreciado.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi depreciado.

## Other Breaking Changes {/*other-breaking-changes*/}

*   **Consistent useEffect timing**: React agora sempre sincronamente flui funções de efeito se a atualização foi acionada durante um evento de entrada do usuário discreto, como um evento de clique ou tecla pressionada. Anteriormente, o comportamento nem sempre era previsível ou consistente.
*   **Stricter hydration errors**: Hydration mismatches devido a conteúdo de texto ausente ou extra agora são tratados como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente na tentativa de corresponder à marcação do servidor e voltará à renderização do cliente até o limite do `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis buracos de privacidade e segurança que podem ser causados por erros de hidratação.
*   **Suspense trees are always consistent:** Se um componente suspender antes que ele seja totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto nem acionará seus efeitos. Em vez disso, o React descartará completamente a nova árvore, esperará que a operação assíncrona seja concluída e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
*   **Layout Effects with Suspense**: Quando uma árvore re-suspende e reverte para um fallback, o React agora limpa os efeitos de layout e, em seguida, os recria quando o conteúdo dentro do limite é mostrado novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o layout quando usadas com Suspense.
*   **New JS Environment Requirements**: React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos modernos do navegador nativamente ou têm implementações não compatíveis, considere incluir um polyfill global em seu aplicativo em bundle.

## Other Notable Changes {/*other-notable-changes*/}

### React {/*react*/}

*   **Components can now render `undefined`:** React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componente permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos o uso de um linter para evitar erros como esquecer uma instrução `return` antes do JSX.
*   **In tests, `act` warnings are now opt-in:** Se você está executando testes end-to-end, os avisos de `act` são desnecessários. Introduzimos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade onde são úteis e benéficos.
*   **No warning about `setState` on unmounted components:** Anteriormente, o React avisava sobre vazamentos de memória quando você chamava `setState` em um componente desmontado. Esse aviso foi adicionado para assinaturas, mas as pessoas geralmente se deparam com ele em cenários em que definir o state é bom, e soluções alternativas tornam o código pior. Nós [removemos](https://github.com/facebook/react/pull/22114) este aviso.
*   **No suppression of console logs:** Quando você usa o Strict Mode, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para tornar os logs mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver as React DevTools instaladas, as renderizações do segundo log serão exibidas em cinza, e haverá uma opção (desativada por padrão) para suprimi-los completamente.
*   **Improved memory usage:** React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos severo.

### React DOM Server {/*react-dom-server*/}

*   **`renderToString`:** Não irá mais gerar erros ao suspender no servidor. Ele emitirá a HTML de fallback para o limite do `<Suspense>` mais próximo e, em seguida, tentará renderizar novamente o mesmo conteúdo no cliente. Ainda é recomendado que você mude para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream`.
*   **`renderToStaticMarkup`:** Não irá mais gerar erros ao suspender no servidor. Em vez disso, ele emitirá a HTML de fallback para o limite do `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode ver o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).