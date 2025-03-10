---
title: "Como atualizar para React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos na publicação de lançamento, o React 18 introduz recursos baseados em nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, vamos guiá-lo pelas etapas para atualizar para o React 18.
---

Março 08, 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos na [publicação de lançamento](/blog/2022/03/29/react-v18), o React 18 introduz recursos baseados em nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, vamos guiá-lo pelas etapas para atualizar para o React 18.

Por favor, [relate quaisquer erros](https://github.com/facebook/react/issues/new/choose) que você encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 depende da Nova Arquitetura do React Native para se beneficiar dos novos recursos apresentados nesta postagem do blog. Para mais informações, consulte o [discurso principal da React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

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

## Atualizações para as APIs de Renderização do Cliente {/*updates-to-client-rendering-apis*/}

Quando você instala o React 18 pela primeira vez, verá um aviso no console:

<ConsoleBlock level="error">

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você altere para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 introduz uma nova API root que fornece melhor ergonomia para gerenciar roots. A nova API root também habilita o novo renderizador concorrente, que permite que você opte por recursos concorrentes.

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

Também removemos o callback de render, pois ele geralmente não tem o resultado esperado ao usar Suspense:

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

Não há uma substituição individual para a antiga API de callback de render — ela depende do seu caso de uso. Consulte a postagem do grupo de trabalho para [Substituir a renderização com createRoot](https://github.com/reactwg/react-18/discussions/5) para obter mais informações.

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

Para obter mais informações, consulte a [discussão do grupo de trabalho aqui](https://github.com/reactwg/react-18/discussions/5).

<Note>

**Se seu aplicativo não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** [O Modo Strict ficou mais rígido no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resilientes às novas verificações que ele adiciona no modo de desenvolvimento. Se a remoção do Strict Mode corrigir seu aplicativo, você pode removê-lo durante a atualização e, em seguida, adicioná-lo de volta (seja no topo ou para uma parte da árvore) depois de corrigir os problemas que ele está apontando.

</Note>

## Atualizações para as APIs de Renderização do Servidor {/*updates-to-server-rendering-apis*/}

Nesta versão, estamos renovando nossas APIs `react-dom/server` para oferecer suporte total ao Suspense no servidor e SSR de Streaming. Como parte dessas alterações, estamos descontinuando a antiga API de streaming do Node, que não oferece suporte ao streaming incremental do Suspense no servidor.

O uso dessa API agora avisará:

* `renderToNodeStream`: **Descontinuado ⛔️️**

Em vez disso, para streaming em ambientes Node, use:
* `renderToPipeableStream`: **Novo ✨**

Também estamos introduzindo uma nova API para oferecer suporte a SSR de streaming com Suspense para ambientes de tempo de execução de ponta modernos, como Deno e Cloudflare workers:
* `renderToReadableStream`: **Novo ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limitado** ⚠️
* `renderToStaticMarkup`: **Limitado** ⚠️

Finalmente, esta API continuará funcionando para renderizar e-mails:
* `renderToStaticNodeStream`

Para obter mais informações sobre as alterações nas APIs de renderização do servidor, consulte a postagem do grupo de trabalho sobre [Como atualizar para o React 18 no servidor](https://github.com/reactwg/react-18/discussions/22), uma [análise aprofundada da nova Arquitetura Suspense SSR](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) na React Conf 2021.

## Atualizações para definições de TypeScript {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e capturam problemas que costumavam ser ignorados pelo verificador de tipos. A mudança mais notável é que a prop `children` agora precisa ser listada explicitamente ao definir props, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Consulte a [solicitação de pull de tipagem do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para obter uma lista completa de alterações somente de tipo. Ele se conecta a correções de exemplo em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a transferir o código do seu aplicativo para as novas e mais seguras tipagens com mais rapidez.

Se você encontrar um erro nas tipagens, por favor, [crie um erro](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## Agrupamento Automático {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho prontas para uso, fazendo mais agrupamento por padrão. O agrupamento é quando o React agrupa várias atualizações de estado em uma única nova renderização para melhorar o desempenho. Antes do React 18, apenas agrupamos atualizações dentro de manipuladores de eventos do React. As atualizações dentro de promises, setTimeout, manipuladores de eventos nativos ou qualquer outro evento não foram agrupadas no React por padrão:

```js
// Antes do React 18 apenas os eventos do React foram agrupados

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

A partir do React 18 com `createRoot`, todas as atualizações serão agrupadas automaticamente, não importa de onde elas se originem. Isso significa que as atualizações dentro de timeouts, promises, manipuladores de eventos nativos ou qualquer outro evento serão agrupadas da mesma forma que as atualizações dentro dos eventos do React:

```js
// Depois do React 18, as atualizações dentro de timeouts, promises,
// manipuladores de eventos nativos ou qualquer outro evento são agrupadas.

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

Esta é uma alteração de última hora, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para desativar o agrupamento automático, você pode usar `flushSync`:

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

Para obter mais informações, consulte a [análise aprofundada do agrupamento automático](https://github.com/reactwg/react-18/discussions/21).

## Novas APIs para Bibliotecas {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os mantenedores da biblioteca para criar novas APIs necessárias para oferecer suporte à renderização concorrente para casos de uso específicos em áreas como estilos e lojas externas. Para dar suporte ao React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo Hook que permite que lojas externas ofereçam suporte a leituras concorrentes, forçando as atualizações da loja a serem síncronas. Esta nova API é recomendada para qualquer biblioteca que se integra a um estado externo ao React. Para obter mais informações, consulte a [post do useSyncExternalStore overview](https://github.com/reactwg/react-18/discussions/70) e [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo Hook que permite que as bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em renderizações. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado depois que o DOM for mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o layout. Para obter mais informações, consulte o [Guia de atualização da biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também introduz novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais na [publicação de lançamento](/blog/2022/03/29/react-v18).

## Atualizações para o Modo Strict {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permita ao React adicionar e remover seções da UI, preservando o estado. Por exemplo, quando um usuário sai de uma tela e retorna, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado do componente de antes.

Esse recurso proporcionará ao React melhor desempenho imediato, mas requer que os componentes sejam resilientes a efeitos que são montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos assumem que eles só são montados ou destruídos uma vez.

Para ajudar a expor esses problemas, o React 18 introduz uma nova verificação somente de desenvolvimento para o Strict Mode. Esta nova verificação irá automaticamente desmontar e remontar cada componente, sempre que um componente montar pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes dessa mudança, o React montaria o componente e criaria os efeitos:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
```

Com o Strict Mode no React 18, o React simulará a desmontagem e a remontagem do componente no modo de desenvolvimento:

```
* React monta o componente.
    * Efeitos de layout são criados.
    * Efeitos de efeito são criados.
* React simula a desmontagem do componente.
    * Efeitos de layout são destruídos.
    * Os efeitos são destruídos.
* React simula a montagem do componente com o estado anterior.
    * O código de configuração do efeito de layout é executado
    * O código de configuração do efeito é executado
```

Para obter mais informações, consulte as postagens do Grupo de Trabalho para [Adicionando estado reutilizável ao StrictMode](https://github.com/reactwg/react-18/discussions/19) e [Como oferecer suporte ao estado reutilizável em efeitos](https://github.com/reactwg/react-18/discussions/18).

## Configurando seu ambiente de teste {/*configuring-your-testing-environment*/}

Quando você atualiza seus testes pela primeira vez para usar `createRoot`, pode ver este aviso no console de teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para oferecer suporte a act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// No seu arquivo de configuração de teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo da flag é dizer ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir a flag como `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes de ponta a ponta que simulam um ambiente completo do navegador.

Eventualmente, esperamos que as bibliotecas de teste configurem isso automaticamente para você. Por exemplo, a [próxima versão do React Testing Library tem suporte integrado para o React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e as alterações relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Removendo o suporte para o Internet Explorer {/*dropping-support-for-internet-explorer*/}

Nesta versão, o React está removendo o suporte ao Internet Explorer, que [deixará de ter suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo esta mudança agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtarefas, que não podem ser adequadamente preenchidas no IE.

Se você precisar dar suporte ao Internet Explorer, recomendamos que você use o React 17.

## Descontinuações {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi descontinuado. Usá-lo avisará e executará seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi descontinuado.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi descontinuado.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi descontinuado.

## Outras alterações de última hora {/*other-breaking-changes*/}

* **Tempo de execução useEffect consistente**: React agora sempre libera funções de efeito de forma síncrona se a atualização for acionada durante um evento de entrada do usuário discreto, como um evento de clique ou keydown. Anteriormente, o comportamento nem sempre era previsível ou consistente.
* **Erros de hidratação mais rigorosos**: As incompatibilidades de hidratação devido à falta ou conteúdo de texto extra agora são tratadas como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente na tentativa de corresponder à marcação do servidor e reverterá para a renderização do cliente até o limite do `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evite possíveis buracos de privacidade e segurança que podem ser causados por incompatibilidades de hidratação.
* **Árvores Suspense são sempre consistentes:** Se um componente suspender antes de ser totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto ou disparará seus efeitos. Em vez disso, o React descartará completamente a nova árvore, aguardará a conclusão da operação assíncrona e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa de repetição simultaneamente e sem bloquear o navegador.
* **Efeitos de layout com Suspense**: Quando uma árvore suspender novamente e reverter para uma reserva, o React agora limpará os efeitos de layout e, em seguida, os recriará quando o conteúdo dentro do limite for exibido novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o layout quando usado com o Suspense.
* **Novos requisitos de ambiente JS**: React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você oferece suporte a navegadores e dispositivos mais antigos, como o Internet Explorer, que não fornecem recursos modernos do navegador de forma nativa ou têm implementações não compatíveis, considere incluir um preenchimento global em seu aplicativo empacotado.

## Outras mudanças notáveis {/*other-notable-changes*/}

### React {/*react*/}

* **Os componentes agora podem renderizar `undefined`:** React não emite mais avisos se você retornar `undefined` de um componente. Isso torna os valores de retorno de componentes permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos o uso de um linter para evitar erros como esquecer uma instrução `return` antes do JSX.
* **Em testes, os avisos `act` agora são opt-in:** Se você estiver executando testes de ponta a ponta, os avisos `act` são desnecessários. Introduzimos um mecanismo [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa ativá-los apenas para testes de unidade, onde eles são úteis e benéficos.
* **Sem aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chamava `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas o encontram principalmente em cenários em que a configuração do estado está bem e soluções alternativas tornam o código pior. [Removemos](https://github.com/facebook/react/pull/22114) este aviso.
* **Sem supressão de logs do console:** Quando você usa o Modo Strict, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os logs do console para uma das duas renderizações para tornar os logs mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo log serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
* **Uso de memória aprimorado:** O React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos grave.

### React DOM Server {/*react-dom-server*/}

* **`renderToString`:** Não dará mais erro ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo e, em seguida, tentará renderizar o mesmo conteúdo no cliente. Ainda é recomendado que você alterne para uma API de streaming como `renderToPipeableStream` ou `renderToReadableStream`.
* **`renderToStaticMarkup`:** Não dará mais erro ao suspender no servidor. Em vez disso, ele emitirá o HTML de fallback para o limite `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode consultar o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).