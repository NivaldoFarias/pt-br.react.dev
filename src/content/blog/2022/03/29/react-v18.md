html
---
title: "React v18.0"
author: A Equipe React
date: 2022/03/08
description: React 18 está disponível no npm! Em nosso último post, compartilhamos instruções passo a passo para atualizar seu aplicativo para o React 18. Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.
---

29 de março de 2022 por [A Equipe React](/community/team)

---

<Intro>

React 18 já está disponível no npm! Em nosso último post, compartilhamos instruções passo a passo para [atualizar seu aplicativo para React 18](/blog/2022/03/08/react-18-upgrade-guide). Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.

</Intro>

---

Nossa última versão principal inclui melhorias prontas para uso, como *batching* automático, novas APIs como *startTransition* e *streaming server-side rendering* com suporte para *Suspense*.

Muitos dos recursos no React 18 são construídos em cima do nosso novo *concurrent renderer*, uma mudança nos bastidores que desbloqueia novos recursos poderosos. O React Concorrente é opcional — ele só é ativado quando você usa um recurso concorrente — mas achamos que isso terá um grande impacto na maneira como as pessoas constroem aplicativos.

Passamos anos pesquisando e desenvolvendo suporte para concorrência no React e tomamos cuidado extra para fornecer um caminho de adoção gradual para os usuários existentes. No verão passado, [formamos o React 18 Working Group](/blog/2021/06/08/the-plan-for-react-18) para coletar *feedback* de especialistas da comunidade e garantir uma experiência de atualização tranquila para todo o ecossistema React.

Caso você tenha perdido, compartilhamos grande parte dessa visão no React Conf 2021:

* Na [palestra principal](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa), explicamos como o React 18 se encaixa em nossa missão de facilitar o desenvolvimento de ótimas experiências do usuário
* [Shruti Kapoor](https://twitter.com/shrutikapoor08) [demonstrou como usar os novos recursos do React 18](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
* [Shaundai Person](https://twitter.com/shaundai) nos deu uma visão geral do [streaming server rendering com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

Abaixo está uma visão geral completa do que esperar nesta versão, começando com *Concurrent Rendering*.

<Note>

Para usuários do React Native, o React 18 será fornecido no React Native com a Nova Arquitetura React Native. Para obter mais informações, consulte a [palestra principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

## O que é React Concorrente? {/*what-is-concurrent-react*/}

A adição mais importante no React 18 é algo com o qual esperamos que você nunca precise se preocupar: concorrência. Achamos que isso é amplamente verdadeiro para desenvolvedores de aplicativos, embora a história possa ser um pouco mais complicada para os mantenedores de bibliotecas.

Concorrência não é um recurso, em si. É um novo mecanismo nos bastidores que permite que o React prepare várias versões da sua UI ao mesmo tempo. Você pode pensar na concorrência como um detalhe de implementação — ela é valiosa por causa dos recursos que desbloqueia. React usa técnicas sofisticadas em sua implementação interna, como filas de prioridade e *multiple buffering*. Mas você não verá esses conceitos em nenhum lugar em nossas APIs públicas.

Quando projetamos APIs, tentamos esconder os detalhes da implementação dos desenvolvedores. Como um desenvolvedor React, você se concentra no *que* deseja que a experiência do usuário se pareça e o React lida com *como* fornecer essa experiência. Portanto, não esperamos que os desenvolvedores React saibam como a concorrência funciona sob o capô.

No entanto, o React Concorrente é mais importante do que um detalhe de implementação típico — é uma atualização fundamental para o modelo de renderização central do React. Então, embora não seja super importante saber como a concorrência funciona, pode valer a pena saber o que ela é em alto nível.

Uma propriedade chave do React Concorrente é que a renderização é interrompível. Quando você atualiza pela primeira vez para o React 18, antes de adicionar quaisquer recursos concorrentes, as atualizações são renderizadas da mesma forma que nas versões anteriores do React — em uma única transação síncrona ininterrupta. Com a renderização síncrona, depois que uma atualização começa a renderizar, nada pode interrompê-la até que o usuário possa ver o resultado na tela.

Em uma renderização concorrente, este não é sempre o caso. React pode começar a renderizar uma atualização, pausar no meio e continuar mais tarde. Pode até abandonar uma renderização em andamento completamente. React garante que a UI apareça consistente, mesmo que uma renderização seja interrompida. Para fazer isso, ele espera para executar mutações do DOM até o final, uma vez que toda a árvore foi avaliada. Com esse recurso, o React pode preparar novas telas em segundo plano sem bloquear a *main thread*. Isso significa que a UI pode responder imediatamente à entrada do usuário, mesmo que esteja no meio de uma grande tarefa de renderização, criando uma experiência de usuário fluida.

Outro exemplo é o estado reutilizável. O React Concorrente pode remover seções da UI da tela e, em seguida, adicioná-las de volta posteriormente, reutilizando o estado anterior. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de restaurar a tela anterior no mesmo estado em que estava antes. Em um próximo lançamento secundário, estamos planejando adicionar um novo componente chamado `<Offscreen>` que implementa este padrão. Da mesma forma, você poderá usar Offscreen para preparar novas UIs em segundo plano para que ela esteja pronta antes que o usuário a revele.

A renderização concorrente é uma nova e poderosa ferramenta no React e a maioria de nossos novos recursos são construídos para tirar proveito dela, incluindo *Suspense*, transições e *streaming server rendering*. Mas o React 18 é apenas o começo do que pretendemos construir sobre essa nova base.

## Adotando Gradualmente os Recursos Concorrentes {/*gradually-adopting-concurrent-features*/}

Tecnicamente, a renderização concorrente é uma mudança *breaking*. Como a renderização concorrente é interrompível, os componentes se comportam de maneira um pouco diferente quando ela é ativada.

Em nossos testes, atualizamos milhares de componentes para React 18. O que descobrimos é que quase todos os componentes existentes "simplesmente funcionam" com a renderização concorrente, sem nenhuma alteração. No entanto, alguns deles podem exigir algum esforço de migração adicional. Embora as alterações geralmente sejam pequenas, você ainda terá a capacidade de fazê-las no seu próprio ritmo. O novo comportamento de renderização no React 18 **só é ativado nas partes do seu aplicativo que usam novos recursos.**

A estratégia geral de atualização é fazer com que seu aplicativo seja executado no React 18 sem quebrar o código existente. Então você pode gradualmente começar a adicionar recursos concorrentes no seu próprio ritmo. Você pode usar [`<StrictMode>`](/reference/react/StrictMode) para ajudar a expor erros relacionados a concorrência durante o desenvolvimento. O *Strict Mode* não afeta o comportamento de produção, mas durante o desenvolvimento ele registrará avisos extras e invocará funções duas vezes que devem ser idempotentes. Ele não detectará tudo, mas é eficaz na prevenção dos tipos mais comuns de erros.

Depois de atualizar para o React 18, você poderá começar a usar os recursos concorrentes imediatamente. Por exemplo, você pode usar *startTransition* para navegar entre as telas sem bloquear a entrada do usuário. Ou *useDeferredValue* para limitar as novas renderizações caras.

No entanto, a longo prazo, esperamos que a principal maneira de adicionar concorrência ao seu aplicativo seja usando uma biblioteca ou *framework* compatível com concorrência. Na maioria dos casos, você não interagirá com as APIs concorrentes diretamente. Por exemplo, em vez de os desenvolvedores chamarem *startTransition* sempre que navegam para uma nova tela, as bibliotecas de roteamento automaticamente envolverão as navegações em *startTransition*.

Pode levar algum tempo para que as bibliotecas atualizem para serem compatíveis com a concorrência. Fornecemos novas APIs para facilitar o aproveitamento dos recursos concorrentes pelas bibliotecas. Enquanto isso, seja paciente com os mantenedores, pois estamos trabalhando para migrar gradualmente o ecossistema React.

Para mais informações, consulte nossa postagem anterior: [Como atualizar para React 18](/blog/2022/03/08/react-18-upgrade-guide).

## Suspense em *Data Frameworks* {/*suspense-in-data-frameworks*/}

No React 18, você pode começar a usar [Suspense](/reference/react/Suspense) para *data fetching* em *frameworks* opinativos como Relay, Next.js, Hydrogen ou Remix. *Data fetching* ad hoc com *Suspense* é tecnicamente possível, mas ainda não recomendado como uma estratégia geral.

No futuro, podemos expor primitivos adicionais que podem facilitar o acesso aos seus dados com *Suspense*, talvez sem o uso de um *framework* opinativo. No entanto, o *Suspense* funciona melhor quando está profundamente integrado à arquitetura do seu aplicativo: seu roteador, sua camada de dados e seu ambiente de renderização do servidor. Portanto, mesmo a longo prazo, esperamos que as bibliotecas e *frameworks* desempenhem um papel crucial no ecossistema React.

Como nas versões anteriores do React, você também pode usar *Suspense* para *code splitting* no cliente com React.lazy. Mas nossa visão para *Suspense* sempre foi muito mais do que carregar código — o objetivo é estender o suporte para *Suspense* para que, eventualmente, a mesma alternativa declarativa do *Suspense* possa lidar com qualquer operação assíncrona (carregamento de código, dados, imagens, etc).

## Componentes do Servidor ainda estão em Desenvolvimento {/*server-components-is-still-in-development*/}

[**Componentes do Servidor**](/blog/2020/12/21/data-fetching-with-react-server-components) é um recurso futuro que permite aos desenvolvedores criar aplicativos que abrangem o servidor e o cliente, combinando a rica interatividade de aplicativos do lado do cliente com o desempenho aprimorado da renderização tradicional do servidor. Os Componentes do Servidor não são inerentemente acoplados ao React Concorrente, mas são projetados para funcionar melhor com recursos concorrentes como *Suspense* e *streaming server rendering*.

Componentes do Servidor ainda é experimental, mas esperamos lançar uma versão inicial em um lançamento secundário 18.x. Enquanto isso, estamos trabalhando com *frameworks* como Next.js, Hydrogen e Remix para avançar na proposta e prepará-la para ampla adoção.

## O que há de novo no React 18 {/*whats-new-in-react-18*/}

### Novo Recurso: *Batching* Automático {/*new-feature-automatic-batching*/}

*Batching* é quando o React agrupa várias atualizações de estado em uma única nova renderização para melhor desempenho. Sem *batching* automático, só fizemos no *batching* atualizações dentro dos *event handlers* do React. As atualizações dentro de *promises*, *setTimeout*, *native event handlers* ou qualquer outro evento não foram *batched* no React por padrão. Com *batching* automático, essas atualizações serão *batched* automaticamente:

```js
// Antes: apenas eventos React foram batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React irá renderizar duas vezes, uma para cada atualização de estado (sem batching)
}, 1000);

// Depois: atualizações dentro de timeouts, promises,
// native event handlers ou qualquer outro evento são batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React só irá renderizar uma vez no final (isso é batching!)
}, 1000);
```

Para obter mais informações, consulte esta publicação para [Batching automático para menos renderizações no React 18](https://github.com/reactwg/react-18/discussions/21).

### Novo Recurso: Transições {/*new-feature-transitions*/}

Uma transição é um novo conceito no React para distinguir atualizações urgentes e não urgentes.

* **Atualizações urgentes** refletem interação direta, como digitação, clique, pressionamento e assim por diante.
* **Atualizações de transição** transicionam a UI de uma visualização para outra.

Atualizações urgentes como digitação, clique ou pressionamento precisam de resposta imediata para corresponder às nossas intuições sobre como os objetos físicos se comportam. Caso contrário, eles parecem "errados". No entanto, as transições são diferentes porque o usuário não espera ver todos os valores intermediários na tela.

Por exemplo, quando você seleciona um filtro em um menu suspenso, espera que o próprio botão de filtro responda imediatamente quando você clica. No entanto, os resultados reais podem fazer a transição separadamente. Um pequeno atraso seria imperceptível e, muitas vezes, o esperado. E se você alterar o filtro novamente antes que os resultados terminem de renderizar, você só se importa em ver os resultados mais recentes.

Normalmente, para a melhor experiência do usuário, uma única entrada do usuário deve resultar em uma atualização urgente e uma não urgente. Você pode usar a API *startTransition* dentro de um evento de entrada para informar ao React quais atualizações são urgentes e quais são "transições":

```js
import { startTransition } from 'react';

// Urgente: Mostre o que foi digitado
setInputValue(input);

// Marque as atualizações de estado internas como transições
startTransition(() => {
  // Transição: Mostre os resultados
  setSearchQuery(input);
});
```

As atualizações encapsuladas em *startTransition* são tratadas como não urgentes e serão interrompidas se atualizações mais urgentes, como cliques ou pressionamentos de tecla, entrarem. Se uma transição for interrompida pelo usuário (por exemplo, digitando vários caracteres em uma linha), o React descartará o trabalho de renderização desatualizado que não foi finalizado e renderizará apenas a atualização mais recente.

* `useTransition`: um *Hook* para iniciar transições, incluindo um valor para rastrear o estado pendente.
* `startTransition`: um método para iniciar transições quando o *Hook* não pode ser usado.

As transições aceitarão a renderização concorrente, o que permite que a atualização seja interrompida. Se o conteúdo suspender, as transições também informam ao React para continuar mostrando o conteúdo atual enquanto renderizam o conteúdo da transição em segundo plano (consulte o [Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md) para obter mais informações).

[Veja os documentos para transições aqui](/reference/react/useTransition).

### Novos Recursos de Suspense {/*new-suspense-features*/}

Suspense permite especificar declarativamente o estado de carregamento para uma parte da árvore de componentes, se ela ainda não estiver pronta para ser exibida:

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

*Suspense* torna o "estado de carregamento da UI" um conceito declarativo de primeira classe no modelo de programação do React. Isso nos permite construir recursos de nível mais alto em cima dele.

Apresentamos uma versão limitada do *Suspense* há vários anos. No entanto, o único caso de uso suportado era o *code splitting* com React.lazy, e ele não era suportado de forma alguma ao renderizar no servidor.

No React 18, adicionamos suporte para *Suspense* no servidor e expandimos seus recursos usando recursos de renderização concorrentes.

O *Suspense* no React 18 funciona melhor quando combinado com a API de transição. Se você suspender durante uma transição, o React impedirá que o conteúdo já visível seja substituído por um fallback. Em vez disso, o React atrasará a renderização até que dados suficientes tenham sido carregados para evitar um estado de carregamento ruim.

Para mais informações, consulte o RFC para [Suspense no React 18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md).

### Novas APIs de Renderização do Cliente e do Servidor {/*new-client-and-server-rendering-apis*/}

Nesta versão, aproveitamos a oportunidade para redesenhar as APIs que expomos para renderização no cliente e no servidor. Essas mudanças permitem que os usuários continuem usando as APIs antigas no modo React 17 enquanto atualizam para as novas APIs no React 18.

#### React DOM Client {/*react-dom-client*/}

Essas novas APIs agora são exportadas de `react-dom/client`:

* `createRoot`: Novo método para criar uma raiz para `render` ou `unmount`. Use-o em vez de `ReactDOM.render`. Os novos recursos no React 18 não funcionam sem ele.
* `hydrateRoot`: Novo método para hidratar um aplicativo renderizado no servidor. Use-o em vez de `ReactDOM.hydrate` em conjunto com as novas APIs do React DOM Server. Os novos recursos no React 18 não funcionam sem ele.

`createRoot` e `hydrateRoot` aceitam uma nova opção chamada `onRecoverableError` caso você queira ser notificado quando o React se recuperar de erros durante a renderização ou hidratação para registro. Por padrão, o React usará [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError) ou `console.error` nos navegadores mais antigos.

[Veja os documentos para React DOM Client aqui](/reference/react-dom/client).

#### React DOM Server {/*react-dom-server*/}

Essas novas APIs agora são exportadas de `react-dom/server` e têm suporte total para *Suspense streaming* no servidor:

* `renderToPipeableStream`: para *streaming* em ambientes Node.
* `renderToReadableStream`: para ambientes de tempo de execução de ponta modernos, como o Deno e os *workers* do Cloudflare.

O método `renderToString` existente continua funcionando, mas é desencorajado.

[Veja os documentos para React DOM Server aqui](/reference/react-dom/server).

### Novos comportamentos do *Strict Mode* {/*new-strict-mode-behaviors*/}

No futuro, gostaríamos de adicionar um recurso que permite que o React adicione e remova seções da UI enquanto preserva o estado. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado do componente de antes.

Esse recurso dará aos aplicativos React melhor desempenho pronto para uso, mas exige que os componentes sejam resilientes a efeitos que são montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos presumem que são montados ou destruídos apenas uma vez.

Para ajudar a expor esses problemas, o React 18 introduz uma nova verificação de desenvolvimento apenas no *Strict Mode*. Esta nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes dessa alteração, o React montava o componente e criava os efeitos:
```* O React monta o componente.
  * Efeitos de layout são criados.
  * Os efeitos são criados.
```

Com o Modo Estrito no React 18, o React vai simular a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* O React monta o componente.
  * Efeitos de layout são criados.
  * Os efeitos são criados.
* O React simula a desmontagem do componente.
  * Efeitos de layout são destruídos.
  * Efeitos são destruídos.
* O React simula a montagem do componente com o estado anterior.
  * Efeitos de layout são criados.
  * Os efeitos são criados.
```

[Veja a documentação para garantir o estado reutilizável aqui](/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development).

### Novos Hooks {/*new-hooks*/}

#### useId {/*useid*/}

`useId` é um novo Hook para gerar IDs únicos tanto no cliente quanto no servidor, evitando incompatibilidades de hidratação. É usado principalmente para bibliotecas de componentes que se integram com as APIs de acessibilidade que exigem IDs únicos. Isso resolve um problema que já existe no React 17 e anteriores, mas é ainda mais importante no React 18 por causa da forma como o novo renderizador de servidor de streaming entrega o HTML fora de ordem. [Veja a documentação aqui](/reference/react/useId).

> Observação
>
> `useId` **não** é para gerar [chaves em uma lista](/learn/rendering-lists#where-to-get-your-key). As chaves devem ser geradas a partir de seus dados.

#### useTransition {/*usetransition*/}

`useTransition` e `startTransition` permitem que você marque algumas atualizações de estado como não urgentes. Outras atualizações de estado são consideradas urgentes por padrão. O React permitirá que as atualizações de estado urgentes (por exemplo, atualizar uma entrada de texto) interrompam as atualizações de estado não urgentes (por exemplo, renderizar uma lista de resultados da pesquisa). [Veja a documentação aqui](/reference/react/useTransition).

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` permite adiar a renderização de uma parte não urgente da árvore. Ele é semelhante a debouncing, mas tem algumas vantagens em comparação com ele. Não há atraso de tempo fixo, então o React tentará a renderização adiada logo após a primeira renderização ser refletida na tela. A renderização adiada é interrompível e não bloqueia a entrada do usuário. [Veja a documentação aqui](/reference/react/useDeferredValue).

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` é um novo Hook que permite que lojas externas suportem leituras simultâneas, forçando as atualizações na loja a serem síncronas. Ele remove a necessidade de useEffect ao implementar assinaturas em fontes de dados externas e é recomendado para qualquer biblioteca que se integre com o estado externo ao React. [Veja a documentação aqui](/reference/react/useSyncExternalStore).

> Observação
>
> `useSyncExternalStore` é destinado a ser usado por bibliotecas, não pelo código da aplicação.

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho da injeção de estilos na renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você o use. Este Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 porque o React cede para o navegador durante a renderização simultânea, dando a ele a chance de recalcular o layout. [Veja a documentação aqui](/reference/react/useInsertionEffect).

> Observação
>
> `useInsertionEffect` é destinado a ser usado por bibliotecas, não pelo código da aplicação.

## Como Fazer o Upgrade {/*how-to-upgrade*/}

Veja [Como Fazer o Upgrade para o React 18](/blog/2022/03/08/react-18-upgrade-guide) para obter instruções passo a passo e uma lista completa de mudanças importantes e notáveis.

## Changelog {/*changelog*/}

### React {/*react*/}

* Adicionado `useTransition` e `useDeferredValue` para separar atualizações urgentes de transições. ([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicionado `useId` para gerar IDs únicos. ([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicionado `useSyncExternalStore` para ajudar as bibliotecas de lojas externas a se integrarem ao React. ([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@drarmstr](https://github.com/drarmstr))
* Adicionado `startTransition` como uma versão de `useTransition` sem feedback pendente. ([#19696](https://github.com/facebook/react/pull/19696)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Adicionado `useInsertionEffect` para bibliotecas CSS-in-JS. ([#21913](https://github.com/facebook/react/pull/21913)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Faça com que os efeitos de layout do Suspense remontem quando o conteúdo reaparecer.  ([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Faça com que `<StrictMode>` reexecute efeitos para verificar o estado restaurável. ([#19523](https://github.com/facebook/react/pull/19523) , [#21418](https://github.com/facebook/react/pull/21418)  por [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Presuma que os símbolos estão sempre disponíveis. ([#23348](https://github.com/facebook/react/pull/23348)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Removido o polyfill `object-assign`. ([#23351](https://github.com/facebook/react/pull/23351)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Remover API `unstable_changedBits` sem suporte.  ([#20953](https://github.com/facebook/react/pull/20953)  por [@acdlite](https://github.com/acdlite))
* Permitir que componentes renderizem undefined. ([#21869](https://github.com/facebook/react/pull/21869)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Limpar `useEffect `resultante de eventos discretos como cliques de forma síncrona. ([#21150](https://github.com/facebook/react/pull/21150)  por [@acdlite](https://github.com/acdlite))
* Suspense `fallback={undefined}` agora se comporta da mesma forma que `null` e não é ignorado. ([#21854](https://github.com/facebook/react/pull/21854)  por [@rickhanlonii](https://github.com/rickhanlonii))
* Considere todos os `lazy()` resolvendo para o mesmo componente equivalente. ([#20357](https://github.com/facebook/react/pull/20357)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Não corrigir o console durante a primeira renderização. ([#22308](https://github.com/facebook/react/pull/22308)  por [@lunaruan](https://github.com/lunaruan))
* Melhorar o uso da memória. ([#21039](https://github.com/facebook/react/pull/21039)  por [@bgirard](https://github.com/bgirard))
* Melhorar mensagens se a coerção de string lançar (Temporal.*, Symbol, etc.) ([#22064](https://github.com/facebook/react/pull/22064)  por [@justingrant](https://github.com/justingrant))
* Usar `setImmediate` quando disponível em vez de `MessageChannel`. ([#20834](https://github.com/facebook/react/pull/20834)  por [@gaearon](https://github.com/gaearon))
* Corrigir contexto falhando em propagar dentro de árvores suspensas. ([#23095](https://github.com/facebook/react/pull/23095)  por [@gaearon](https://github.com/gaearon))
* Corrigir `useReducer` observando props incorretas removendo o mecanismo de saída ansioso. ([#22445](https://github.com/facebook/react/pull/22445)  por [@josephsavona](https://github.com/josephsavona))
* Corrigir `setState` sendo ignorado no Safari ao anexar iframes. ([#23111](https://github.com/facebook/react/pull/23111)  por [@gaearon](https://github.com/gaearon))
* Corrigir uma falha ao renderizar `ZonedDateTime` na árvore. ([#20617](https://github.com/facebook/react/pull/20617)  por [@dimaqq](https://github.com/dimaqq))
* Corrigir uma falha quando o documento está definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695)  por [@SimenB](https://github.com/SimenB))
* Corrigir `onLoad` não sendo acionado quando os recursos simultâneos estão ativados. ([#23316](https://github.com/facebook/react/pull/23316)  por [@gnoff](https://github.com/gnoff))
* Corrigir um aviso quando um seletor retorna `NaN`.  ([#23333](https://github.com/facebook/react/pull/23333)  por [@hachibeeDI](https://github.com/hachibeeDI))
* Corrigir uma falha quando o documento está definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) por [@SimenB](https://github.com/SimenB))
* Corrigir o cabeçalho da licença gerado. ([#23004](https://github.com/facebook/react/pull/23004)  por [@vitaliemiron](https://github.com/vitaliemiron))
* Adicione `package.json` como um dos pontos de entrada. ([#22954](https://github.com/facebook/react/pull/22954)  por [@Jack](https://github.com/Jack-Works))
* Permitir suspensão fora de uma limite Suspense. ([#23267](https://github.com/facebook/react/pull/23267)  por [@acdlite](https://github.com/acdlite))
* Registrar um erro recuperável sempre que a hidratação falhar. ([#23319](https://github.com/facebook/react/pull/23319)  por [@acdlite](https://github.com/acdlite))

### React DOM {/*react-dom*/}

* Adicione `createRoot` e `hydrateRoot`. ([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/pull/20888), [#21072](https://github.com/facebook/react/pull/21072), [#21417](https://github.com/facebook/react/pull/21417), [#21652](https://github.com/facebook/react/pull/21652), [#21687](https://github.com/facebook/react/pull/21687), [#23207](https://github.com/facebook/react/pull/23207), [#23385](https://github.com/facebook/react/pull/23385) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), [@gaearon](https://github.com/gaearon), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), [@trueadm](https://github.com/trueadm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione hidratação seletiva. ([#14717](https://github.com/facebook/react/pull/14717), [#14884](https://github.com/facebook/react/pull/14884), [#16725](https://github.com/facebook/react/pull/16725), [#16880](https://github.com/facebook/react/pull/16880), [#17004](https://github.com/facebook/react/pull/17004), [#22416](https://github.com/facebook/react/pull/22416), [#22629](https://github.com/facebook/react/pull/22629), [#22448](https://github.com/facebook/react/pull/22448), [#22856](https://github.com/facebook/react/pull/22856), [#23176](https://github.com/facebook/react/pull/23176) por [@acdlite](https://github.com/acdlite), [@gaearon](https://github.com/gaearon), [@salazarm](https://github.com/salazarm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione `aria-description` à lista de atributos ARIA conhecidos. ([#22142](https://github.com/facebook/react/pull/22142)  por [@mahyareb](https://github.com/mahyareb))
* Adicione o evento `onResize` aos elementos de vídeo. ([#21973](https://github.com/facebook/react/pull/21973)  por [@rileyjshaw](https://github.com/rileyjshaw))
* Adicione `imageSizes` e `imageSrcSet` às props conhecidas. ([#22550](https://github.com/facebook/react/pull/22550)  por [@eps1lon](https://github.com/eps1lon))
* Permitir filhos `<option>` que não sejam string se `value` for fornecido.  ([#21431](https://github.com/facebook/react/pull/21431)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrigir o estilo `aspectRatio` não sendo aplicado. ([#21100](https://github.com/facebook/react/pull/21100)  por [@gaearon](https://github.com/gaearon))
* Avisar se `renderSubtreeIntoContainer` for chamado. ([#23355](https://github.com/facebook/react/pull/23355)  por [@acdlite](https://github.com/acdlite))

### React DOM Server {/*react-dom-server-1*/}

* Adicione o novo renderizador de streaming. ([#14144](https://github.com/facebook/react/pull/14144), [#20970](https://github.com/facebook/react/pull/20970), [#21056](https://github.com/facebook/react/pull/21056), [#21255](https://github.com/facebook/react/pull/21255), [#21200](https://github.com/facebook/react/pull/21200), [#21257](https://github.com/facebook/react/pull/21257), [#21276](https://github.com/facebook/react/pull/21276), [#22443](https://github.com/facebook/react/pull/22443), [#22450](https://github.com/facebook/react/pull/22450), [#23247](https://github.com/facebook/react/pull/23247), [#24025](https://github.com/facebook/react/pull/24025), [#24030](https://github.com/facebook/react/pull/24030) por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrigir provedores de contexto no SSR ao lidar com várias solicitações. ([#23171](https://github.com/facebook/react/pull/23171)  por [@frandiox](https://github.com/frandiox))
* Reverter para renderização do cliente em incompatibilidade de texto. ([#23354](https://github.com/facebook/react/pull/23354)  por [@acdlite](https://github.com/acdlite))
* Descontinuar `renderToNodeStream`. ([#23359](https://github.com/facebook/react/pull/23359)  por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrigir um log de erro falso no novo renderizador do servidor. ([#24043](https://github.com/facebook/react/pull/24043)  por [@eps1lon](https://github.com/eps1lon))
* Corrigir um bug no novo renderizador do servidor. ([#22617](https://github.com/