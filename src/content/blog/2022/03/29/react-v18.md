---
title: "React v18.0"
author: The React Team
date: 2022/03/08
description: React 18 está disponível no npm! Em nossa última publicação, compartilhamos as instruções passo a passo para atualizar seu aplicativo para React 18. Nesta publicação, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.
---

29 de março de 2022 por [The React Team](/community/team)

---

<Intro>

React 18 já está disponível no npm! Em nossa última publicação, compartilhamos as instruções passo a passo para [atualizar seu aplicativo para React 18](/blog/2022/03/08/react-18-upgrade-guide). Nesta publicação, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.

</Intro>

---

Nossa versão principal mais recente inclui melhorias prontas para uso, como `automatic batching`, novas APIs como `startTransition` e `streaming server-side rendering` com suporte para `Suspense`.

Muitos dos recursos do React 18 são construídos sobre nosso novo renderizador concorrente, uma alteração nos bastidores que desbloqueia novos recursos poderosos. O React concorrente é opcional — ele só é habilitado quando você usa um recurso concorrente — mas achamos que ele terá um grande impacto na forma como as pessoas constroem aplicativos.

Passamos anos pesquisando e desenvolvendo suporte para concorrência no React e tomamos cuidado extra para fornecer um caminho de adoção gradual para usuários existentes. No verão passado, [formamos o Grupo de Trabalho do React 18](/blog/2021/06/08/the-plan-for-react-18) para coletar feedback de especialistas da comunidade e garantir uma experiência de atualização tranquila para todo o ecossistema React.

Caso você tenha perdido, compartilhamos muito dessa visão no React Conf 2021:

*  No [discurso principal](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa), explicamos como o React 18 se encaixa em nossa missão de facilitar para os desenvolvedores a construção de ótimas experiências do usuário
*  [Shruti Kapoor](https://twitter.com/shrutikapoor08) [demonstrou como usar os novos recursos no React 18](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
*  [Shaundai Person](https://twitter.com/shaundai) nos deu uma visão geral do [streaming server rendering com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

Abaixo está uma visão geral completa do que esperar nesta versão, começando com a Renderização Concorrente.

<Note>

Para usuários do React Native, o React 18 será lançado no React Native com a Nova Arquitetura do React Native. Para mais informações, consulte o [discurso principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

## O que é React Concorrente? {/*what-is-concurrent-react*/}

A adição mais importante no React 18 é algo que esperamos que você nunca precise pensar: a concorrência. Achamos que isso é amplamente verdadeiro para desenvolvedores de aplicativos, embora a história possa ser um pouco mais complicada para os mantenedores de bibliotecas.

A concorrência não é um recurso, por si só. É um novo mecanismo nos bastidores que permite que o React prepare várias versões da sua UI ao mesmo tempo. Você pode pensar na concorrência como um detalhe de implementação — é valioso por causa dos recursos que desbloqueia. O React usa técnicas sofisticadas em sua implementação interna, como filas de prioridade e buffer múltiplo. Mas você não verá esses conceitos em nenhum lugar em nossas APIs públicas.

Quando projetamos APIs, tentamos esconder os detalhes da implementação dos desenvolvedores. Como um desenvolvedor React, você se concentra no *que* deseja que a experiência do usuário pareça e o React lida com *como* fornecer essa experiência. Portanto, não esperamos que os desenvolvedores React saibam como a concorrência funciona sob o capô.

No entanto, o React Concorrente é mais importante do que um detalhe de implementação típico — é uma atualização fundamental para o modelo de renderização principal do React. Portanto, embora não seja super importante saber como a concorrência funciona, pode valer a pena saber o que ela é em um nível superior.

Uma propriedade chave do React Concorrente é que a renderização é interrompível. Quando você atualiza pela primeira vez para o React 18, antes de adicionar quaisquer recursos concorrentes, as atualizações são renderizadas da mesma forma que nas versões anteriores do React — em uma única transação síncrona e ininterrupta. Com a renderização síncrona, uma vez que uma atualização começa a renderizar, nada pode interrompê-la até que o usuário possa ver o resultado na tela.

Em uma renderização concorrente, este nem sempre é o caso. O React pode começar a renderizar uma atualização, pausar no meio e, em seguida, continuar mais tarde. Pode até abandonar uma renderização em andamento completamente. O React garante que a UI aparecerá consistente mesmo que uma renderização seja interrompida. Para fazer isso, ele espera para realizar mutações do DOM até o final, depois que toda a árvore foi avaliada. Com essa capacidade, o React pode preparar novas telas em segundo plano sem bloquear a `main thread`. Isso significa que a UI pode responder imediatamente à entrada do usuário, mesmo que esteja no meio de uma grande tarefa de renderização, criando uma experiência do usuário fluida.

Outro exemplo é o `state` reutilizável. O React Concorrente pode remover seções da UI da tela e, em seguida, adicioná-las de volta mais tarde, reutilizando o `state` anterior. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de restaurar a tela anterior no mesmo `state` em que estava antes. Em uma próxima versão secundária, estamos planejando adicionar um novo componente chamado `<Offscreen>` que implementa esse padrão. Da mesma forma, você poderá usar Offscreen para preparar uma nova UI em segundo plano para que ela esteja pronta antes que o usuário a revele.

A renderização concorrente é uma nova ferramenta poderosa no React e a maioria de nossos novos recursos são construídos para tirar proveito dela, incluindo `Suspense`, `transitions` e `streaming server rendering`. Mas o React 18 é apenas o começo do que pretendemos construir com essa nova base.

## Adotando Gradualmente Recursos Concorrentes {/*gradually-adopting-concurrent-features*/}

Tecnicamente, a renderização concorrente é uma **mudança radical**. Como a renderização concorrente é interrompível, os componentes se comportam de maneira um pouco diferente quando ela é habilitada.

Em nossos testes, atualizamos milhares de componentes para o React 18. O que descobrimos é que quase todos os componentes existentes "simplesmente funcionam" com a renderização concorrente, sem nenhuma alteração. No entanto, alguns deles podem exigir algum esforço de migração adicional. Embora as alterações geralmente sejam pequenas, você ainda terá a capacidade de fazê-las no seu próprio ritmo. O novo comportamento de renderização no React 18 é **habilitado apenas nas partes do seu aplicativo que usam novos recursos.**

A estratégia geral de atualização é fazer com que seu aplicativo seja executado no React 18 sem interromper o código existente. Então você pode gradualmente começar a adicionar recursos concorrentes no seu próprio ritmo. Você pode usar [`<StrictMode>`](/reference/react/StrictMode) para ajudar a detectar erros relacionados à concorrência durante o desenvolvimento. O Strict Mode não afeta o comportamento de produção, mas durante o desenvolvimento ele registrará avisos extras e invocará funções duplas que devem ser idempotentes. Ele não pegará tudo, mas é eficaz para prevenir os tipos mais comuns de erros.

Depois de atualizar para o React 18, você poderá começar a usar os recursos concorrentes imediatamente. Por exemplo, você pode usar `startTransition` para navegar entre as telas sem bloquear a entrada do usuário. Ou `useDeferredValue` para limitar as re-renderizações caras.

No entanto, a longo prazo, esperamos que a principal forma pela qual você adicionará concorrência ao seu aplicativo seja usando uma biblioteca ou `framework` habilitado para concorrência. Na maioria dos casos, você não interagirá com as APIs concorrentes diretamente. Por exemplo, em vez de os desenvolvedores chamarem `startTransition` sempre que navegarem para uma nova tela, as bibliotecas de roteador automaticamente encapsularão as navegações no `startTransition`.

Pode levar algum tempo para que as bibliotecas sejam atualizadas para serem compatíveis com a concorrência. Fornecemos novas APIs para facilitar que as bibliotecas aproveitem os recursos concorrentes. Enquanto isso, seja paciente com os mantenedores enquanto trabalhamos para migrar gradualmente o ecossistema React.

Para mais informações, consulte nossa publicação anterior: [Como atualizar para React 18](/blog/2022/03/08/react-18-upgrade-guide).

## Suspense em Frameworks de Dados {/*suspense-in-data-frameworks*/}

No React 18, você pode começar a usar [Suspense](/reference/react/Suspense) para busca de dados em `frameworks` opinativos como Relay, Next.js, Hydrogen ou Remix. A busca de dados ad hoc com Suspense é tecnicamente possível, mas ainda não é recomendada como uma estratégia geral.

No futuro, podemos expor primitivos adicionais que poderiam facilitar o acesso aos seus dados com Suspense, talvez sem o uso de um `framework` de opinião. No entanto, o Suspense funciona melhor quando está profundamente integrado à arquitetura de seu aplicativo: seu roteador, sua camada de dados e seu ambiente de renderização do servidor. Portanto, mesmo a longo prazo, esperamos que as bibliotecas e os `frameworks` desempenhem um papel crucial no ecossistema React.

Como nas versões anteriores do React, você também pode usar o Suspense para divisão de código no cliente com `React.lazy`. Mas nossa visão para o Suspense sempre foi muito mais do que carregar código — o objetivo é estender o suporte para o Suspense para que, eventualmente, o mesmo `fallback` declarativo do Suspense possa lidar com qualquer operação assíncrona (carregar código, dados, imagens, etc).

## Server Components ainda está em desenvolvimento {/*server-components-is-still-in-development*/}

[**Server Components**](/blog/2020/12/21/data-fetching-with-react-server-components) é um recurso futuro que permite aos desenvolvedores construir aplicativos que abrangem o servidor e o cliente, combinando a rica interatividade de aplicativos do lado do cliente com o desempenho aprimorado da renderização tradicional do servidor. O Server Components não é inerentemente acoplado ao React Concorrente, mas foi projetado para funcionar melhor com recursos concorrentes como Suspense e `streaming server rendering`.

O Server Components ainda é experimental, mas esperamos lançar uma versão inicial em uma versão secundária 18.x. Enquanto isso, estamos trabalhando com `frameworks` como Next.js, Hydrogen e Remix para avançar na proposta e prepará-la para ampla adoção.

## O que há de novo no React 18 {/*whats-new-in-react-18*/}

### Novo recurso: `Automatic Batching` {/*new-feature-automatic-batching*/}

`Batching` é quando o React agrupa várias atualizações de `state` em uma única re-renderização para melhor desempenho. Sem o `automatic batching`, só fizemos a `batching` de atualizações dentro dos `event handlers` do React. As atualizações dentro de `promises`, `setTimeout`, `native event handlers` ou qualquer outro evento não foram `batched` no React por padrão. Com o `automatic batching`, essas atualizações serão `batched` automaticamente:

```js
// Anterior: apenas os eventos React foram batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React fará render duas vezes, uma para cada atualização de estado (sem batching)
}, 1000);

// Depois: as atualizações dentro de timeouts, promises,
// native event handlers ou qualquer outro evento são batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React só fará re-render uma vez no final (isso é batching!)
}, 1000);
```

Para mais informações, veja esta publicação para [Automatic batching para menos renders no React 18](https://github.com/reactwg/react-18/discussions/21).

### Novo recurso: `Transitions` {/*new-feature-transitions*/}

Uma `transition` é um novo conceito no React para distinguir entre atualizações urgentes e não urgentes.

*  **Atualizações urgentes** refletem a interação direta, como digitar, clicar, pressionar e assim por diante.
*  **Atualizações de transição** fazem a transição da UI de uma visualização para outra.

Atualizações urgentes como digitação, cliques ou pressionamentos precisam de resposta imediata para combinar com nossas intuições sobre como os objetos físicos se comportam. Caso contrário, eles parecem "errados". No entanto, as `transitions` são diferentes porque o usuário não espera ver cada valor intermediário na tela.

Por exemplo, ao selecionar um filtro em um menu suspenso, você espera que o próprio botão de filtro responda imediatamente quando você clica. No entanto, os resultados reais podem realizar a transição separadamente. Um pequeno atraso seria imperceptível e muitas vezes esperado. E se você alterar o filtro novamente antes que os resultados terminem de renderizar, você só se importa em ver os resultados mais recentes.

Normalmente, para a melhor experiência do usuário, uma única entrada do usuário deve resultar em uma atualização urgente e não urgente. Você pode usar a API `startTransition` dentro de um evento de entrada para informar ao React quais atualizações são urgentes e quais são "`transitions`":

```js
import { startTransition } from 'react';

// Urgente: Mostra o que foi digitado
setInputValue(input);

// Marcar todas as atualizações de estado internas como transitions
startTransition(() => {
  // Transition: Mostra os resultados
  setSearchQuery(input);
});
```

As atualizações encapsuladas em `startTransition` são tratadas como não urgentes e serão interrompidas se atualizações mais urgentes, como cliques ou pressionamentos de teclas, entrarem. Se uma `transition` for interrompida pelo usuário (por exemplo, digitando vários caracteres seguidos), o React descartará o trabalho de renderização obsoleto que não foi concluído e renderizará apenas a atualização mais recente.

*  `useTransition`: um Hook para iniciar `transitions`, incluindo um valor para rastrear o `state` pendente.
*  `startTransition`: um método para iniciar `transitions` quando o Hook não puder ser usado.

As `transitions` aceitarão a renderização concorrente, o que permite que a atualização seja interrompida. Se o conteúdo re-suspender, as `transitions` também informarão ao React para continuar mostrando o conteúdo atual enquanto renderiza o conteúdo da `transition` em segundo plano (consulte o [Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md) para mais informações).

[Veja a documentação para transitions aqui](/reference/react/useTransition).

### Novos recursos do Suspense {/*new-suspense-features*/}

O Suspense permite que você especifique declarativamente o `state` de carregamento para uma parte da árvore de componentes, se ainda não estiver pronto para ser exibido:

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

O Suspense torna o "UI loading state" um conceito declarativo de primeira classe no modelo de programação do React. Isso nos permite construir recursos de nível superior com base nele.

Introduzimos uma versão limitada do Suspense há vários anos. No entanto, o único caso de uso suportado foi a divisão de código com `React.lazy` e não foi suportado em absoluto ao renderizar no servidor.

No React 18, adicionamos suporte para Suspense no servidor e expandimos seus recursos usando recursos de renderização concorrente.

O Suspense no React 18 funciona melhor quando combinado com a API `transition`. Se você suspender durante uma `transition`, o React impedirá que o conteúdo já visível seja substituído por um `fallback`. Em vez disso, o React atrasará a renderização até que dados suficientes sejam carregados para evitar um mau `state` de carregamento.

Para mais informações, consulte o RFC para [Suspense em React 18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md).

### Novas APIs de renderização de cliente e servidor {/*new-client-and-server-rendering-apis*/}

Nesta versão, aproveitamos a oportunidade para redesenhar as APIs que expomos para renderização no cliente e no servidor. Essas alterações permitem que os usuários continuem usando as APIs antigas no modo React 17 enquanto atualizam para as novas APIs no React 18.

#### React DOM Client {/*react-dom-client*/}

Essas novas APIs agora são exportadas de `react-dom/client`:

*  `createRoot`: Novo método para criar uma raiz para `render` ou `unmount`. Use-o em vez de `ReactDOM.render`. Novos recursos no React 18 não funcionam sem ele.
*  `hydrateRoot`: Novo método para hidratar um aplicativo renderizado no servidor. Use-o em vez de `ReactDOM.hydrate` em conjunto com as novas APIs do React DOM Server. Novos recursos no React 18 não funcionam sem ele.

`createRoot` e `hydrateRoot` aceitam uma nova opção chamada `onRecoverableError` caso você queira ser notificado quando o React se recuperar de erros durante a renderização ou hidratação para registro. Por padrão, React usará [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError), ou `console.error` nos navegadores mais antigos.

[Veja a documentação para React DOM Client aqui](/reference/react-dom/client).

#### React DOM Server {/*react-dom-server*/}

Essas novas APIs agora são exportadas de `react-dom/server` e têm suporte total para `streaming Suspense` no servidor:

*  `renderToPipeableStream`: para `streaming` em ambientes Node.
*  `renderToReadableStream`: para ambientes de tempo de execução de ponta modernos, como Deno e Cloudflare workers.

O método `renderToString` existente continua funcionando, mas é desencorajado.

[Veja a documentação para React DOM Server aqui](/reference/react-dom/server).

### Novos comportamentos do Strict Mode {/*new-strict-mode-behaviors*/}

No futuro, gostaríamos de adicionar um recurso que permita que o React adicione e remova seções da UI, preservando o `state`. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria as árvores usando o mesmo `state` de componente de antes.

Este recurso dará aos aplicativos React um melhor desempenho pronto para uso, mas exige que os componentes sejam resistentes a `effects` sendo montados e destruídos várias vezes. A maioria dos `effects` funcionará sem nenhuma alteração, mas alguns `effects` presumem que sejam apenas montados ou destruídos uma vez.

Para ajudar a detectar esses problemas, o React 18 apresenta uma nova verificação somente para desenvolvimento no Strict Mode. Essa nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o `state` anterior na segunda montagem.

Antes desta alteração, o React montaria o componente e criaria os `effects`:
``````
* React renderiza o componente.
  * Efeitos de layout são criados.
  * Efeitos são criados.
```

Com o Strict Mode no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React renderiza o componente.
  * Efeitos de layout são criados.
  * Efeitos são criados.
* React simula a desmontagem do componente.
  * Efeitos de layout são destruídos.
  * Efeitos são destruídos.
* React simula a montagem do componente com o estado anterior.
  * Efeitos de layout são criados.
  * Efeitos são criados.
```

[Veja a documentação para garantir o estado reutilizável aqui](/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development).

### Novos Hooks {/*new-hooks*/}

#### useId {/*useid*/}

`useId` é um novo Hook para gerar IDs únicos no cliente e no servidor, evitando incompatibilidades de hidratação. Ele é usado principalmente para bibliotecas de componentes que se integram com APIs de acessibilidade que exigem IDs únicos. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 por causa de como o novo renderizador de servidor de streaming entrega HTML fora de ordem. [Veja a documentação aqui](/reference/react/useId).

> Nota
>
> `useId` **não** é para gerar [chaves em uma lista](/learn/rendering-lists#where-to-get-your-key). Chaves devem ser geradas a partir dos seus dados.

#### useTransition {/*usetransition*/}

`useTransition` e `startTransition` permitem que você marque algumas atualizações de estado como não urgentes. Outras atualizações de estado são consideradas urgentes por padrão. O React permitirá que atualizações de estado urgentes (por exemplo, atualizar uma entrada de texto) interrompam atualizações de estado não urgentes (por exemplo, renderizar uma lista de resultados de pesquisa). [Veja a documentação aqui](/reference/react/useTransition).

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` permite que você adie a renderização de uma parte não urgente da árvore. É semelhante a debouncing, mas tem algumas vantagens em comparação a ele. Não há um atraso de tempo fixo, então o React tentará a renderização adiada logo após a primeira renderização ser refletida na tela. A renderização adiada é interrompível e não bloqueia a entrada do usuário. [Veja a documentação aqui](/reference/react/useDeferredValue).

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` é um novo Hook que permite que lojas externas suportem leituras concorrentes, forçando as atualizações na loja a serem síncronas. Ele remove a necessidade de useEffect ao implementar assinaturas em fontes de dados externas e é recomendado para qualquer biblioteca que se integre ao estado externo ao React. [Veja a documentação aqui](/reference/react/useSyncExternalStore).

> Nota
>
> `useSyncExternalStore` deve ser usado por bibliotecas, não por código de aplicação.

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização simultânea, dando a ele uma chance de recalcular o layout. [Veja a documentação aqui](/reference/react/useInsertionEffect).

> Nota
>
> `useInsertionEffect` deve ser usado por bibliotecas, não por código de aplicação.

## Como Atualizar {/*how-to-upgrade*/}

Veja [Como Atualizar para o React 18](/blog/2022/03/08/react-18-upgrade-guide) para obter instruções passo a passo e uma lista completa de mudanças importantes e notáveis.

## Changelog {/*changelog*/}

### React {/*react*/}

* Adiciona `useTransition` e `useDeferredValue` para separar atualizações urgentes de transições. ([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `useId` para gerar IDs únicos. ([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `useSyncExternalStore` para ajudar as bibliotecas de loja externa a serem integradas com o React. ([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@drarmstr](https://github.com/drarmstr))
* Adiciona `startTransition` como uma versão de `useTransition` sem feedback pendente. ([#19696](https://github.com/facebook/react/pull/19696)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Adiciona `useInsertionEffect` para bibliotecas CSS-in-JS. ([#21913](https://github.com/facebook/react/pull/21913)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Faz com que Suspense remonte os efeitos de layout quando o conteúdo reaparece.  ([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Faz com que `<StrictMode>` re-execute os efeitos para verificar o estado restaurável. ([#19523](https://github.com/facebook/react/pull/19523) , [#21418](https://github.com/facebook/react/pull/21418)  por [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Assume que os Símbolos estão sempre disponíveis. ([#23348](https://github.com/facebook/react/pull/23348)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Remove o polyfill `object-assign`. ([#23351](https://github.com/facebook/react/pull/23351)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Remove a API `unstable_changedBits` sem suporte.  ([#20953](https://github.com/facebook/react/pull/20953)  por [@acdlite](https://github.com/acdlite))
* Permite que os componentes renderizem undefined. ([#21869](https://github.com/facebook/react/pull/21869)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Limpa `useEffect` resultante de eventos discretos como cliques de forma síncrona. ([#21150](https://github.com/facebook/react/pull/21150)  por [@acdlite](https://github.com/acdlite))
* `fallback={undefined}` do Suspense agora se comporta da mesma forma que `null` e não é ignorado. ([#21854](https://github.com/facebook/react/pull/21854)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Considera todos `lazy()` resolvendo para o mesmo componente equivalente. ([#20357](https://github.com/facebook/react/pull/20357)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Não corrige o console durante a primeira renderização. ([#22308](https://github.com/facebook/react/pull/22308)  por [@lunaruan](https://github.com/lunaruan))
* Melhora o uso da memória. ([#21039](https://github.com/facebook/react/pull/21039)  por [@bgirard](https://github.com/bgirard))
* Melhora as mensagens se a coerção de string lançar (Temporal.*, Symbol, etc.) ([#22064](https://github.com/facebook/react/pull/22064)  por [@justingrant](https://github.com/justingrant))
* Usa `setImmediate` quando disponível sobre `MessageChannel`. ([#20834](https://github.com/facebook/react/pull/20834)  por [@gaearon](https://github.com/gaearon))
* Corrige a falha do contexto na propagação dentro de árvores suspensas. ([#23095](https://github.com/facebook/react/pull/23095)  por [@gaearon](https://github.com/gaearon))
* Corrige `useReducer` observando props incorretas, removendo o mecanismo de resgate ansioso. ([#22445](https://github.com/facebook/react/pull/22445)  por [@josephsavona](https://github.com/josephsavona))
* Corrige `setState` sendo ignorado no Safari ao anexar iframes. ([#23111](https://github.com/facebook/react/pull/23111)  por [@gaearon](https://github.com/gaearon))
* Corrige uma falha ao renderizar `ZonedDateTime` na árvore. ([#20617](https://github.com/facebook/react/pull/20617)  por [@dimaqq](https://github.com/dimaqq))
* Corrige uma falha quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695)  por [@SimenB](https://github.com/SimenB))
* Corrige `onLoad` não sendo acionado quando os recursos simultâneos estão ativos. ([#23316](https://github.com/facebook/react/pull/23316)  por [@gnoff](https://github.com/gnoff))
* Corrige um aviso quando um seletor retorna `NaN`.  ([#23333](https://github.com/facebook/react/pull/23333)  por [@hachibeeDI](https://github.com/hachibeeDI))
* Corrige uma falha quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) por [@SimenB](https://github.com/SimenB))
* Corrige o cabeçalho de licença gerado. ([#23004](https://github.com/facebook/react/pull/23004)  por [@vitaliemiron](https://github.com/vitaliemiron))
* Adiciona `package.json` como um dos pontos de entrada. ([#22954](https://github.com/facebook/react/pull/22954)  por [@Jack](https://github.com/Jack-Works))
* Permite suspender fora de uma fronteira Suspense. ([#23267](https://github.com/facebook/react/pull/23267)  por [@acdlite](https://github.com/acdlite))
* Registra um erro recuperável sempre que a hidratação falha. ([#23319](https://github.com/facebook/react/pull/23319)  por [@acdlite](https://github.com/acdlite))

### React DOM {/*react-dom*/}

* Adiciona `createRoot` e `hydrateRoot`. ([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/pull/20888), [#21072](https://github.com/facebook/react/pull/21072), [#21417](https://github.com/facebook/react/pull/21417), [#21652](https://github.com/facebook/react/pull/21652), [#21687](https://github.com/facebook/react/pull/21687), [#23207](https://github.com/facebook/react/pull/23207), [#23385](https://github.com/facebook/react/pull/23385) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), [@gaearon](https://github.com/gaearon), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), [@trueadm](https://github.com/trueadm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona hidratação seletiva. ([#14717](https://github.com/facebook/react/pull/14717), [#14884](https://github.com/facebook/react/pull/14884), [#16725](https://github.com/facebook/react/pull/16725), [#16880](https://github.com/facebook/react/pull/16880), [#17004](https://github.com/facebook/react/pull/17004), [#22416](https://github.com/facebook/react/pull/22416), [#22629](https://github.com/facebook/react/pull/22629), [#22448](https://github.com/facebook/react/pull/22448), [#22856](https://github.com/facebook/react/pull/22856), [#23176](https://github.com/facebook/react/pull/23176) por [@acdlite](https://github.com/acdlite), [@gaearon](https://github.com/gaearon), [@salazarm](https://github.com/salazarm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `aria-description` à lista de atributos ARIA conhecidos. ([#22142](https://github.com/facebook/react/pull/22142)  por [@mahyareb](https://github.com/mahyareb))
* Adiciona evento `onResize` a elementos de vídeo. ([#21973](https://github.com/facebook/react/pull/21973)  por [@rileyjshaw](https://github.com/rileyjshaw))
* Adiciona `imageSizes` e `imageSrcSet` a props conhecidas. ([#22550](https://github.com/facebook/react/pull/22550)  por [@eps1lon](https://github.com/eps1lon))
* Permite filhos `<option>` que não sejam strings se `value` for fornecido.  ([#21431](https://github.com/facebook/react/pull/21431)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige o estilo `aspectRatio` não sendo aplicado. ([#21100](https://github.com/facebook/react/pull/21100)  por [@gaearon](https://github.com/gaearon))
* Avisa se `renderSubtreeIntoContainer` for chamado. ([#23355](https://github.com/facebook/react/pull/23355)  por [@acdlite](https://github.com/acdlite))

### React DOM Server {/*react-dom-server-1*/}

* Adiciona o novo renderizador de streaming. ([#14144](https://github.com/facebook/react/pull/14144), [#20970](https://github.com/facebook/react/pull/20970), [#21056](https://github.com/facebook/react/pull/21056), [#21255](https://github.com/facebook/react/pull/21255), [#21200](https://github.com/facebook/react/pull/21200), [#21257](https://github.com/facebook/react/pull/21257), [#21276](https://github.com/facebook/react/pull/21276), [#22443](https://github.com/facebook/react/pull/22443), [#22450](https://github.com/facebook/react/pull/22450), [#23247](https://github.com/facebook/react/pull/23247), [#24025](https://github.com/facebook/react/pull/24025), [#24030](https://github.com/facebook/react/pull/24030) por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige provedores de contexto no SSR ao lidar com várias solicitações. ([#23171](https://github.com/facebook/react/pull/23171)  por [@frandiox](https://github.com/frandiox))
* Retorna para a renderização do cliente em incompatibilidade de texto. ([#23354](https://github.com/facebook/react/pull/23354)  por [@acdlite](https://github.com/acdlite))
* Deprecia `renderToNodeStream`. ([#23359](https://github.com/facebook/react/pull/23359)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige um log de erro falso no novo renderizador de servidor. ([#24043](https://github.com/facebook/react/pull/24043)  por [@eps1lon](https://github.com/eps1lon))
* Corrige um erro no novo renderizador de servidor. ([#22617](https://github.com/facebook/react/pull/22617)  por [@shuding](https://github.com/shuding))
* Ignora valores de função e símbolo dentro de elementos personalizados no servidor. ([#21157](https://github.com/facebook/react/pull/21157)  por [@sebmarkbage](https://github.com/sebmarkbage))

### React DOM Test Utils {/*react-dom-test-utils*/}

* Lança exceção quando `act` é usado em produção. ([#21686](https://github.com/facebook/react/pull/21686)  por [@acdlite](https://github.com/acdlite))
* Suporta a desativação de avisos de ato falsos com `global.IS_REACT_ACT_ENVIRONMENT`. ([#22561](https://github.com/facebook/react/pull/22561)  por [@acdlite](https://github.com/acdlite))
* Expande o aviso de ato para cobrir todas as APIs que podem agendar o trabalho do React. ([#22607](https://github.com/facebook/react/pull/22607)  por [@acdlite](https://github.com/acdlite))
* Faz com que `act` atualize o lote. ([#21797](https://github.com/facebook/react/pull/21797)  por [@acdlite](https://github.com/acdlite))
* Remove o aviso para efeitos passivos pendentes. ([#22609](https://github.com/facebook/react/pull/22609)  por [@acdlite](https://github.com/acdlite))

### React Refresh {/*react-refresh*/}

* Rastreia raízes montadas tardiamente em Fast Refresh. ([#22740](https://github.com/facebook/react/pull/22740)  por [@anc95](https://github.com/anc95))
* Adiciona o campo `exports` a `package.json`. ([#23087](https://github.com/facebook/react/pull/23087)  por [@otakustay](https://github.com/otakustay))

### Server Components (Experimental) {/*server-components-experimental*/}

* Adiciona suporte ao Server Context. ([#23244](https://github.com/facebook/react/pull/23244)  por [@salazarm](https://github.com/salazarm))
* Adiciona suporte `lazy`. ([#24068](https://github.com/facebook/react/pull/24068)  por [@gnoff](https://github.com/gnoff))
* Atualiza o plugin webpack para webpack 5 ([#22739](https://github.com/facebook/react/pull/22739)  por [@michenly](https://github.com/michenly))
* Corrige um engano no carregador do Node. ([#22537](https://github.com/facebook/react/pull/22537)  por [@btea](https://github.com/btea))
* Usa `globalThis` em vez de `window` para ambientes de borda. ([#22777](https://github.com/facebook/react/pull/22777)  por [@huozhi](https://github.com/huozhi))
```