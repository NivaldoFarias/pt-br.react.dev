---
title: "Como atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos na publicação de lançamento, o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, guiaremos você pelas etapas para atualizar para o React 18.
---

08 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, guiaremos você pelas etapas para atualizar para o React 18.

Por favor, [relate quaisquer erros](https://github.com/facebook/react/issues/new/choose) que você encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para os usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 depende da Nova Arquitetura do React Native para se beneficiar dos novos recursos apresentados nesta publicação do blog. Para mais informações, consulte a [apresentação principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

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

## Atualizações nas APIs de Renderização do Cliente {/*updates-to-client-rendering-apis*/}

Quando você instala o React 18 pela primeira vez, verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use o createRoot em vez disso. Até que você mude para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 apresenta uma nova API root que oferece melhor ergonomia para o gerenciamento de roots. A nova API root também habilita o novo renderizador concorrente, que permite que você opte por recursos concorrentes.

```js
// Antes
import { render } from 'react-dom';
const container = document.getElementById('app');
render(<App tab="home" />, container);

// Depois
import { createRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = createRoot(container); // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />);
```

Também mudamos `unmountComponentAtNode` para `root.unmount`:

```js
// Antes
unmountComponentAtNode(container);

// Depois
root.unmount();
```

Também removemos o callback de render, pois ele geralmente não tem o resultado esperado ao usar o Suspense:

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

Não existe um substituto um para um para a API de callback de render antiga — ela depende do seu caso de uso. Consulte a postagem do grupo de trabalho para [Substituindo render com createRoot](https://github.com/reactwg/react-18/discussions/5) para mais informações.

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
// Unlike with createRoot, you don't need a separate root.render() call here.
```

Para mais informações, consulte a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu aplicativo não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** [O Strict Mode ficou mais rigoroso no React 18](#updates-to-strict-mode) e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Strict Mode corrigir seu aplicativo, você pode removê-lo durante a atualização e, em seguida, adicioná-lo de volta (no topo ou para uma parte da árvore) depois de corrigir os problemas que ele está apontando.

</Note>

## Atualizações nas APIs de Renderização do Servidor {/*updates-to-server-rendering-apis*/}

Neste lançamento, estamos renovando nossas APIs `react-dom/server` para oferecer suporte total ao Suspense no servidor e Streaming SSR. Como parte dessas mudanças, estamos descontinuando a antiga API de streaming do Node, que não oferece suporte ao streaming incremental do Suspense no servidor.

O uso dessa API agora emitirá um aviso:

* `renderToNodeStream`: **Obsoleto ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Também estamos introduzindo uma nova API para oferecer suporte ao Streaming SSR com Suspense para ambientes de tempo de execução de ponta modernos, como Deno e Cloudflare workers:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
* `renderToStaticNodeStream`

Para obter mais informações sobre as alterações nas APIs de renderização do servidor, consulte a postagem do grupo de trabalho sobre [Como atualizar para o React 18 no servidor](https://github.com/reactwg/react-18/discussions/22), uma [análise aprofundada da nova Arquitetura SSR do Suspense](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) em [Streaming Server Rendering with Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) no React Conf 2021.

## Atualizações nas definições do TypeScript {/*updates-to-typescript-definitions*/}

Se o seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e detectam problemas que costumavam ser ignorados pelo verificador de tipos. A mudança mais notável é que a prop `children` agora precisa ser listada explicitamente ao definir props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Consulte a [solicitação de pull de tipagem do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para obter uma lista completa de alterações somente de tipo. Ele vincula correções de exemplo nos tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a converter o código do seu aplicativo para as tipagens novas e mais seguras mais rapidamente.

Se você encontrar um erro nas tipagens, por favor, [crie um problema](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Batching Automático {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais batching por padrão. Batching é quando o React agrupa várias atualizações de state em uma única re-renderização para melhor desempenho. Antes do React 18, só fazíamos batch de atualizações dentro dos manipuladores de eventos React. As atualizações dentro de promises, setTimeout, manipuladores de eventos nativos ou qualquer outro evento não eram agrupadas no React por padrão:

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

A partir do React 18 com `createRoot`, todas as atualizações serão agrupadas automaticamente, não importa de onde se originem. Isso significa que as atualizações dentro de timeouts timers, promises, manipuladores de eventos nativos ou qualquer outro evento serão agrupadas da mesma forma que as atualizações dentro dos eventos React:

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

Esta é uma mudança disruptiva (breaking change), mas esperamos que isso resulte em menos trabalho de render, e, portanto, melhor desempenho em seus aplicativos. Para optar por não usar o batching automático, você pode usar `flushSync`:

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

Para mais informações, consulte a [análise aprofundada do batching automático](https://github.com/reactwg/react-18/discussions/21).

## Novas APIs para Bibliotecas {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os mantenedores da biblioteca para criar novas APIs necessárias para oferecer suporte a renderização concorrente para casos de uso específicos para seus casos de uso em áreas como estilos e stores externos. Para dar suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo Hook que permite que stores externos suportem leituras concorrentes, forçando as atualizações no store a serem síncronas. Essa nova API é recomendada para qualquer biblioteca que se integra ao state externo ao React. Para mais informações, consulte a [postagem de visão geral do useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo Hook que permite que as bibliotecas CSS-in-JS abordem problemas de desempenho da injeção de estilos na renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você a use. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede para o navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para mais informações, consulte o [Guia de Atualização da Biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também apresenta novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais no [post de lançamento](/blog/2022/03/29/react-v18).

## Atualizações no Strict Mode {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite que o React adicione e remova seções da UI, preservando o state. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria as árvores usando o mesmo componente state de antes.

Esse recurso dará ao React melhor desempenho pronto para uso, mas requer que os componentes sejam resilientes a efeitos que estão sendo montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos pressupõem que eles só são montados ou destruídos uma vez.

Para ajudar a detectar esses problemas, o React 18 introduz uma nova verificação somente para desenvolvimento no Strict Mode. Essa nova verificação irá desmontar e remontar automaticamente cada componente sempre que um componente for montado pela primeira vez, restaurando o state anterior na segunda montagem.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:

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

Para obter mais informações, consulte as postagens do Grupo de Trabalho para [Adicionando State Reutilizável ao StrictMode](https://github.com/reactwg/react-18/discussions/19) e [Como oferecer suporte ao State Reutilizável em Efeitos](https://github.com/reactwg/react-18/discussions/18).

## Configurando seu Ambiente de Teste {/*configuring-your-testing-environment*/}

Quando você atualizar seus testes pela primeira vez para usar `createRoot`, poderá ver este aviso no console do teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para dar suporte a act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// No arquivo de configuração do seu teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo do flag é informar ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir o flag como `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes de ponta a ponta que simulam um ambiente de navegador completo.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão da React Testing Library tem suporte integrado para React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e alterações relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Parando o Suporte para o Internet Explorer {/*dropping-support-for-internet-explorer*/}

Neste lançamento, o React está parando o suporte ao Internet Explorer, que [vai parar de ser suportado em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa alteração agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtasks, que não podem ser adequadamente preenchidos no IE.

Se você precisar dar suporte ao Internet Explorer, recomendamos que você fique com o React 17.

## Depreciações {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi descontinuado.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi descontinuado.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi descontinuado.

## Outras Mudanças Disruptivas {/*other-breaking-changes*/}

* **Temporização consistentes de useEffect**: O React agora sempre libera as funções de efeito de forma síncrona se a atualização foi acionada durante um evento de entrada de usuário discreto, como um clique ou um evento keydown. Anteriormente, o comportamento nem sempre era previsível ou consistente.
* **Erros de hidratação mais rigorosos**: Incompatibilidades de hidratação devido a conteúdo textual ausente ou extra agora são tratadas como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente em uma tentativa de corresponder a marcação do servidor e reverterá para renderização do cliente até o limite `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis falhas de privacidade e segurança que podem ser causadas por incompatibilidades de hidratação.
* **Árvores do Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um state incompleto ou disparará seus efeitos. Em vez disso, o React descartará completamente a nova árvore, esperará até que a operação assíncrona termine e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
* **Efeitos de layout com Suspense**: Quando uma árvore reassume e reverte para um fallback, o React agora limpará os efeitos de layout e, em seguida, os recriará quando o conteúdo dentro do limite for exibido novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o layout quando usado com Suspense.
* **Novos requisitos de ambiente JS**: O React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos modernos de navegador nativamente ou têm implementações não compatíveis, considere incluir um polyfill global em seu aplicativo empacotado.

## Outras Mudanças Notáveis {/*other-notable-changes*/}

### React {/*react*/}

* **Componentes agora podem renderizar `undefined`:** O React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componentes permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos o uso de um linter para evitar erros como esquecer uma instrução `return` antes do JSX.
* **Em testes, os avisos `act` agora são opt-in:** Se você estiver executando testes de ponta a ponta, os avisos `act` são desnecessários. Apresentamos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade onde são úteis e benéficos.
* **Nenhum aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chama `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas encontram principalmente em cenários onde definir state é bom, e soluções alternativas tornam o código pior. [Removemos](https://github.com/facebook/react/pull/22114) este aviso.
* **Sem supressão de logs do console:** Quando você usa o Strict Mode, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para tornar os logs mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo log serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
* **Uso de memória aprimorado:** O React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos grave.

### React DOM Server {/*react-dom-server*/}

* **`renderToString`:** Não gerará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo e, em seguida, tentará renderizar o mesmo conteúdo no cliente novamente. Ainda é recomendado que você mude para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream` em vez disso.
* **`renderToStaticMarkup`:** Não gerará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode visualizar o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).
```