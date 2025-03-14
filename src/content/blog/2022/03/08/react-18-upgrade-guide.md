---
title: "Como Atualizar para o React 18"
author: Rick Hanlon
date: 2022/03/08
description: Como compartilhamos no post de lançamento, o React 18 apresenta recursos com tecnologia de nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, vamos orientá-lo pelas etapas de atualização para o React 18.
---

8 de março de 2022 por [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Como compartilhamos no [post de lançamento](/blog/2022/03/29/react-v18), o React 18 introduz recursos com tecnologia de nosso novo renderizador concorrente, com uma estratégia de adoção gradual para aplicativos existentes. Neste post, vamos orientá-lo pelas etapas de atualização para o React 18.

Por favor, [reporte quaisquer erros](https://github.com/facebook/react/issues/new/choose) que encontrar ao atualizar para o React 18.

</Intro>

<Note>

Para usuários do React Native, o React 18 será lançado em uma versão futura do React Native. Isso ocorre porque o React 18 se baseia na Nova Arquitetura do React Native para se beneficiar das novas funcionalidades apresentadas nesta publicação do blog. Para obter mais informações, consulte o [discurso principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

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

ReactDOM.render não é mais suportado no React 18. Use createRoot em vez disso. Até que você mude para a nova API, seu aplicativo se comportará como se estivesse executando o React 17. Saiba mais: https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

O React 18 introduz uma nova API de raiz que oferece melhor ergonomia para gerenciar raízes. A nova API de raiz também habilita o novo *renderer* concorrente, que permite que você opte por recursos concorrentes.

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

Também removemos o *callback* de render, pois ele geralmente não tem o resultado esperado ao usar Suspense:

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

Não há substituição direta para a antiga API de *callback* de render — isso depende do seu caso de uso. Consulte a publicação do grupo de trabalho para [Replacing render with createRoot](https://github.com/reactwg/react-18/discussions/5) para obter mais informações.

</Note>

Finalmente, se seu aplicativo usar renderização do lado do servidor com hidratação, atualize `hydrate` para `hydrateRoot`:

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

**Se seu aplicativo não funcionar após a atualização, verifique se ele está encapsulado em `<StrictMode>`.** [O Modo Strict ficou mais rigoroso no React 18](#updates-to-strict-mode), e nem todos os seus componentes podem ser resistentes às novas verificações que ele adiciona no modo de desenvolvimento. Se remover o Modo Strict corrigir seu aplicativo, você pode removê-lo durante a atualização e, em seguida, adicioná-lo novamente (no topo ou em parte da árvore) depois de corrigir os problemas que ele está apontando.

</Note>

## Atualizações para as APIs de Renderização do Servidor {/*updates-to-server-rendering-apis*/}

Nesta versão, estamos reformulando nossas APIs `react-dom/server` para suportar totalmente o Suspense no servidor e o *Streaming SSR*. Como parte dessas alterações, estamos descontinuando a antiga API de *streaming* do Node, que não suporta o *streaming* incremental do Suspense no servidor.

Usar esta API agora avisará:

* `renderToNodeStream`: **Deprecated ⛔️️**

Em vez disso, para *streaming* em ambientes Node, use:
* `renderToPipeableStream`: **New ✨**

Também estamos introduzindo uma nova API para suportar o *streaming SSR* com Suspense para ambientes de tempo de execução modernos de borda, como Deno e *workers* do Cloudflare:
* `renderToReadableStream`: **New ✨**

As seguintes APIs continuarão funcionando, mas com suporte limitado para Suspense:
* `renderToString`: **Limited** ⚠️
* `renderToStaticMarkup`: **Limited** ⚠️

Finalmente, esta API continuará funcionando para renderização de e-mails:
* `renderToStaticNodeStream`

Para obter mais informações sobre as alterações nas APIs de renderização do servidor, consulte a publicação do grupo de trabalho sobre [Upgrading to React 18 on the server](https://github.com/reactwg/react-18/discussions/22), uma [análise aprofundada da nova Arquitetura de SSR do Suspense](https://github.com/reactwg/react-18/discussions/37) e a palestra de [Shaundai Person](https://twitter.com/shaundai) sobre [Streaming Server Rendering with Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc) no React Conf 2021.

## Atualizações para definições do TypeScript {/*updates-to-typescript-definitions*/}

Se seu projeto usa TypeScript, você precisará atualizar suas dependências `@types/react` e `@types/react-dom` para as versões mais recentes. Os novos tipos são mais seguros e detectam problemas que costumavam ser ignorados pelo verificador de tipos. A alteração mais notável é que a *prop* `children` agora precisa ser listada explicitamente ao definir *props*, por exemplo:

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

Consulte a [solicitação de *pull* de *typings* do React 18](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) para obter uma lista completa de alterações somente de tipo. Ele vincula exemplos de correções em tipos de biblioteca para que você possa ver como ajustar seu código. Você pode usar o [script de migração automatizado](https://github.com/eps1lon/types-react-codemod) para ajudar a portar o código do seu aplicativo para os *typings* novos e mais seguros mais rapidamente.

Se você encontrar um erro nos *typings*, por favor, [crie um *issue*](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package) no repositório DefinitelyTyped.

## *Batching* Automático {/*automatic-batching*/}

O React 18 adiciona melhorias de desempenho *out-of-the-box* fazendo mais *batching* por padrão. *Batching* é quando o React agrupa várias atualizações de estado em uma única rerenderização para melhor desempenho. Antes do React 18, nós apenas fazíamos *batching* de atualizações dentro de *event handlers* do React. Atualizações dentro de *promises*, *setTimeout*, *event handlers* nativos ou qualquer outro evento não eram *batched* no React por padrão:

```js
// Antes do React 18 apenas os eventos do React eram batched

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React irá rerenderizar apenas uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React irá renderizar duas vezes, uma vez para cada atualização de estado (sem batching)
}, 1000);
```

A partir do React 18 com `createRoot`, todas as atualizações serão automaticamente *batched*, não importa de onde elas se originem. Isso significa que as atualizações dentro de *timeouts*, *promises*, *event handlers* nativos ou qualquer outro evento farão o *batching* da mesma forma que as atualizações dentro dos eventos do React:

```js
// Depois do React 18 as atualizações dentro de timeouts, promises,
// event handlers nativos ou qualquer outro evento são batched.

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React irá rerenderizar apenas uma vez no final (isso é batching!)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React irá rerenderizar apenas uma vez no final (isso é batching!)
}, 1000);
```

Essa é uma mudança *breaking*, mas esperamos que isso resulte em menos trabalho de renderização e, portanto, melhor desempenho em seus aplicativos. Para desativar o *batching* automático, você pode usar `flushSync`:

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React atualizou o DOM por agora
  flushSync(() => {
    setFlag(f => !f);
  });
  // React atualizou o DOM por agora
}
```

Para obter mais informações, consulte a [análise aprofundada do *batching* automático](https://github.com/reactwg/react-18/discussions/21).

## Novas APIs para Bibliotecas {/*new-apis-for-libraries*/}

No React 18 Working Group, trabalhamos com os responsáveis pela manutenção da biblioteca para criar novas APIs necessárias para suportar a renderização concorrente para casos de uso específicos para seu caso de uso em áreas como estilos e *stores* externos. Para suportar o React 18, algumas bibliotecas podem precisar mudar para uma das seguintes APIs:

* `useSyncExternalStore` é um novo *Hook* que permite que *stores* externos suportem leituras concorrentes, forçando as atualizações no *store* a serem síncronas. Essa nova API é recomendada para qualquer biblioteca que se integra ao estado externo ao React. Para obter mais informações, consulte a [publicação de visão geral do useSyncExternalStore](https://github.com/reactwg/react-18/discussions/70) e os [detalhes da API useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86).
* `useInsertionEffect` é um novo *Hook* que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este *Hook* será executado após o DOM ser mutado, mas antes que os efeitos de *layout* leiam o novo *layout*. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o *layout*. Para obter mais informações, consulte o [Guia de Atualização da Biblioteca para `<style>`](https://github.com/reactwg/react-18/discussions/110).

O React 18 também apresenta novas APIs para renderização concorrente, como `startTransition`, `useDeferredValue` e `useId`, sobre as quais compartilhamos mais no [post de lançamento](/blog/2022/03/29/react-v18).

## Atualizações ao Modo Strict {/*updates-to-strict-mode*/}

No futuro, gostaríamos de adicionar um recurso que permite ao React adicionar e remover seções da UI, preservando o estado. Por exemplo, quando um usuário muda de aba em uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado do componente de antes.

Esse recurso dará ao React melhor desempenho *out-of-the-box*, mas exige que os componentes sejam resistentes a efeitos sendo montados e destruídos várias vezes. A maioria dos efeitos funcionará sem quaisquer alterações, mas alguns efeitos assumem que eles só são montados ou destruídos uma vez.

Para ajudar a mostrar esses problemas, o React 18 apresenta uma nova verificação apenas para desenvolvimento no Strict Mode. Essa nova verificação desmontará e remontará automaticamente cada componente sempre que um componente for montado pela primeira vez, restaurando o estado anterior no segundo *mount*.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:

```
* O React monta o componente.
    * Os efeitos de *layout* são criados.
    * Os efeito de *effect* são criados.
```

Com o Modo Strict no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* O React monta o componente.
    * Os efeitos de *layout* são criados.
    * Os efeitos de *effect* são criados.
* O React simula a desmontagem do componente.
    * Os efeitos de *layout* são destruídos.
    * Os efeitos são destruídos.
* O React simula a montagem do componente com o estado anterior.
    * O código de configuração do efeito de layout é executado
    * O código de configuração do efeito é executado
```

Para obter mais informações, consulte as postagens do Grupo de Trabalho para [Adding Reusable State to StrictMode](https://github.com/reactwg/react-18/discussions/19) e [How to support Reusable State in Effects](https://github.com/reactwg/react-18/discussions/18).

## Configurando seu Ambiente de Teste {/*configuring-your-testing-environment*/}

Quando você atualiza seus testes pela primeira vez para usar `createRoot`, pode ver este aviso no console de teste:

<ConsoleBlock level="error">

O ambiente de teste atual não está configurado para dar suporte a act(...)

</ConsoleBlock>

Para corrigir isso, defina `globalThis.IS_REACT_ACT_ENVIRONMENT` como `true` antes de executar seu teste:

```js
// No seu arquivo de configuração de teste
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

O objetivo da *flag* é informar ao React que ele está sendo executado em um ambiente semelhante a um teste de unidade. O React registrará avisos úteis se você esquecer de encapsular uma atualização com `act`.

Você também pode definir a *flag* como `false` para dizer ao React que `act` não é necessário. Isso pode ser útil para testes de ponta a ponta que simulam um ambiente de navegador completo.

Eventualmente, esperamos que as bibliotecas de teste configurem isso automaticamente para você. Por exemplo, a [próxima versão da React Testing Library tem suporte integrado para React 18](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936) sem nenhuma configuração adicional.

[Mais informações sobre a API de teste `act` e alterações relacionadas](https://github.com/reactwg/react-18/discussions/102) estão disponíveis no grupo de trabalho.

## Removendo o Suporte para o Internet Explorer {/*dropping-support-for-internet-explorer*/}

Nesta versão, o React está removendo o suporte para o Internet Explorer, que [vai sair de suporte em 15 de junho de 2022](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge). Estamos fazendo essa alteração agora porque os novos recursos introduzidos no React 18 são construídos usando recursos modernos do navegador, como microtarefas, que não podem ser adequadamente *polyfilled* no IE.

Se você precisar suportar o Internet Explorer, recomendamos que você fique com o React 17.

## Depreciações {/*deprecations*/}

* `react-dom`: `ReactDOM.render` foi *deprecated*. Usá-lo irá avisar e executar seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.hydrate` foi *deprecated*. Usá-lo irá avisar e executar seu aplicativo no modo React 17.
* `react-dom`: `ReactDOM.unmountComponentAtNode` foi *deprecated*.
* `react-dom`: `ReactDOM.renderSubtreeIntoContainer` foi *deprecated*.
* `react-dom/server`: `ReactDOMServer.renderToNodeStream` foi *deprecated*.

## Outras Alterações *Breaking* {/*other-breaking-changes*/}

*   **Tempo consistente de `useEffect`**: O React agora sempre *flushes* as funções de efeito de forma síncrona se a atualização foi acionada durante um evento de entrada do usuário discreto, como um evento de clique ou *keydown*. Anteriormente, o comportamento nem sempre era previsível ou consistente.
*   **Erros de hidratação mais rigorosos**: Desajustes de hidratação devido à falta ou texto extra agora são tratados como erros em vez de avisos. O React não tentará mais "corrigir" nós individuais inserindo ou excluindo um nó no cliente em uma tentativa de corresponder a marcação do servidor e reverterá para a renderização do cliente até o limite de `<Suspense>` mais próximo na árvore. Isso garante que a árvore hidratada seja consistente e evita possíveis buracos de privacidade e segurança que podem ser causados por desajustes de hidratação.
*   **As árvores do Suspense são sempre consistentes:** Se um componente suspender antes que seja totalmente adicionado à árvore, o React não o adicionará à árvore em um estado incompleto ou acionará seus efeitos. Em vez disso, o React descartará a nova árvore completamente, aguardará a conclusão da operação assíncrona e, em seguida, tentará renderizar novamente do zero. O React renderizará a tentativa novamente simultaneamente e sem bloquear o navegador.
*   **Efeitos de *Layout* com Suspense**: Quando uma árvore é suspensa novamente e reverte para um *fallback*, o React agora limpará os efeitos de *layout* e, em seguida, os recriará quando o conteúdo dentro do limite for mostrado novamente. Isso corrige um problema que impedia as bibliotecas de componentes de medir corretamente o *layout* quando usado com Suspense.
*   **Novos Requisitos de Ambiente JS**: React agora depende de recursos modernos do navegador, incluindo `Promise`, `Symbol` e `Object.assign`. Se você suporta navegadores e dispositivos mais antigos como o Internet Explorer que não fornecem recursos modernos do navegador nativamente ou têm implementações não compatíveis, considere incluir um *polyfill* global em seu aplicativo em pacote.

## Outras Alterações Notáveis {/*other-notable-changes*/}

### React {/*react*/}

*   **Componentes podem agora renderizar `undefined`:** O React não avisa mais se você retornar `undefined` de um componente. Isso torna os valores de retorno de componentes permitidos consistentes com os valores que são permitidos no meio de uma árvore de componentes. Sugerimos o uso de um *linter* para evitar erros como esquecer uma instrução `return` antes do JSX.
*   **Nos testes, os avisos de `act` são agora de *opt-in*:** Se você estiver executando testes de ponta a ponta, os avisos de `act` são desnecessários. Introduzimos um mecanismo de [opt-in](https://github.com/reactwg/react-18/discussions/102) para que você possa habilitá-los apenas para testes de unidade onde eles são úteis e benéficos.
*   **Sem aviso sobre `setState` em componentes desmontados:** Anteriormente, o React avisava sobre vazamentos de memória quando você chama `setState` em um componente desmontado. Este aviso foi adicionado para assinaturas, mas as pessoas o encontram principalmente em cenários em que definir o estado está bom e as soluções alternativas tornam o código pior. Nós [removemos](https://github.com/facebook/react/pull/22114) este aviso.
*   **Sem supressão de *logs* do console:** Ao usar o Modo Strict, o React renderiza cada componente duas vezes para ajudá-lo a encontrar efeitos colaterais inesperados. No React 17, suprimimos os *logs* do console para uma das duas renderizações para tornar os *logs* mais fáceis de ler. Em resposta ao [feedback da comunidade](https://github.com/facebook/react/issues/21783) sobre isso ser confuso, removemos a supressão. Em vez disso, se você tiver o React DevTools instalado, as renderizações do segundo *log* serão exibidas em cinza e haverá uma opção (desativada por padrão) para suprimi-las completamente.
*   **Uso de memória melhorado:** O React agora limpa mais campos internos na desmontagem, tornando o impacto de vazamentos de memória não corrigidos que podem existir no código do seu aplicativo menos severo.

### React DOM Server {/*react-dom-server*/}

*   **`renderToString`:** Não irá mais gerar erros ao suspender no servidor. Em vez disso, ele emitirá o *fallback HTML* para o limite de `<Suspense>` mais próximo e, em seguida, tentará renderizar o mesmo conteúdo no cliente. Ainda é recomendado que você mude para uma API de *streaming* como `renderToPipeableStream` ou `renderToReadableStream` em vez disso.
*   **`renderToStaticMarkup`:** Não irá mais gerar erros ao suspender no servidor. Em vez disso, ele emitirá o *fallback HTML* para o limite de `<Suspense>` mais próximo.

## Changelog {/*changelog*/}

Você pode visualizar o [changelog completo aqui](https://github.com/facebook/react/blob/main/CHANGELOG.md).