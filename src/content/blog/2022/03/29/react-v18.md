---
title: "React v18.0"
author: The React Team
date: 2022/03/08
description: React 18 está disponível no npm! Em nosso último post, compartilhamos instruções passo a passo para atualizar seu aplicativo para React 18. Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.
---

29 de março de 2022, por [The React Team](/community/team)

---

<Intro>

O React 18 está disponível no npm! Em nosso último post, compartilhamos as instruções passo a passo para [atualizar seu aplicativo para React 18](/blog/2022/03/08/react-18-upgrade-guide). Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.

</Intro>

---

Nossa versão principal mais recente inclui melhorias prontas para uso, como *automatic batching*, novas APIs como startTransition e *streaming server-side rendering* com suporte para Suspense.

Muitos dos recursos do React 18 são construídos sobre nosso novo *concurrent renderer*, uma alteração nos bastidores que desbloqueia novos recursos poderosos. O React Concorrente (Concurrent React) é *opt-in* — ele só é habilitado quando você usa um recurso concorrente — mas achamos que isso terá um grande impacto na forma como as pessoas constroem aplicativos.

Passamos anos pesquisando e desenvolvendo suporte para concorrência no React e tomamos cuidado extra para fornecer um caminho de adoção gradual para usuários existentes. No verão passado, [formamos o Grupo de Trabalho do React 18](/blog/2021/06/08/the-plan-for-react-18) para coletar feedback de especialistas da comunidade e garantir uma experiência de atualização tranquila para todo o ecossistema React.

Caso você tenha perdido, compartilhamos grande parte dessa visão na React Conf 2021:

*   Na [palestra principal](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa), explicamos como o React 18 se encaixa em nossa missão de facilitar para os desenvolvedores a criação de ótimas experiências do usuário.
*   [Shruti Kapoor](https://twitter.com/shrutikapoor08) [demonstrou como usar os novos recursos no React 18](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
*   [Shaundai Person](https://twitter.com/shaundai) nos deu uma visão geral da [renderização do servidor de streaming com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

Abaixo está uma visão geral completa do que esperar nesta versão, começando com a Renderização Concorrente.

<Note>

Para usuários do React Native, o React 18 será enviado no React Native com a Nova Arquitetura do React Native. Para mais informações, consulte a [palestra principal da React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

## O que é React Concorrente? {/*what-is-concurrent-react*/}

A adição mais importante no React 18 é algo que esperamos que você nunca precise pensar: concorrência. Achamos que isso é amplamente verdadeiro para desenvolvedores de aplicativos, embora a história possa ser um pouco mais complicada para os mantenedores de bibliotecas.

Concorrência não é um recurso, *per se*. É um novo mecanismo nos bastidores que permite que o React prepare várias versões da sua UI ao mesmo tempo. Você pode pensar na concorrência como um detalhe de implementação — ela é valiosa por causa dos recursos que desbloqueia. O React usa técnicas sofisticadas em sua implementação interna, como filas de prioridade e *multiple buffering*. Mas você não verá esses conceitos em nenhum lugar em nossas APIs públicas.

Ao projetar APIs, tentamos ocultar os detalhes da implementação dos desenvolvedores. Como um desenvolvedor React, você se concentra no *que* deseja que a experiência do usuário pareça e o React lida com *como* fornecer essa experiência. Portanto, não esperamos que os desenvolvedores React saibam como a concorrência funciona por dentro.

No entanto, o React Concorrente é mais importante do que um detalhe de implementação típico — é uma atualização fundamental para o modelo de renderização principal do React. Portanto, embora não seja super importante saber como a concorrência funciona, pode valer a pena saber o que é em um alto nível.

Uma propriedade chave do React Concorrente é que a renderização é *interruptible*. Quando você atualiza pela primeira vez para o React 18, antes de adicionar quaisquer recursos concorrentes, as atualizações são renderizadas da mesma forma que nas versões anteriores do React — em uma única transação síncrona ininterrupta. Com a renderização síncrona, uma vez que uma atualização começa a renderizar, nada pode interrompê-la até que o usuário possa ver o resultado na tela.

Em uma renderização concorrente, este nem sempre é o caso. O React pode começar a renderizar uma atualização, pausar no meio e, em seguida, continuar mais tarde. Ele pode até abandonar uma renderização em andamento completamente. O React garante que a UI aparecerá consistente, mesmo que uma renderização seja interrompida. Para fazer isso, ele espera para executar mutações do DOM até o final, uma vez que a árvore inteira tenha sido avaliada. Com esse recurso, o React pode preparar novas telas em segundo plano sem bloquear a *main thread*. Isso significa que a UI pode responder imediatamente à entrada do usuário, mesmo que esteja no meio de uma grande tarefa de renderização, criando uma experiência do usuário fluida.

Outro exemplo é o estado reutilizável. O React Concorrente pode remover seções da UI da tela e, em seguida, adicioná-las de volta mais tarde, reutilizando o estado anterior. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de restaurar a tela anterior no mesmo estado em que estava antes. Em uma próxima versão secundária, planejamos adicionar um novo componente chamado `<Offscreen>` que implementa esse padrão. Da mesma forma, você poderá usar Offscreen para preparar a nova UI em segundo plano para que ela esteja pronta antes que o usuário a revele.

A renderização concorrente é uma nova e poderosa ferramenta no React e a maioria de nossos novos recursos são construídos para tirar proveito dela, incluindo Suspense, transitions e *streaming server rendering*. Mas o React 18 é apenas o começo do que pretendemos construir sobre essa nova base.

## Adotando gradualmente os recursos concorrentes {/*gradually-adopting-concurrent-features*/}

Tecnicamente, a renderização concorrente é uma *breaking change*. Como a renderização concorrente é *interruptible*, os componentes se comportam de maneira ligeiramente diferente quando ela é habilitada.

Em nossos testes, atualizamos milhares de componentes para React 18. O que descobrimos é que quase todos os componentes existentes "simplesmente funcionam" com a renderização concorrente, sem nenhuma alteração. No entanto, alguns deles podem exigir algum esforço de migração adicional. Embora as alterações geralmente sejam pequenas, você ainda terá a capacidade de fazê-las no seu próprio ritmo. O novo comportamento de renderização no React 18 é **habilitado apenas nas partes do seu aplicativo que usam novos recursos.**

A estratégia geral de atualização é fazer com que seu aplicativo seja executado no React 18 sem quebrar o código existente. Então você pode gradualmente começar a adicionar recursos concorrentes no seu próprio ritmo. Você pode usar [`<StrictMode>`](/reference/react/StrictMode) para ajudar a revelar erros relacionados à concorrência durante o desenvolvimento. O Strict Mode não afeta o comportamento de produção, mas durante o desenvolvimento fará o log de avisos extras e invocará funções duplamente que devem ser *idempotent*. Ele não pegará tudo, mas é eficaz para evitar os tipos mais comuns de erros.

Depois de atualizar para o React 18, você poderá começar a usar os recursos concorrentes imediatamente. Por exemplo, você pode usar `startTransition` para navegar entre as telas sem bloquear a entrada do usuário. Ou use `useDeferredValue` para *throttle* re-renders caros.

No entanto, a longo prazo, esperamos que a principal maneira de adicionar concorrência ao seu aplicativo seja usando uma biblioteca ou *framework* habilitado para concorrência. Na maioria dos casos, você não interagirá com as APIs concorrentes diretamente. Por exemplo, em vez de os desenvolvedores chamarem `startTransition` sempre que navegarem para uma nova tela, as bibliotecas de roteamento irão automaticamente encapsular as navegações em `startTransition`.

Pode levar algum tempo para que as bibliotecas sejam atualizadas para serem compatíveis com a concorrência. Fornecemos novas APIs para facilitar o aproveitamento dos recursos concorrentes pelas bibliotecas. Enquanto isso, seja paciente com os mantenedores enquanto trabalhamos para migrar gradualmente o ecossistema React.

Para mais informações, consulte nosso post anterior: [Como atualizar para React 18](/blog/2022/03/08/react-18-upgrade-guide).

## Suspense em Frameworks de Dados {/*suspense-in-data-frameworks*/}

No React 18, você pode começar a usar [Suspense](/reference/react/Suspense) para busca de dados em *frameworks* opinativos como Relay, Next.js, Hydrogen ou Remix. A busca de dados *ad hoc* com Suspense é tecnicamente possível, mas ainda não é recomendada como uma estratégia geral.

No futuro, podemos expor primitivos adicionais que poderiam facilitar o acesso aos seus dados com Suspense, talvez sem o uso de um *framework* opinativo. No entanto, o Suspense funciona melhor quando está profundamente integrado à arquitetura do seu aplicativo: seu roteador, sua camada de dados e seu ambiente de renderização do servidor. Portanto, mesmo a longo prazo, esperamos que bibliotecas e *frameworks* desempenhem um papel crucial no ecossistema React.

Como nas versões anteriores do React, você também pode usar o Suspense para *code splitting* no cliente com React.lazy. Mas nossa visão para o Suspense sempre foi sobre muito mais do que carregar código — o objetivo é estender o suporte para o Suspense para que, eventualmente, o mesmo *fallback* declarativo do Suspense possa lidar com qualquer operação assíncrona (carregamento de código, dados, imagens, etc).

## Server Components ainda está em desenvolvimento {/*server-components-is-still-in-development*/}

[**Server Components**](/blog/2020/12/21/data-fetching-with-react-server-components) é um recurso futuro que permite aos desenvolvedores criar aplicativos que abrangem o servidor e o cliente, combinando a rica interatividade dos aplicativos do lado do cliente com o desempenho aprimorado da renderização tradicional do servidor. Server Components não é inerentemente acoplado ao React Concorrente, mas é projetado para funcionar melhor com recursos concorrentes, como Suspense e *streaming server rendering*.

Server Components ainda é experimental, mas esperamos lançar uma versão inicial em uma versão secundária 18.x. Enquanto isso, estamos trabalhando com *frameworks* como Next.js, Hydrogen e Remix para promover a proposta e prepará-la para ampla adoção.

## O que há de novo no React 18 {/*whats-new-in-react-18*/}

### Novo Recurso: Automatic Batching {/*new-feature-automatic-batching*/}

*Batching* é quando o React agrupa várias atualizações de estado em um único re-render para um melhor desempenho. Sem *automatic batching*, nós só agrupamos atualizações dentro de *event handlers* do React. Atualizações dentro de *promises*, setTimeout, *native event handlers* ou qualquer outro evento não foram agrupadas no React por padrão. Com *automatic batching*, essas atualizações serão agrupadas automaticamente:

```js
// Antes: apenas eventos do React foram agrupados.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar duas vezes, uma para cada atualização de estado (sem agrupamento)
}, 1000);

// Depois: atualizações dentro de timeouts, promises,
// native event handlers ou qualquer outro evento são agrupadas.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React vai renderizar apenas uma vez no final (isso é agrupamento!)
}, 1000);
```

Para mais informações, consulte este post para [Automatic batching for fewer renders in React 18](https://github.com/reactwg/react-18/discussions/21).

### Novo Recurso: Transitions {/*new-feature-transitions*/}

Uma *transition* é um novo conceito no React para distinguir entre atualizações urgentes e não urgentes.

*   **Atualizações urgentes** refletem a interação direta, como digitação, cliques, pressionamentos e assim por diante.
*   **Atualizações de transition** fazem a transição da UI de uma exibição para outra.

Atualizações urgentes como digitação, cliques ou pressionamentos precisam de resposta imediata para corresponder às nossas intuições sobre como os objetos físicos se comportam. Caso contrário, eles parecem "errados". No entanto, as *transitions* são diferentes porque o usuário não espera ver todos os valores intermediários na tela.

Por exemplo, ao selecionar um filtro em um *dropdown*, você espera que o próprio botão do filtro responda imediatamente quando você clica. No entanto, os resultados reais podem fazer a transição separadamente. Um pequeno atraso seria imperceptível e geralmente esperado. E se você alterar o filtro novamente antes que os resultados terminem a renderização, você só se importa em ver os resultados mais recentes.

Normalmente, para a melhor experiência do usuário, uma única entrada do usuário deve resultar em uma atualização urgente e outra não urgente. Você pode usar a API `startTransition` dentro de um evento de *input* para informar ao React quais atualizações são urgentes e quais são "transitions":

```js
import { startTransition } from 'react';

// Urgente: Mostrar o que foi digitado
setInputValue(input);

// Marcar quaisquer atualizações de estado como transitions
startTransition(() => {
  // Transition: Mostrar os resultados
  setSearchQuery(input);
});
```

As atualizações encapsuladas em `startTransition` são tratadas como não urgentes e serão interrompidas se atualizações mais urgentes, como cliques ou pressionamentos de tecla, entrarem. Se uma *transition* for interrompida pelo usuário (por exemplo, digitando vários caracteres em uma linha), o React descartará o trabalho de renderização desatualizado que não foi finalizado e renderizará apenas a atualização mais recente.

*   `useTransition`: um *Hook* para iniciar *transitions*, incluindo um valor para rastrear o estado pendente.
*   `startTransition`: um método para iniciar *transitions* quando o *Hook* não pode ser usado.

*Transitions* optarão por renderização concorrente, o que permite que a atualização seja interrompida. Se o conteúdo re-suspender, as *transitions* também informam ao React para continuar mostrando o conteúdo atual enquanto renderizam o conteúdo da *transition* em segundo plano (consulte o [RFC do Suspense](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md) para obter mais informações).

[Veja a documentação para *transitions* aqui](/reference/react/useTransition).

### Novos recursos de Suspense {/*new-suspense-features*/}

O Suspense permite que você especifique declarativamente o estado de carregamento para uma parte da árvore de componentes, caso ela ainda não esteja pronta para ser exibida:

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

Suspense torna o "estado de carregamento da UI" um conceito declarativo de primeira classe no modelo de programação do React. Isso nos permite construir recursos de nível superior em cima dele.

Apresentamos uma versão limitada do Suspense há vários anos. No entanto, o único caso de uso suportado era *code splitting* com React.lazy e não era suportado ao renderizar no servidor.

No React 18, adicionamos suporte para Suspense no servidor e expandimos seus recursos usando os recursos de renderização concorrente.

Suspense no React 18 funciona melhor quando combinado com a API de *transition*. Se você suspender durante uma *transition*, o React impedirá que o conteúdo já visível seja substituído por um *fallback*. Em vez disso, o React atrasará a renderização até que dados suficientes sejam carregados para evitar um estado de carregamento ruim.

Para mais informações, consulte o RFC para [Suspense no React 18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md).

### Novas APIs de renderização do cliente e do servidor {/*new-client-and-server-rendering-apis*/}

Nesta versão, aproveitamos a oportunidade para redesenhar as APIs que expomos para renderização no cliente e no servidor. Essas alterações permitem que os usuários continuem usando as APIs antigas no modo React 17 enquanto atualizam para as novas APIs no React 18.

#### React DOM Client {/*react-dom-client*/}

Essas novas APIs agora são exportadas do `react-dom/client`:

*   `createRoot`: Novo método para criar uma raiz para `renderizar` ou `desmontar`. Use-o em vez de `ReactDOM.render`. Novos recursos no React 18 não funcionam sem ele.
*   `hydrateRoot`: Novo método para *hydratar* um aplicativo renderizado no servidor. Use-o em vez de `ReactDOM.hydrate` em conjunto com as novas APIs do React DOM Server. Novos recursos no React 18 não funcionam sem ele.

`createRoot` e `hydrateRoot` aceitam uma nova opção chamada `onRecoverableError` caso você queira ser notificado quando o React se recuperar de erros durante a renderização ou *hydratation* para registro. Por padrão, o React usará [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError), ou `console.error` nos navegadores mais antigos.

[Veja a documentação para React DOM Client aqui](/reference/react-dom/client).

#### React DOM Server {/*react-dom-server*/}

Essas novas APIs agora são exportadas de `react-dom/server` e possuem suporte total para *streaming* do Suspense no servidor:

*   `renderToPipeableStream`: para *streaming* em ambientes Node.
*   `renderToReadableStream`: para modernos ambientes de *runtime* de borda, como Deno e Cloudflare workers.

O método `renderToString` existente continua funcionando, mas é desencorajado.

[Veja a documentação para React DOM Server aqui](/reference/react-dom/server).

### Novos comportamentos do Strict Mode {/*new-strict-mode-behaviors*/}

No futuro, gostaríamos de adicionar um recurso que permite ao React adicionar e remover seções da UI, preservando o estado. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado de componente de antes.

Esse recurso dará aos aplicativos React melhor desempenho *out-of-the-box*, mas exige que os componentes sejam resilientes a ter efeitos montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos pressupõem que eles sejam montados ou destruídos apenas uma vez.

Para ajudar a identificar esses problemas, o React 18 apresenta uma nova verificação apenas para desenvolvimento no Strict Mode. Essa nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes dessa alteração, o React montaria o componente e criaria os efeitos:
``````
* React renderiza o componente.
  * Efeitos de layout são criados.
  * Efeitos são criados.
```

Com o Modo Estrito no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

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

 `useId` é um novo Hook para gerar IDs únicos no cliente e no servidor, evitando incompatibilidades de hidratação. É fundamentalmente útil para bibliotecas de componentes que se integram com APIs de acessibilidade que exigem IDs únicos. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 por causa da forma como o novo renderizador de servidor de streaming entrega HTML fora de ordem. [Veja a documentação aqui](/reference/react/useId).

> Nota
>
> `useId` **não** serve para gerar [chaves em uma lista](/learn/rendering-lists#where-to-get-your-key). As chaves devem ser geradas a partir de seus dados.

#### useTransition {/*usetransition*/}

`useTransition` e `startTransition` permitem que você marque algumas atualizações de estado como não urgentes. Outras atualizações de estado são consideradas urgentes por padrão. O React permitirá que atualizações de estado urgentes (por exemplo, atualizar uma entrada de texto) interrompam atualizações de estado não urgentes (por exemplo, renderizar uma lista de resultados de pesquisa). [Veja a documentação aqui](/reference/react/useTransition).

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` permite que você adie a renderização de uma parte não urgente da árvore. Ele é semelhante ao debouncing, mas tem algumas vantagens em comparação com ele. Não há atraso de tempo fixo, então o React tentará a renderização adiada logo após a primeira renderização ser refletida na tela. A renderização adiada é interrompível e não bloqueia a entrada do usuário. [Veja a documentação aqui](/reference/react/useDeferredValue).

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` é um novo Hook que permite que lojas externas suportem leituras simultâneas, forçando as atualizações na loja a serem síncronas. Ele remove a necessidade de useEffect ao implementar assinaturas em fontes de dados externas, e é recomendado para qualquer biblioteca que se integre com estado externo ao React. [Veja a documentação aqui](/reference/react/useSyncExternalStore).

> Nota
>
> `useSyncExternalStore` é destinado a ser usado por bibliotecas, não pelo código do aplicativo.

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos em renderização. A menos que você já tenha criado uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização simultânea, dando a ele uma chance de recalcular o layout. [Veja a documentação aqui](/reference/react/useInsertionEffect).

> Nota
>
> `useInsertionEffect` é destinado a ser usado por bibliotecas, não pelo código do aplicativo.

## Como Atualizar {/*how-to-upgrade*/}

Veja [Como Atualizar para React 18](/blog/2022/03/08/react-18-upgrade-guide) para obter instruções passo a passo e uma lista completa de alterações importantes e notáveis.

## Changelog {/*changelog*/}

### React {/*react*/}

* Adiciona `useTransition` e `useDeferredValue` para separar atualizações urgentes de transições. ([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) by [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), and [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `useId` para gerar IDs únicos. ([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) by [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), and [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `useSyncExternalStore` para ajudar bibliotecas de lojas externas a se integrarem ao React. ([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) by [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), and [@drarmstr](https://github.com/drarmstr))
* Adiciona `startTransition` como uma versão de `useTransition` sem feedback pendente. ([#19696](https://github.com/facebook/react/pull/19696)  by [@rickhanlonii](https://github.com/rickhanlonii))
* Adiciona `useInsertionEffect` para bibliotecas CSS-in-JS. ([#21913](https://github.com/facebook/react/pull/21913)  by [@rickhanlonii](https://github.com/rickhanlonii))
* Faz com que Suspense remonte os efeitos de layout quando o conteúdo reaparece.  ([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) by [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), and [@lunaruan](https://github.com/lunaruan))
* Faço `<StrictMode>` reexecutar efeitos para verificar o estado restaurável. ([#19523](https://github.com/facebook/react/pull/19523) , [#21418](https://github.com/facebook/react/pull/21418)  by [@bvaughn](https://github.com/bvaughn) and [@lunaruan](https://github.com/lunaruan))
* Assume que Symbols estão sempre disponíveis. ([#23348](https://github.com/facebook/react/pull/23348)  by [@sebmarkbage](https://github.com/sebmarkbage))
* Remove o polyfill `object-assign`. ([#23351](https://github.com/facebook/react/pull/23351)  by [@sebmarkbage](https://github.com/sebmarkbage))
* Remove a API `unstable_changedBits` não suportada.  ([#20953](https://github.com/facebook/react/pull/20953)  by [@acdlite](https://github.com/acdlite))
* Permite que os componentes renderizem undefined. ([#21869](https://github.com/facebook/react/pull/21869)  by [@rickhanlonii](https://github.com/rickhanlonii))
* Libera `useEffect` resultantes de eventos discretos como cliques de forma síncrona. ([#21150](https://github.com/facebook/react/pull/21150)  by [@acdlite](https://github.com/acdlite))
* Suspense `fallback={undefined}` agora se comporta da mesma forma que `null` e não é ignorado. ([#21854](https://github.com/facebook/react/pull/21854)  by [@rickhanlonii](https://github.com/rickhanlonii))
* Considera que todos os `lazy()` que resolvem o mesmo componente são equivalentes. ([#20357](https://github.com/facebook/react/pull/20357)  by [@sebmarkbage](https://github.com/sebmarkbage))
* Não corrige o console durante a primeira renderização. ([#22308](https://github.com/facebook/react/pull/22308)  by [@lunaruan](https://github.com/lunaruan))
* Melhora o uso da memória. ([#21039](https://github.com/facebook/react/pull/21039)  by [@bgirard](https://github.com/bgirard))
* Melhora as mensagens se a coerção de string lançar (Temporal.*, Symbol, etc.) ([#22064](https://github.com/facebook/react/pull/22064)  by [@justingrant](https://github.com/justingrant))
* Usa `setImmediate` quando disponível em vez de `MessageChannel`. ([#20834](https://github.com/facebook/react/pull/20834)  by [@gaearon](https://github.com/gaearon))
* Corrige o contexto que não propaga dentro de árvores suspensas. ([#23095](https://github.com/facebook/react/pull/23095)  by [@gaearon](https://github.com/gaearon))
* Corrige `useReducer` observando props incorretas removendo o mecanismo de saída ansiosa. ([#22445](https://github.com/facebook/react/pull/22445)  by [@josephsavona](https://github.com/josephsavona))
* Corrige o `setState` sendo ignorado no Safari ao anexar iframes. ([#23111](https://github.com/facebook/react/pull/23111)  by [@gaearon](https://github.com/gaearon))
* Corrige uma falha ao renderizar `ZonedDateTime` na árvore. ([#20617](https://github.com/facebook/react/pull/20617)  by [@dimaqq](https://github.com/dimaqq))
* Corrige uma falha quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695)  by [@SimenB](https://github.com/SimenB))
* Corrige `onLoad` não sendo acionado quando recursos simultâneos estão ativados. ([#23316](https://github.com/facebook/react/pull/23316)  by [@gnoff](https://github.com/gnoff))
* Corrige um aviso quando um seletor retorna `NaN`.  ([#23333](https://github.com/facebook/react/pull/23333)  by [@hachibeeDI](https://github.com/hachibeeDI))
* Corrige uma falha quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) by [@SimenB](https://github.com/SimenB))
* Corrige o cabeçalho de licença gerado. ([#23004](https://github.com/facebook/react/pull/23004)  by [@vitaliemiron](https://github.com/vitaliemiron))
* Adiciona `package.json` como um dos pontos de entrada. ([#22954](https://github.com/facebook/react/pull/22954)  by [@Jack](https://github.com/Jack-Works))
* Permite suspender fora de um limite de Suspense. ([#23267](https://github.com/facebook/react/pull/23267)  by [@acdlite](https://github.com/acdlite))
* Registra um erro recuperável sempre que a hidratação falha. ([#23319](https://github.com/facebook/react/pull/23319)  by [@acdlite](https://github.com/acdlite))

### React DOM {/*react-dom*/}

* Adiciona `createRoot` e `hydrateRoot`. ([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/pull/20888), [#21072](https://github.com/facebook/react/pull/21072), [#21417](https://github.com/facebook/react/pull/21417), [#21652](https://github.com/facebook/react/pull/21652), [#21687](https://github.com/facebook/react/pull/21687), [#23207](https://github.com/facebook/react/pull/23207), [#23385](https://github.com/facebook/react/pull/23385) by [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), [@gaearon](https://github.com/gaearon), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), [@trueadm](https://github.com/trueadm), and [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona hidratação seletiva. ([#14717](https://github.com/facebook/react/pull/14717), [#14884](https://github.com/facebook/react/pull/14884), [#16725](https://github.com/facebook/react/pull/16725), [#16880](https://github.com/facebook/react/pull/16880), [#17004](https://github.com/facebook/react/pull/17004), [#22416](https://github.com/facebook/react/pull/22416), [#22629](https://github.com/facebook/react/pull/22629), [#22448](https://github.com/facebook/react/pull/22448), [#22856](https://github.com/facebook/react/pull/22856), [#23176](https://github.com/facebook/react/pull/23176) by [@acdlite](https://github.com/acdlite), [@gaearon](https://github.com/gaearon), [@salazarm](https://github.com/salazarm), and [@sebmarkbage](https://github.com/sebmarkbage))
* Adiciona `aria-description` à lista de atributos ARIA conhecidos. ([#22142](https://github.com/facebook/react/pull/22142)  by [@mahyareb](https://github.com/mahyareb))
* Adiciona o evento `onResize` aos elementos de vídeo. ([#21973](https://github.com/facebook/react/pull/21973)  by [@rileyjshaw](https://github.com/rileyjshaw))
* Adiciona `imageSizes` e `imageSrcSet` às props conhecidas. ([#22550](https://github.com/facebook/react/pull/22550)  by [@eps1lon](https://github.com/eps1lon))
* Permite filhos de `<option>` que não são string se `value` for fornecido.  ([#21431](https://github.com/facebook/react/pull/21431)  by [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige `aspectRatio` style não sendo aplicada. ([#21100](https://github.com/facebook/react/pull/21100)  by [@gaearon](https://github.com/gaearon))
* Avisa se `renderSubtreeIntoContainer` for chamado. ([#23355](https://github.com/facebook/react/pull/23355)  by [@acdlite](https://github.com/acdlite))

### React DOM Server {/*react-dom-server-1*/}

* Adiciona o novo renderizador de streaming. ([#14144](https://github.com/facebook/react/pull/14144), [#20970](https://github.com/facebook/react/pull/20970), [#21056](https://github.com/facebook/react/pull/21056), [#21255](https://github.com/facebook/react/pull/21255), [#21200](https://github.com/facebook/react/pull/21200), [#21257](https://github.com/facebook/react/pull/21257), [#21276](https://github.com/facebook/react/pull/21276), [#22443](https://github.com/facebook/react/pull/22443), [#22450](https://github.com/facebook/react/pull/22450), [#23247](https://github.com/facebook/react/pull/23247), [#24025](https://github.com/facebook/react/pull/24025), [#24030](https://github.com/facebook/react/pull/24030) by [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige provedores de contexto em SSR ao lidar com várias solicitações. ([#23171](https://github.com/facebook/react/pull/23171)  by [@frandiox](https://github.com/frandiox))
* Reverte para renderização do cliente em incompatibilidade de texto. ([#23354](https://github.com/facebook/react/pull/23354)  by [@

* Remove a função `renderToNodeStream`. ([#23359](https://github.com/facebook/react/pull/23359)  by [@sebmarkbage](https://github.com/sebmarkbage))
* Corrige um log de erro falso no novo renderizador do servidor. ([#24043](https://github.com/facebook/react/pull/24043)  by [@eps1lon](https://github.com/eps1lon))
* Corrige um erro no novo renderizador do servidor. ([#22617](https://github.com/facebook/react/pull/22617)  by [@shuding](https://github.com/shuding))
* Ignora valores de função e símbolo dentro de elementos personalizados no servidor. ([#21157](https://github.com/facebook/react/pull/21157)  by [@sebmarkbage](https://github.com/sebmarkbage))

### React DOM Test Utils {/*react-dom-test-utils*/}

* Lança exceção quando `act` é usado em produção. ([#21686](https://github.com/facebook/react/pull/21686)  by [@acdlite](https://github.com/acdlite))
* Suporta a desativação de avisos de ato falsos com `global.IS_REACT_ACT_ENVIRONMENT`. ([#22561](https://github.com/facebook/react/pull/22561)  by [@acdlite](https://github.com/acdlite))
* Expande o aviso de ato para cobrir todas as APIs que podem agendar o trabalho do React. ([#22607](https://github.com/facebook/react/pull/22607)  by [@acdlite](https://github.com/acdlite))
* Faz `act` atualizar em lote. ([#21797](https://github.com/facebook/react/pull/21797)  by [@acdlite](https://github.com/acdlite))
* Remove o aviso para efeitos passivos pendentes. ([#22609](https://github.com/facebook/react/pull/22609)  by [@acdlite](https://github.com/acdlite))

### React Refresh {/*react-refresh*/}

* Rastreia raízes montadas tardiamente no Fast Refresh. ([#22740](https://github.com/facebook/react/pull/22740)  by [@anc95](https://github.com/anc95))
* Adiciona o campo `exports` a `package.json`. ([#23087](https://github.com/facebook/react/pull/23087)  by [@otakustay](https://github.com/otakustay))

### Server Components (Experimental) {/*server-components-experimental*/}

* Adiciona suporte a Server Context. ([#23244](https://github.com/facebook/react/pull/23244)  by [@salazarm](https://github.com/salazarm))
* Adiciona suporte a `lazy`. ([#24068](https://github.com/facebook/react/pull/24068)  by [@gnoff](https://github.com/gnoff))
* Atualiza o plugin de webpack para webpack 5 ([#22739](https://github.com/facebook/react/pull/22739)  by [@michenly](https://github.com/michenly))
* Corrige um engano no carregador do Node. ([#22537](https://github.com/facebook/react/pull/22537)  by [@btea](https://github.com/btea))
* Usa `globalThis` em vez de `window` para ambientes de borda. ([#22777](https://github.com/facebook/react/pull/22777)  by [@huozhi](https://github.com/huozhi))
```