---
title: "Como atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos na publicação de lançamento, o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicações existentes. Nesta publicação, guiaremos você pelas etapas para atualizar para o React 18.
---

08 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicações existentes. Nesta publicação, guiaremos você pelas etapas para atualizar para o React 18.

Por favor, [relate qualquer erro](https://github.com/facebook/react/issues/new/choose) que você encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 confia na New React Native Architecture para se beneficiar dos novos recursos apresentados nesta postagem do blog. Para mais informações, consulte a [keynote do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

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

Quando você instala o React 18 pela primeira vez, verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu app vai se comportar como se estivesse rodando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 introduz uma nova API de raiz que fornece melhor ergonomia para o gerenciamento de raízes. A nova API de raiz também habilita o novo renderizador concorrente, que permite que você opte por usar recursos concorrentes.

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

Também removemos o callback de render, já que normalmente não tem o resultado esperado quando se usa Suspense:

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

Não há uma substituição um-para-um para a antiga API de callback de render — isso depende do seu caso de uso. Veja a publicação do grupo de trabalho para [Replacing render with createRoot](https://github.com/reactwg/react-18/discussions/5) para mais informações.

</Note>

Finalmente, se seu app usa renderização do lado do servidor com hidratação, atualize `hydrate` para `hydrateRoot`:

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

Para mais informações, veja a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu app não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** O [Strict Mode ficou mais restrito no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Strict Mode corrigir seu app, você pode removê-lo durante a atualização e, em seguida, adicioná-lo de volta (no topo ou em parte da árvore) após corrigir os problemas que ele está apontando.

</Note>

## Updates to Server Rendering APIs {/*updates-to-server-rendering-apis*/}

Nesta versão, estamos renovando nossas APIs `react-dom/server` para dar suporte total ao Suspense no servidor e Streaming SSR. Como parte dessas mudanças, estamos descontinuando a antiga API de streaming do Node, que não suporta o streaming incremental do Suspense no servidor.

Usar essa API agora avisará:

* `renderToNodeStream`: **Obsoleto ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Também estamos apresentando uma nova API para dar suporte ao streaming SSR com Suspense para ambientes de tempo de execução de ponta modernos, como Deno e Cloudflare workers:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderização de e-mails:
* `renderToStaticNodeStream`

Para mais informações sobre as mudanças nas APIs de renderização do servidor, consulte a publicação do grupo de trabalho sobre [Upgrading to React 18 on the server](https://github.com/reactwg/react-18/discussions/22), uma [análise aprofundada da nova arquitetura Suspense SSR](https://github.com/reactwg/react-18/discussions/37), e a apresentação de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering with Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Updates to TypeScript definitions {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos types são mais seguros e pegam problemas que costumavam ser ignorados pelo verificador de type. A mudança mais notável é que a prop `children` agora precisa ser listada explicitamente ao definir props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Veja a [pull request do React 18 typings](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para uma lista completa de mudanças apenas de type. Ela vincula exemplos de correções em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizada](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código da sua aplicação para os novos e mais seguros typings mais rapidamente.

Se você encontrar um erro nos typings, por favor, [envie um problema](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Automatic Batching {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais batching por padrão. Batching é quando o React agrupa múltiplas atualizações de state em um único re-render para melhor desempenho. Antes do React 18, nós só fazíamos batch das atualizações dentro dos manipuladores de eventos React. As atualizações dentro de promises, setTimeout, manipuladores de eventos nativos, ou qualquer outro evento não eram feitas em batch no React por padrão:

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

Começando no React 18 com `createRoot`, todas as atualizações serão automaticamente feitas em batch, não importa de onde elas se originam. Isso significa que as atualizações dentro de timeouts, promises, manipuladores de eventos nativos ou qualquer outro evento farão batch da mesma forma que as atualizações dentro dos eventos React:

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

Esta é uma mudança que quebra, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em suas aplicações. Para optar por não usar o batching automático, você pode usar `flushSync`:

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

Para mais informações, veja o [Automatic batching deep dive](https://github.com/reactwg/react-18/discussions/21).

## New APIs for Libraries {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os mantenedores de bibliotecas para criar novas APIs necessárias para dar suporte à renderização concorrente para casos de uso específicos em áreas como estilos e lojas externas. Para dar suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo Hook que permite que lojas externas deem suporte a leituras concorrentes, forçando as atualizações da loja a serem síncronas. Esta nova API é recomendada para qualquer biblioteca que se integre ao state externo ao React. Para mais informações, veja a [publicação de visão geral do useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e os [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em render. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede para o navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para mais informações, veja o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também introduz novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais na [publicação de lançamento](/blog/2022/03/29/react-v18).

## Updates to Strict Mode {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite que o React adicione e remova seções da UI, preservando o state. Por exemplo, quando um usuário sai de uma tela com a guia e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo state do componente de antes.

Este recurso dará ao React melhor desempenho pronto para uso, mas requer que os componentes sejam resilientes aos efeitos de serem montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos presumem que eles são montados ou destruídos apenas uma vez.

Para ajudar a apresentar esses problemas, o React 18 apresenta uma nova verificação somente para desenvolvimento no Strict Mode. Esta nova verificação irá automaticamente desmontar e remontar cada componente, sempre que um componente for montado pela primeira vez, restaurando o state anterior na segunda montagem.

Antes dessa mudança, o React montaria o componente e criaria os efeitos:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos são criados.
```

Com o Strict Mode no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos são criados.
* React simula a desmontagem do componente.
    * Efeitos de layout são destruídos.
    * Efeitos são destruídos.
* React simula a montagem do componente com o state anterior.
    * O código de configuração do efeito de layout é executado
    * O código de configuração do efeito é executado
```

Para mais informações, veja as publicações do Grupo de Trabalho para [Adding Reusable State to StrictMode](https://github.com/reactwg/react-18/discussions/19) e [How to support Reusable State in Effects](https://github.com/reactwg/react-18/discussions/18).

## Configuring Your Testing Environment {/*configuring-your-testing-environment*/}

Ao atualizar pela primeira vez seus testes para usar `createRoot`, você pode ver este aviso no console do seu teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para dar suporte ao act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar o teste:

```js
// In your test setup file
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo da flag é informar ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de envolver uma atualização com `act`.

Você também pode definir a flag como `false` para informar ao React que `act` não é necessário. Isso pode ser útil para testes de ponta a ponta que simulam um ambiente completo de navegador.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão do React Testing Library tem suporte integrado para React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e mudanças relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Dropping Support for Internet Explorer {/*dropping-support-for-internet-explorer*/}

Nesta versão, o React está removendo o suporte para o Internet Explorer, que está [saindo do suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa alteração agora porque os novos recursos introduzidos no React 18 são construídos com recursos modernos do navegador, como microtarefas, que não podem ser adequadamente polyfilled no IE.

Se você precisar dar suporte ao Internet Explorer, recomendamos que você fique com o React 17.

## Deprecations {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi descontinuado. Usá-lo irá avisar e executar seu app no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi descontinuado. Usá-lo irá avisar e executar seu app no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi descontinuado.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi descontinuado.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi descontinuado.

## Other Breaking Changes {/*other-breaking-changes*/}

* **Consistent useEffect timing**: O React agora sempre libera funções de efeito de forma síncrona se a atualização foi acionada durante um evento de entrada do usuário discreto, como um evento de clique ou keydown. Anteriormente, o comportamento nem sempre era previsível ou consistente.
* **Erros de hidratação mais rígidos**: Desajustes de hidratação devido a conteúdo de texto ausente ou extra agora são tratados como erros em vez de avisos. O React não tentará mais "remendar" nós individuais inserindo ou excluindo um nó no cliente em uma tentativa de corresponder a marcação do servidor, e reverterá para a renderização do cliente até o limite `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis buracos de privacidade e segurança que podem ser causados por desajustes de hidratação.
* **Árvores Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto nem acionará seus efeitos. Em vez disso, o React descartará a nova árvore completamente, aguardará a conclusão da operação assíncrona e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
* **Layout Effects com Suspense**: Quando uma árvore ressuspende e reverte para um fallback, o React agora irá limpar os efeitos de layout, e então recriá-los quando o conteúdo dentro do limite for mostrado novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o layout quando usado com Suspense.
* **Novos requisitos de ambiente JS**: O React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferecer suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos de navegador modernos nativamente ou têm implementações não compatíveis, considere incluir um polyfill global em seu aplicativo empacotado.

## Other Notable Changes {/*other-notable-changes*/}

### React {/*react*/}

* **Componentes agora podem renderizar `undefined`:** O React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componente permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos usar um linter para evitar erros, como esquecer uma declaração `return` antes do JSX.
* **Em testes, os avisos `act` agora são opcionais:** Se você estiver executando testes de ponta a ponta, os avisos `act` são desnecessários. Apresentamos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade, onde eles são úteis e benéficos.
* **Sem aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chama `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas encontram principalmente em cenários em que a configuração do state é boa, e as soluções alternativas pioram o código. Nós [removemos](https://github.com/facebook/react/pull/22114) este aviso.
* **Sem supressão de console logs:** Quando você usa o Strict Mode, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os registros de console para uma das duas renderizações para tornar os registros mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo log serão exibidas em cinza, e haverá uma opção (desativada por padrão) para suprimi-las completamente.
* **Uso de memória aprimorado:** O React agora limpa mais campos internos na desmontagem, tornando o impacto dos vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos severo.

### React DOM Server {/*react-dom-server*/}

* **`renderToString`:** Não irá mais falhar ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo e, em seguida, tentará renderizar o mesmo conteúdo no cliente. Ainda é recomendado que você mude para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream` em vez disso.
* **`renderToStaticMarkup`:** Não irá mais falhar ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode ver o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).
```