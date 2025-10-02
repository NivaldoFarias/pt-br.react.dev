---
title: "Como atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Conforme compartilhamos na publicação de lançamento, o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Nesta publicação, iremos guiá-lo pelas etapas para atualizar para o React 18.
---

08 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Conforme compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 apresenta recursos alimentados por nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Nesta publicação, iremos guiá-lo pelas etapas para atualizar para o React 18.

Por favor, [relate quaisquer problemas](https://github.com/facebook/react/issues/new/choose) que encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 depende da Nova Arquitetura do React Native para se beneficiar dos novos recursos apresentados nesta postagem do blog. Para obter mais informações, consulte a [palestra principal da React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

---

## Instalando {/*installing*/}

Para instalar a versão mais recente do React:

```bash
npm install react react-dom
```

Ou se você estiver usando o yarn:

```bash
yarn add react react-dom
```

## Atualizações para APIs de Renderização do Cliente {/*updates-to-client-rendering-apis*/}

Ao instalar o React 18 pela primeira vez, você verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 apresenta uma nova root API que oferece melhor ergonomia para gerenciar roots. A nova root API também habilita o novo renderizador concorrente, que permite que você opte por recursos concorrentes.

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

Também mudamos `unmountComponentAtNode` para `root.unmount`:

```js
// Antes
unmountComponentAtNode(container);

// Depois
root.unmount();
```

Também removemos o *callback* de *render*, uma vez que geralmente não tem o resultado esperado ao usar Suspense:

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

Não há substituição direta para a antiga API de *callback* de *render* — depende do seu caso de uso. Veja a publicação do grupo de trabalho para [Substituindo *render* por createRoot](https://github.com/reactwg/react-18/discussions/5) para mais informações.

</Note>

Finalmente, se seu aplicativo usa renderização do lado do servidor com *hydration*, atualize `hydrate` para `hydrateRoot`:

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

**Se o seu aplicativo não funcionar após a atualização, verifique se ele está envolvido em `<StrictMode>`.** O [Modo Estrito ficou mais rigoroso no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Modo Estrito corrigir seu aplicativo, você pode removê-lo durante a atualização e, em seguida, adicioná-lo novamente (na parte superior ou para uma parte da árvore) depois de corrigir os problemas que ele está apontando.

</Note>

## Atualizações para APIs de Renderização do Lado do Servidor {/*updates-to-server-rendering-apis*/}

Neste lançamento, estamos reformulando nossas APIs `react-dom/server` para oferecer suporte total ao Suspense no servidor e ao Streaming SSR. Como parte dessas mudanças, estamos depreciando a antiga API de *streaming* Node, que não suporta *streaming* incremental do Suspense no servidor.

Usar esta API agora avisará:

* `renderToNodeStream`: **Obsoleto ⛔️️**

Em vez disso, para *streaming* em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Também estamos apresentando uma nova API para dar suporte ao *streaming* SSR com Suspense para ambientes de tempo de execução de *edge* modernos, como Deno e *workers* do Cloudflare:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado ao Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
* `renderToStaticNodeStream`

Para obter mais informações sobre as mudanças nas APIs de renderização do servidor, consulte a postagem do grupo de trabalho sobre [Atualizando para o React 18 no servidor](https://github.com/reactwg/react-18/discussions/22), um [mergulho profundo na nova Arquitetura SSR do Suspense](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Renderização de Servidor de *Streaming* com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Atualizações para definições do TypeScript {/*updates-to-typescript-definitions*/}

Se o seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e capturam problemas que costumavam ser ignorados pelo verificador de tipo. A mudança mais notável é que a *prop* `children` agora precisa ser listada explicitamente ao definir as *props*, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Veja o [*pull request* de tipagem do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para uma lista completa de mudanças somente de tipo. Ele se vincula a exemplos de correções em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código do seu aplicativo para as tipagens novas e mais seguras mais rapidamente.

Se você encontrar um erro nas tipagens, por favor, [registre um problema](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Batching Automático {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho *out-of-the-box* fazendo mais *batching* por padrão. *Batching* é quando o React agrupa várias atualizações de *state* em uma única *re-renderização* para melhor desempenho. Antes do React 18, só fazíamos *batching* de atualizações dentro de *event handlers* do React. As atualizações dentro de *promises*, setTimeout, *native event handlers* ou qualquer outro evento não eram *batched* no React por padrão:

```js
// Antes do React 18, apenas eventos React eram batched

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React só vai re-renderizar uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar duas vezes, uma para cada atualização de state (sem batching)
}, 1000);
```


A partir do React 18 com `createRoot`, todas as atualizações serão automaticamente *batched*, não importa de onde se originem. Isso significa que as atualizações dentro de *timeouts*, *promises*, *native event handlers* ou qualquer outro evento serão *batched* da mesma forma que as atualizações dentro de eventos React:

```js
// Após o React 18, as atualizações dentro de timeouts, promises,
// native event handlers ou qualquer outro evento são batched.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React só vai re-renderizar uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React só vai re-renderizar uma vez no final (isso é batching!)
}, 1000);
```

Esta é uma mudança radical, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para desativar o *batching* automático, você pode usar `flushSync`:

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React já atualizou o DOM
  flushSync(() => {
    setFlag(f => !f);
  });
  // React já atualizou o DOM
}
```

Para obter mais informações, consulte o [aprofundamento do *batching* automático](https://github.com/reactwg/react-18/discussions/21).

## Novas APIs para Bibliotecas {/*new-apis-for-libraries*/}

No Grupo de Trabalho do React 18, trabalhamos com mantenedores de bibliotecas para criar novas APIs necessárias para dar suporte à renderização concorrente para casos de uso específicos para seu caso de uso em áreas como estilos e *stores* externos. Para dar suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

*   `useSyncExternalStore` é um novo Hook que permite que *stores* externos suportem leituras simultâneas, forçando as atualizações no *store* a serem síncronas. Esta nova API é recomendada para qualquer biblioteca que se integre com o *state* externo ao React. Para obter mais informações, consulte a [postagem de visão geral do useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e os [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
*   `useInsertionEffect` é um novo Hook que permite que as bibliotecas CSS-in-JS abordem problemas de desempenho da injeção de estilos em *render*. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após a mutação do DOM, mas antes que os efeitos de *layout* leiam o novo *layout*. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o *layout*. Para obter mais informações, consulte o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também apresenta novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais na [publicação de lançamento](/blog/2022/03/29/react-v18).

## Atualizações para o Modo Estrito {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permita ao React adicionar e remover seções da UI, preservando o *state*. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo *state* do componente de antes.

Este recurso dará ao React um melhor desempenho *out-of-the-box*, mas exige que os componentes sejam resilientes aos efeitos sendo montados e destruídos várias vezes. A maioria dos efeitos funcionará sem quaisquer alterações, mas alguns efeitos supõem que eles são montados ou destruídos apenas uma vez.

Para ajudar a trazer à tona esses problemas, o React 18 apresenta uma nova verificação somente para desenvolvimento no Modo Estrito. Esta nova verificação irá desmontar e remontar automaticamente cada componente, sempre que um componente é montado pela primeira vez, restaurando o *state* anterior na segunda montagem.

Antes dessa mudança, o React montaria o componente e criaria os efeitos:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
```

Com o Modo Estrito no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
* React simula desmontar o componente.
    * Efeitos de layout são destruídos.
    * Efeitos são destruídos.
* React simula montar o componente com o state anterior.
    * O código de configuração do efeito de layout é executado
    * O código de configuração do efeito é executado
```

Para obter mais informações, consulte as postagens do Grupo de Trabalho para [Adicionando *State* Reutilizável ao Modo Estrito](https://github.com/reactwg/react-18/discussions/19) e [Como dar suporte ao *State* Reutilizável em Efeitos](https://github.com/reactwg/react-18/discussions/18).

## Configurando seu Ambiente de Teste {/*configuring-your-testing-environment*/}

Ao atualizar seus testes para usar `createRoot` pela primeira vez, você pode ver este aviso no seu console de teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para dar suporte ao act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// No seu arquivo de configuração de teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo do *flag* é dizer ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React irá registrar avisos úteis se você esquecer de envolver uma atualização com `act`.

Você também pode definir o *flag* como `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes *end-to-end* que simulam um ambiente de navegador completo.

Eventualmente, esperamos que as bibliotecas de teste configurem isso para você automaticamente. Por exemplo, a [próxima versão da React Testing Library tem suporte integrado para o React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem qualquer configuração adicional.

[Mais informações sobre a API de teste `act` e mudanças relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Descontinuando o suporte para o Internet Explorer {/*dropping-support-for-internet-explorer*/}

Neste lançamento, o React está descontinuando o suporte para o Internet Explorer, que está [saindo do suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa mudança agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtarefas, que não podem ser devidamente *polyfill* no IE.

Se você precisar dar suporte ao Internet Explorer, recomendamos que você permaneça com o React 17.

## Depreciações {/*deprecations*/}

*   `react-dom`: `ReactDOM.render` foi depreciado. Usá-lo avisará e executará seu aplicativo no modo React 17.
*   `react-dom`: `ReactDOM.hydrate` foi depreciado. Usá-lo avisará e executará seu aplicativo no modo React 17.
*   `react-dom`: `ReactDOM.unmountComponentAtNode` foi depreciado.
*   `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi depreciado.
*   `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi depreciado.

## Outras Mudanças Destrutivas {/*other-breaking-changes*/}

*   **Tempo consistente do useEffect**: O React agora sempre descarrega de forma síncrona as funções de efeito se a atualização foi acionada durante um evento de entrada do usuário discreto, como um clique ou um evento de tecla pressionada. Anteriormente, o comportamento nem sempre era previsível ou consistente.
*   **Erros de *hydration* mais rígidos**: Incompatibilidades de *hydration* devido ao conteúdo de texto ausente ou extra agora são tratadas como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente na tentativa de corresponder à marcação do servidor e reverterá para a renderização do cliente até o limite `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis falhas de privacidade e segurança que podem ser causadas por incompatibilidades de *hydration*.
*   **As árvores Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um *state* incompleto nem acionará seus efeitos. Em vez disso, o React descartará completamente a nova árvore, esperará que a operação assíncrona termine e, em seguida, tentará novamente a renderização do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
*   **Efeitos de *Layout* com Suspense:** Quando uma árvore volta a suspender e reverte para um *fallback*, o React agora limpará os efeitos de *layout* e os recriará quando o conteúdo dentro do limite for exibido novamente. Isso corrige um problema que impedia que as bibliotecas de componentes medissem corretamente o *layout* quando usadas com Suspense.
*   **Novos requisitos de ambiente JS**: O React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos modernos do navegador nativamente ou têm implementações não compatíveis, considere incluir um *polyfill* global em seu aplicativo agrupado.

## Outras Mudanças Notáveis {/*other-notable-changes*/}

### React {/*react*/}

*   **Os componentes agora podem renderizar `undefined`:** O React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componente permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos usar um *linter* para evitar erros como esquecer uma declaração `return` antes de JSX.
*   **Em testes, os avisos `act` agora são opcionais:** Se você estiver executando testes *end-to-end*, os avisos `act` são desnecessários. Introduzimos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade onde eles são úteis e benéficos.
*   **Sem aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chamava `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas encontram-no principalmente em cenários onde definir o *state* está bem, e as soluções alternativas pioram o código. Nós [removemos](https://github.com/facebook/react/pull/22114) este aviso.
*   **Nenhuma supressão de logs do console:** Quando você usa o Modo Estrito, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para tornar os logs mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as segundas renderizações do log serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
*   **Uso de memória aprimorado:** O React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos grave.

### React DOM Server {/*react-dom-server*/}

*   **`renderToString`:** Não apresentará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de *fallback* para o limite `<Suspense>` mais próximo e, em seguida, tentará novamente renderizar o mesmo conteúdo no cliente. Ainda é recomendado que você mude para uma API de *streaming* como `renderToPipeableStream` ou `renderToReadableStream`.
*   **`renderToStaticMarkup`:** Não apresentará mais erros ao suspender no servidor. Em vez disso, ele emitirá o HTML de *fallback* para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode ver o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).