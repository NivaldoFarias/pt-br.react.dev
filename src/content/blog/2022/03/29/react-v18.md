---
title: "React v18.0"
author: The React Team
date: 2022/03/08
description: React 18 já está disponível no npm! Em nossa última postagem, compartilhamos instruções passo a passo para atualizar seu aplicativo para o React 18. Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.
---

29 de março de 2022 por [The React Team](/community/team)

---

<Intro>

React 18 já está disponível no npm! Em nossa última postagem, compartilhamos instruções passo a passo para [atualizar seu aplicativo para o React 18](/blog/2022/03/08/react-18-upgrade-guide). Neste post, daremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.

</Intro>

---

Nossa última versão principal inclui melhorias imediatas, como agrupamento automático, novas APIs como startTransition e renderização em streaming do lado do servidor com suporte para Suspense.

Muitos dos recursos do React 18 são construídos com base em nosso novo renderizador concorrente, uma mudança nos bastidores que desbloqueia novas capacidades poderosas. O React concorrente é opt-in — só é ativado quando você usa um recurso concorrente — mas achamos que terá um grande impacto na forma como as pessoas constroem aplicativos.

Levamos anos pesquisando e desenvolvendo suporte para concorrência no React e tomamos um cuidado especial para fornecer um caminho de adoção gradual para os usuários existentes. No verão passado, [formamos o Grupo de Trabalho do React 18](/blog/2021/06/08/the-plan-for-react-18) para coletar feedback de especialistas da comunidade e garantir uma experiência de atualização tranquila para todo o ecossistema React.

Caso você tenha perdido, compartilhamos grande parte dessa visão na React Conf 2021:

* Na [keynote](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa), explicamos como o React 18 se encaixa em nossa missão de facilitar a vida dos desenvolvedores na construção de ótimas experiências de usuário
* [Shruti Kapoor](https://twitter.com/shrutikapoor08) [demonstrou como usar os novos recursos do React 18](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
* [Shaundai Person](https://twitter.com/shaundai) nos deu uma visão geral da [renderização em streaming no lado do servidor com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

Abaixo, você encontrará uma visão geral completa do que esperar nesta versão, começando com Renderização Concorrente.

<Note>

Para usuários do React Native, o React 18 será incluído no React Native com a Nova Arquitetura do React Native. Para mais informações, veja a [keynote da React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

## O que é o React Concorrente? {/*what-is-concurrent-react*/}

A adição mais importante no React 18 é algo que esperamos que você nunca precise pensar: concorrência. Achamos que isso é amplamente verdade para desenvolvedores de aplicativos, embora a situação possa ser um pouco mais complicada para mantenedores de bibliotecas.

Concorrência não é uma funcionalidade, por si só. É um novo mecanismo nos bastidores que permite ao React preparar várias versões de sua interface ao mesmo tempo. Você pode pensar na concorrência como um detalhe de implementação — é valioso por causa dos recursos que desbloqueia. O React usa técnicas sofisticadas em sua implementação interna, como filas de prioridade e múltiplos buffers. Mas você não verá esses conceitos em nossas APIs públicas.

Quando projetamos APIs, tentamos ocultar detalhes de implementação dos desenvolvedores. Como desenvolvedor React, você se concentra em *o que* deseja que a experiência do usuário pareça, e o React cuida de *como* entregar essa experiência. Portanto, não esperamos que os desenvolvedores React saibam como a concorrência funciona nos bastidores.

No entanto, o React Concorrente é mais importante do que um detalhe de implementação típico — é uma atualização fundamental no modelo central de renderização do React. Portanto, embora não seja super importante saber como a concorrência funciona, pode ser interessante saber o que é em um alto nível.

Uma propriedade chave do React Concorrente é que a renderização é interrompível. Quando você primeiro atualiza para o React 18, antes de adicionar quaisquer recursos concorrentes, as atualizações são renderizadas da mesma forma que nas versões anteriores do React — em uma única transação síncrona ininterrupta. Com a renderização síncrona, uma vez que uma atualização começa a ser renderizada, nada pode interrompê-la até que o usuário possa ver o resultado na tela.

Em uma renderização concorrente, esse não é sempre o caso. O React pode começar a renderizar uma atualização, pausar no meio, e continuar mais tarde. Ele pode até abandonar uma renderização em progresso completamente. O React garante que a interface do usuário parecerá consistente mesmo que uma renderização seja interrompida. Para fazer isso, ele espera para realizar mutações do DOM até o final, uma vez que toda a árvore tenha sido avaliada. Com essa capacidade, o React pode preparar novas telas em segundo plano sem bloquear a thread principal. Isso significa que a interface do usuário pode responder imediatamente à entrada do usuário, mesmo que esteja no meio de uma tarefa de renderização grande, criando uma experiência de usuário fluida.

Outro exemplo é o estado reutilizável. O React Concorrente pode remover seções da interface do usuário da tela e, em seguida, adicioná-las de volta mais tarde, reutilizando o estado anterior. Por exemplo, quando um usuário alterna de uma tela e volta, o React deve ser capaz de restaurar a tela anterior no mesmo estado em que estava antes. Em uma atualização menor futura, planejamos adicionar um novo componente chamado `<Offscreen>` que implementa esse padrão. Da mesma forma, você poderá usar Offscreen para preparar novas interfaces em segundo plano, para que estejam prontas antes que o usuário as revele.

A renderização concorrente é uma nova ferramenta poderosa no React e a maioria de nossos novos recursos foi construída para aproveitar isso, incluindo Suspense, transições e renderização em streaming do lado do servidor. Mas o React 18 é apenas o começo do que pretendemos construir sobre essa nova fundação.

## Adoção Gradual de Recursos Concorrentes {/*gradually-adopting-concurrent-features*/}

Tecnicamente, a renderização concorrente é uma mudança quebradora. Como a renderização concorrente é interrompível, os componentes se comportam de maneira ligeiramente diferente quando ela está ativada.

Em nossos testes, atualizamos milhares de componentes para o React 18. O que descobrimos é que quase todos os componentes existentes "simplesmente funcionam" com a renderização concorrente, sem alterações. No entanto, alguns deles podem exigir algum esforço adicional de migração. Embora as mudanças sejam geralmente pequenas, você ainda terá a capacidade de realizá-las em seu próprio ritmo. O novo comportamento de renderização no React 18 é **habilitado apenas nas partes de seu aplicativo que utilizam novos recursos.**

A estratégia geral de atualização é fazer seu aplicativo funcionar no React 18 sem quebrar o código existente. Depois, você pode começar a adicionar recursos concorrentes gradualmente, no seu próprio ritmo. Você pode usar [`<StrictMode>`](/reference/react/StrictMode) para ajudar a mostrar bugs relacionados à concorrência durante o desenvolvimento. O Modo Estrito não afeta o comportamento em produção, mas durante o desenvolvimento ele registrará avisos extras e invocará funções que se espera serem idempotentes duas vezes. Ele não capturará tudo, mas é eficaz em prevenir os tipos mais comuns de erros.

Depois que você atualizar para o React 18, poderá começar a usar recursos concorrentes imediatamente. Por exemplo, você pode usar startTransition para navegar entre telas sem bloquear a entrada do usuário. Ou usar useDeferredValue para limitar re-renders caros.

No entanto, a longo prazo, esperamos que a principal forma de você adicionar concorrência ao seu aplicativo seja usando uma biblioteca ou framework compatível com concorrência. Na maioria dos casos, você não interagirá com APIs concorrentes diretamente. Por exemplo, em vez de os desenvolvedores chamarem startTransition sempre que navegarem para uma nova tela, bibliotecas de roteamento automaticamente envolverão as navegações em startTransition.

Pode levar algum tempo para que as bibliotecas sejam atualizadas para serem compatíveis com concorrência. Fornecemos novas APIs para facilitar que as bibliotecas aproveitem os recursos concorrentes. Enquanto isso, por favor, tenha paciência com os mantenedores enquanto trabalhamos para migrar gradualmente o ecossistema React.

Para mais informações, veja nossa postagem anterior: [Como atualizar para o React 18](/blog/2022/03/08/react-18-upgrade-guide).

## Suspense em Frameworks de Dados {/*suspense-in-data-frameworks*/}

No React 18, você pode começar a usar [Suspense](/reference/react/Suspense) para busca de dados em frameworks opinativos como Relay, Next.js, Hydrogen ou Remix. A busca de dados ad hoc com Suspense é tecnicamente possível, mas ainda não é recomendada como uma estratégia geral.

No futuro, podemos expor primitivas adicionais que poderiam facilitar o acesso aos seus dados com Suspense, talvez sem o uso de um framework opinativo. No entanto, o Suspense funciona melhor quando está profundamente integrado na arquitetura de seu aplicativo: seu roteador, sua camada de dados e seu ambiente de renderização do lado do servidor. Portanto, mesmo a longo prazo, esperamos que bibliotecas e frameworks desempenhem um papel crucial no ecossistema React.

Como nas versões anteriores do React, você também pode usar Suspense para divisão de código no cliente com React.lazy. Mas nossa visão para o Suspense sempre foi sobre muito mais do que carregar código — o objetivo é estender o suporte para o Suspense para que, eventualmente, o mesmo fallback declarativo do Suspense possa lidar com qualquer operação assíncrona (carregando código, dados, imagens, etc).

## Componentes do Servidor Ainda Estão em Desenvolvimento {/*server-components-is-still-in-development*/}

[**Componentes do Servidor**](/blog/2020/12/21/data-fetching-with-react-server-components) é um recurso futuro que permite que os desenvolvedores criem aplicativos que se estendem do servidor ao cliente, combinando a rica interatividade de aplicativos do lado do cliente com o desempenho aprimorado da renderização do lado do servidor tradicional. Os Componentes do Servidor não estão intrinsecamente acoplados ao React Concorrente, mas são projetados para funcionar melhor com recursos concorrentes como Suspense e renderização em streaming do lado do servidor.

Os Componentes do Servidor ainda são experimentais, mas esperamos lançar uma versão inicial em uma atualização menor 18.x. Enquanto isso, estamos trabalhando com frameworks como Next.js, Hydrogen e Remix para avançar a proposta e prepará-la para uma ampla adoção.

## O que há de novo no React 18 {/*whats-new-in-react-18*/}

### Novo Recurso: Agrupamento Automático {/*new-feature-automatic-batching*/}

Agrupamento é quando o React agrupa várias atualizações de estado em uma única renderização para melhor desempenho. Sem agrupamento automático, apenas agrupávamos atualizações dentro de manipuladores de eventos do React. Atualizações dentro de promessas, setTimeout, manipuladores de eventos nativos ou qualquer outro evento não eram agrupadas no React por padrão. Com agrupamento automático, essas atualizações serão agrupadas automaticamente:

```js
// Antes: apenas eventos do React eram agrupados.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // O React irá renderizar duas vezes, uma vez para cada atualização de estado (sem agrupamento)
}, 1000);

// Depois: atualizações dentro de timeouts, promessas,
// manipuladores de eventos nativos ou qualquer outro evento são agrupados.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // O React irá re-renderizar apenas uma vez no final (isso é agrupar!)
}, 1000);
```

Para mais informações, veja esta postagem sobre [Agrupamento automático para menos renderizações no React 18](https://github.com/reactwg/react-18/discussions/21).

### Novo Recurso: Transições {/*new-feature-transitions*/}

Uma transição é um novo conceito no React para distinguir entre atualizações urgentes e não urgentes.

* **Atualizações urgentes** refletem interações diretas, como digitar, clicar, pressionar e assim por diante.
* **Atualizações de transição** fazem a interface do usuário passar de uma visão para outra.

Atualizações urgentes, como digitar, clicar ou pressionar, precisam de resposta imediata para corresponder às nossas intuições sobre como os objetos físicos se comportam. Caso contrário, elas parecem "erradas". No entanto, as transições são diferentes porque o usuário não espera ver todos os valores intermediários na tela.

Por exemplo, quando você seleciona um filtro em um dropdown, você espera que o botão de filtro responda imediatamente quando você clica. No entanto, os resultados reais podem transitar separadamente. Um pequeno atraso seria imperceptível e frequentemente esperado. E se você mudar o filtro novamente antes que os resultados terminem a renderização, você só quer ver os últimos resultados.

Geralmente, para a melhor experiência do usuário, uma única entrada do usuário deve resultar em tanto uma atualização urgente quanto uma não urgente. Você pode usar a API startTransition dentro de um evento de entrada para informar ao React quais atualizações são urgentes e quais são "transições":

```js
import { startTransition } from 'react';

// Urgente: Mostrar o que foi digitado
setInputValue(input);

// Marcar qualquer atualização de estado interior como transições
startTransition(() => {
  // Transição: Mostrar os resultados
  setSearchQuery(input);
});
```

Atualizações envolvidas em startTransition são tratadas como não urgentes e serão interrompidas se mais atualizações urgentes, como cliques ou pressionamentos de tecla, chegarem. Se uma transição for interrompida pelo usuário (por exemplo, digitando vários caracteres em sequência), o React descartará o trabalho de renderização desatualizado que não foi concluído e renderizará apenas a atualização mais recente.

* `useTransition`: um Hook para iniciar transições, incluindo um valor para rastrear o estado pendente.
* `startTransition`: um método para iniciar transições quando o Hook não pode ser usado.

Transições optarão por renderização concorrente, o que permite que a atualização seja interrompida. Se o conteúdo re-suspender, as transições também dizem ao React para continuar mostrando o conteúdo atual enquanto renderiza o conteúdo da transição em segundo plano (veja o [RFC do Suspense](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md) para mais informações).

[Veja a documentação sobre transições aqui](/reference/react/useTransition).

### Novos Recursos do Suspense {/*new-suspense-features*/}

Suspense permite que você especifique declarativamente o estado de carregamento para uma parte da árvore de componentes se ainda não estiver pronta para ser exibida:

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

Suspense torna o "estado de carregamento da interface do usuário" um conceito declarativo de primeira classe no modelo de programação do React. Isso nos permite construir recursos de nível superior em cima dele.

Introduzimos uma versão limitada do Suspense há vários anos. No entanto, o único caso de uso suportado era a divisão de código com React.lazy, e não era suportado de forma alguma ao renderizar no servidor.

No React 18, adicionamos suporte para Suspense no servidor e expandimos suas capacidades usando recursos de renderização concorrente.

Suspense no React 18 funciona melhor quando combinado com a API de transições. Se você suspender durante uma transição, o React impedirá que o conteúdo já visível seja substituído por um fallback. Em vez disso, o React atrasará a renderização até que dados suficientes tenham sido carregados para evitar um mau estado de carregamento.

Para mais informações, veja o RFC sobre [Suspense no React 18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md).

### Novas APIs de Renderização no Cliente e no Servidor {/*new-client-and-server-rendering-apis*/}

Nesta versão, aproveitamos a oportunidade para redesenhar as APIs que expomos para renderização no cliente e no servidor. Essas mudanças permitem que os usuários continuem usando as APIs antigas no modo React 17 enquanto atualizam para as novas APIs no React 18.

#### React DOM Cliente {/*react-dom-client*/}

Essas novas APIs agora são exportadas de `react-dom/client`:

* `createRoot`: Novo método para criar uma raiz para `renderizar` ou `desmontar`. Use-o em vez de `ReactDOM.render`. Novos recursos no React 18 não funcionam sem ele.
* `hydrateRoot`: Novo método para hidratar um aplicativo renderizado no servidor. Use-o em vez de `ReactDOM.hydrate` em conjunto com as novas APIs do React DOM Server. Novos recursos no React 18 não funcionam sem ele.

Tanto `createRoot` quanto `hydrateRoot` aceitam uma nova opção chamada `onRecoverableError` caso você queira ser notificado quando o React se recuperar de erros durante a renderização ou hidratação para registro. Por padrão, o React usará [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError), ou `console.error` em navegadores mais antigos.

[Veja a documentação para React DOM Cliente aqui](/reference/react-dom/client).

#### React DOM Servidor {/*react-dom-server*/}

Essas novas APIs agora são exportadas de `react-dom/server` e têm suporte total para streaming de Suspense no servidor:

* `renderToPipeableStream`: para streaming em ambientes Node.
* `renderToReadableStream`: para ambientes de runtime modernos, como Deno e Cloudflare workers.

O método existente `renderToString` continua funcionando, mas é desencorajado.

[Veja a documentação para React DOM Server aqui](/reference/react-dom/server).

### Novos Comportamentos do Modo Estrito {/*new-strict-mode-behaviors*/}

No futuro, gostaríamos de adicionar um recurso que permita ao React adicionar e remover seções da interface do usuário enquanto preserva o estado. Por exemplo, quando um usuário troca de tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado de componente de antes.

Esse recurso oferecerá aos aplicativos React melhor desempenho desde o início, mas requer que os componentes sejam resilientes para efeitos sendo montados e destruídos várias vezes. A maioria dos efeitos funcionará sem nenhuma alteração, mas alguns efeitos assumem que são montados ou destruídos apenas uma vez.

Para ajudar a mostrar esses problemas, o React 18 introduz uma nova verificação apenas para desenvolvimento no Modo Estrito. Essa nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes desta mudança, o React montaria o componente e criaria os efeitos:

```
* O React monta o componente.
  * Efeitos de layout são criados.
  * Efeitos são criados.
```

Com o Modo Estrito no React 18, o React simulará o desmontar e remontar o componente em modo de desenvolvimento:

```
* O React monta o componente.
  * Efeitos de layout são criados.
  * Efeitos são criados.
* O React simula o desmontar do componente.
  * Efeitos de layout são destruídos.
  * Efeitos são destruídos.
* O React simula montando o componente com o estado anterior.
  * Efeitos de layout são criados.
  * Efeitos são criados.
```

[Veja a documentação para garantir estado reutilizável aqui](/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development).

### Novos Hooks {/*new-hooks*/}

#### useId {/*useid*/}

`useId` é um novo Hook para gerar IDs únicos tanto no cliente quanto no servidor, enquanto evita disparidades de hidratação. É principalmente útil para bibliotecas de componentes integradas com APIs de acessibilidade que requerem IDs únicos. Isso resolve um problema que já existe no React 17 e abaixo, mas é ainda mais importante no React 18 devido à forma como o novo renderizador de servidor em streaming entrega HTML fora de ordem. [Veja a documentação aqui](/reference/react/useId).

> Nota
>
> `useId` **não** é para gerar [chaves em uma lista](/learn/rendering-lists#where-to-get-your-key). Chaves devem ser geradas a partir de seus dados.

#### useTransition {/*usetransition*/}

`useTransition` e `startTransition` permitem que você marque algumas atualizações de estado como não urgentes. Outras atualizações de estado são consideradas urgentes por padrão. O React permitirá que atualizações de estado urgentes (por exemplo, atualizar um campo de texto) interrompam atualizações de estado não urgentes (por exemplo, renderizar uma lista de resultados de busca). [Veja a documentação aqui](/reference/react/useTransition).

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` permite que você adie a re-renderização de uma parte não urgente da árvore. É semelhante ao debounce, mas tem algumas vantagens em comparação. Não há um atraso de tempo fixo, então o React tentará a renderização adiada logo após a primeira renderização ser refletida na tela. A renderização adiada é interrompível e não bloqueia a entrada do usuário. [Veja a documentação aqui](/reference/react/useDeferredValue).

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` é um novo Hook que permite que lojas externas suportem leituras concorrentes forçando atualizações na loja para serem síncronas. Ele remove a necessidade de useEffect ao implementar assinaturas para fontes de dados externas e é recomendado para qualquer biblioteca que integre com estado externo ao React. [Veja a documentação aqui](/reference/react/useSyncExternalStore).

> Nota
>
> `useSyncExternalStore` é destinado a ser usado por bibliotecas, não por código de aplicação.

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho ao injetar estilos durante a renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você jamais use isso. Esse Hook será executado após o DOM ser mutado, mas antes que os efeitos de layout leiam o novo layout. Isso resolve um problema que já existe no React 17 e inferior, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando-lhe a chance de recalcular o layout. [Veja a documentação aqui](/reference/react/useInsertionEffect).

> Nota
>
> `useInsertionEffect` é destinado a ser usado por bibliotecas, não por código de aplicação.

## Como Atualizar {/*how-to-upgrade*/}

Veja [Como atualizar para o React 18](/blog/2022/03/08/react-18-upgrade-guide) para instruções passo a passo e uma lista completa de mudanças quebradoras e notáveis.

## Changelog {/*changelog*/}

### React {/*react*/}

* Adicione `useTransition` e `useDeferredValue` para separar atualizações urgentes de transições. ([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione `useId` para gerar IDs únicos. ([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione `useSyncExternalStore` para ajudar bibliotecas de lojas externas a se integrarem com o React. ([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@drarmstr](https://github.com/drarmstr))
* Adicione `startTransition` como uma versão de `useTransition` sem feedback pendente. ([#19696](https://github.com/facebook/react/pull/19696) por [@rickhanlonii](https://github.com/facebook/react/pull/19696))
* Adicione `useInsertionEffect` para bibliotecas CSS-in-JS. ([#21913](https://github.com/facebook/react/pull/21913) por [@rickhanlonii](https://github.com/facebook/react/pull/21913))
* Faça o Suspense remontar efeitos de layout quando o conteúdo reaparecer. ([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Faça `<StrictMode>` re-executar efeitos para verificar estado restaurável. ([#19523](https://github.com/facebook/react/pull/19523), [#21418](https://github.com/facebook/react/pull/21418) por [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
* Assuma que os Symbols estão sempre disponíveis. ([#23348](https://github.com/facebook/react/pull/23348) por [@sebmarkbage](https://github.com/sebmarkbage))
* Remova o polyfill `object-assign`. ([#23351](https://github.com/facebook/react/pull/23351) por [@sebmarkbage](https://github.com/sebmarkbage))
* Remova a API `unstable_changedBits` não suportada. ([#20953](https://github.com/facebook/react/pull/20953) por [@acdlite](https://github.com/acdlite))
* Permita que componentes renderizem indefinidos. ([#21869](https://github.com/facebook/react/pull/21869) por [@rickhanlonii](https://github.com/rickhanlonii))
* Flush `useEffect` resultante de eventos discretos como cliques de forma síncrona. ([#21150](https://github.com/facebook/react/pull/21150) por [@acdlite](https://github.com/acdlite))
* O `fallback={undefined}` do Suspense agora se comporta da mesma forma que `null` e não é ignorado. ([#21854](https://github.com/facebook/react/pull/21854) por [@rickhanlonii](https://github.com/rickhanlonii))
* Considere todos os `lazy()` resolvendo para o mesmo componente como equivalente. ([#20357](https://github.com/facebook/react/pull/20357) por [@sebmarkbage](https://github.com/sebmarkbage))
* Não patche o console durante a primeira renderização. ([#22308](https://github.com/facebook/react/pull/22308) por [@lunaruan](https://github.com/lunaruan))
* Melhore o uso de memória. ([#21039](https://github.com/facebook/react/pull/21039) por [@bgirard](https://github.com/bgirard))
* Melhore as mensagens se a coercão de strings lançar (Temporal.*, Symbol, etc.) ([#22064](https://github.com/facebook/react/pull/22064) por [@justingrant](https://github.com/justingrant))
* Use `setImmediate` quando disponível em vez de `MessageChannel`. ([#20834](https://github.com/facebook/react/pull/20834) por [@gaearon](https://github.com/gaearon))
* Corrija a falha da propagação de contexto dentro de árvores suspensas. ([#23095](https://github.com/facebook/react/pull/23095) por [@gaearon](https://github.com/gaearon))
* Corrija o `useReducer` observando props incorretos removendo o mecanismo de interrupção ansiosa. ([#22445](https://github.com/facebook/react/pull/22445) por [@josephsavona](https://github.com/josephsavona))
* Corrija o `setState` sendo ignorado no Safari ao anexar iframes. ([#23111](https://github.com/facebook/react/pull/23111) por [@gaearon](https://github.com/gaearon))
* Corrija uma falha ao renderizar `ZonedDateTime` na árvore. ([#20617](https://github.com/facebook/react/pull/20617) por [@dimaqq](https://github.com/dimaqq))
* Corrija uma falha ao definir o documento como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) por [@SimenB](https://github.com/SimenB))
* Corrija `onLoad` não sendo acionado quando recursos concorrentes estão ativos. ([#23316](https://github.com/facebook/react/pull/23316) por [@gnoff](https://github.com/gnoff))
* Corrija um aviso quando um seletor retorna `NaN`. ([#23333](https://github.com/facebook/react/pull/23333) por [@hachibeeDI](https://github.com/hachibeeDI))
* Corrija uma falha ao definir o documento como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) por [@SimenB](https://github.com/SimenB))
* Corrija o cabeçalho de licença gerado. ([#23004](https://github.com/facebook/react/pull/23004) por [@vitaliemiron](https://github.com/vitaliemiron))
* Adicione `package.json` como um dos pontos de entrada. ([#22954](https://github.com/facebook/react/pull/22954) por [@Jack](https://github.com/Jack-Works))
* Permita suspender fora de uma boundary Suspense. ([#23267](https://github.com/facebook/react/pull/23267) por [@acdlite](https://github.com/acdlite))
* Registre um erro recuperável sempre que a hidratação falhar. ([#23319](https://github.com/facebook/react/pull/23319) por [@acdlite](https://github.com/acdlite))

### React DOM {/*react-dom*/}

* Adicione `createRoot` e `hydrateRoot`. ([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/pull/20888), [#21072](https://github.com/facebook/react/pull/21072), [#21417](https://github.com/facebook/react/pull/21417), [#21652](https://github.com/facebook/react/pull/21652), [#21687](https://github.com/facebook/react/pull/21687), [#23207](https://github.com/facebook/react/pull/23207), [#23385](https://github.com/facebook/react/pull/23385) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), [@gaearon](https://github.com/gaearon), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), [@trueadm](https://github.com/trueadm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione hidratação seletiva. ([#14717](https://github.com/facebook/react/pull/14717), [#14884](https://github.com/facebook/react/pull/14884), [#16725](https://github.com/facebook/react/pull/16725), [#16880](https://github.com/facebook/react/pull/16880), [#17004](https://github.com/facebook/react/pull/17004), [#22416](https://github.com/facebook/react/pull/22416), [#22629](https://github.com/facebook/react/pull/22629), [#22448](https://github.com/facebook/react/pull/22448), [#22856](https://github.com/facebook/react/pull/22856), [#23176](https://github.com/facebook/react/pull/23176) por [@acdlite](https://github.com/acdlite), [@gaearon](https://github.com/gaearon), [@salazarm](https://github.com/salazarm) e [@sebmarkbage](https://github.com/sebmarkbage))
* Adicione `aria-description` à lista de atributos ARIA conhecidos. ([#22142](https://github.com/facebook/react/pull/22142) por [@mahyareb](https://github.com/mahyareb))
* Adicione evento `onResize` a elementos de vídeo. ([#21973](https://github.com/facebook/react/pull/21973) por [@rileyjshaw](https://github.com/rileyjshaw))
* Adicione `imageSizes` e `imageSrcSet` a props conhecidos. ([#22550](https://github.com/facebook/react/pull/22550) por [@eps1lon](https://github.com/eps1lon))
* Permita filhos `<option>` não-string se `value` for fornecido. ([#21431](https://github.com/facebook/react/pull/21431) por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrija o estilo `aspectRatio` não sendo aplicado. ([#21100](https://github.com/facebook/react/pull/21100) por [@gaearon](https://github.com/gaearon))
* Avise se `renderSubtreeIntoContainer` for chamado. ([#23355](https://github.com/facebook/react/pull/23355) por [@acdlite](https://github.com/acdlite))

### React DOM Server {/*react-dom-server-1*/}

* Adicione o novo renderizador de streaming. ([#14144](https://github.com/facebook/react/pull/14144), [#20970](https://github.com/facebook/react/pull/20970), [#21056](https://github.com/facebook/react/pull/21056), [#21255](https://github.com/facebook/react/pull/21255), [#21200](https://github.com/facebook/react/pull/21200), [#21257](https://github.com/facebook/react/pull/21257), [#21276](https://github.com/facebook/react/pull/21276), [#22443](https://github.com/facebook/react/pull/22443), [#22450](https://github.com/facebook/react/pull/22450), [#23247](https://github.com/facebook/react/pull/23247), [#24025](https://github.com/facebook/react/pull/24025), [#24030](https://github.com/facebook/react/pull/24030) por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrija provedores de contexto em SSR ao lidar com múltiplos pedidos. ([#23171](https://github.com/facebook/react/pull/23171) por [@frandiox](https://github.com/frandiox))
* Reverta para a renderização do cliente em caso de discrepância de texto. ([#23354](https://github.com/facebook/react/pull/23354) por [@acdlite](https://github.com/acdlite))
* Descontinue `renderToNodeStream`. ([#23359](https://github.com/facebook/react/pull/23359) por [@sebmarkbage](https://github.com/sebmarkbage))
* Corrija um registro de erro espúrio no novo renderizador de servidor. ([#24043](https://github.com/facebook/react/pull/24043) por [@eps1lon](https://github.com/eps1lon))
* Corrija um bug no novo renderizador de servidor. ([#22617](https://github.com/facebook/react/pull/22617) por [@shuding](https://github.com/shuding))
* Ignore valores de função e símbolo dentro de elementos personalizados no servidor. ([#21157](https://github.com/facebook/react/pull/21157) por [@sebmarkbage](https://github.com/sebmarkbage))

### React DOM Test Utils {/*react-dom-test-utils*/}

* Lançar quando `act` é usado em produção. ([#21686](https://github.com/facebook/react/pull/21686) por [@acdlite](https://github.com/acdlite))
* Suporte para desativar avisos espúrios de ato com `global.IS_REACT_ACT_ENVIRONMENT`. ([#22561](https://github.com/facebook/react/pull/22561) por [@acdlite](https://github.com/acdlite))
* Expanda o aviso de ato para cobrir todas as APIs que podem agendar trabalho do React. ([#22607](https://github.com/facebook/react/pull/22607) por [@acdlite](https://github.com/acdlite))
* Faça `act` agrupar atualizações. ([#21797](https://github.com/facebook/react/pull/21797) por [@acdlite](https://github.com/acdlite))
* Remova o aviso para efeitos passivos soltos. ([#22609](https://github.com/facebook/react/pull/22609) por [@acdlite](https://github.com/acdlite))

### React Refresh {/*react-refresh*/}

* Rastreie raízes montadas tardiamente no Fast Refresh. ([#22740](https://github.com/facebook/react/pull/22740) por [@anc95](https://github.com/anc95))
* Adicione o campo `exports` ao `package.json`. ([#23087](https://github.com/facebook/react/pull/23087) por [@otakustay](https://github.com/otakustay))

### Componentes do Servidor (Experimental) {/*server-components-experimental*/}

* Adicione suporte a Contexto do Servidor. ([#23244](https://github.com/facebook/react/pull/23244) por [@salazarm](https://github.com/salazarm))
* Adicione suporte a `lazy`. ([#24068](https://github.com/facebook/react/pull/24068) por [@gnoff](https://github.com/gnoff))
* Atualize o plugin webpack para webpack 5 ([#22739](https://github.com/facebook/react/pull/22739) por [@michenly](https://github.com/michenly))
* Corrija um erro no carregador Node. ([#22537](https://github.com/facebook/react/pull/22537) por [@btea](https://github.com/btea))
* Use `globalThis` em vez de `window` para ambientes de borda. ([#22777](https://github.com/facebook/react/pull/22777) por [@huozhi](https://github.com/huozhi))