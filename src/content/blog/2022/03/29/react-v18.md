React v18.0

29 de março de 2022 por [A Equipe React](/community/team)

---

<Intro>

O React 18 já está disponível no npm! Em nossa última postagem, compartilhámos instruções passo a passo para [atualizar seu aplicativo para o React 18](/blog/2022/03/08/react-18-upgrade-guide). Nesta postagem, apresentaremos uma visão geral do que há de novo no React 18 e o que isso significa para o futuro.

</Intro>

---

Nossa versão principal mais recente inclui melhorias prontas para uso, como "batching" automático, novas APIs como startTransition e renderização do lado do servidor com suporte para Suspense.

Muitos dos recursos do React 18 são construídos em cima do nosso novo renderizador concorrente, uma mudança nos bastidores que desbloqueia novos recursos poderosos. O React Concorrente é opcional — ele só é ativado quando você usa um recurso concorrente — mas achamos que isso terá um grande impacto na forma como as pessoas constroem aplicativos.

Passamos anos pesquisando e desenvolvendo suporte para concorrência no React, e tomamos cuidado extra para fornecer um caminho de adoção gradual para os usuários existentes. No verão passado, [formamos o React 18 Working Group](/blog/2021/06/08/the-plan-for-react-18) para reunir feedback de especialistas na comunidade e garantir uma experiência de atualização tranquila para todo o ecossistema React.

Caso você não tenha visto, compartilhamos muita dessa visão no React Conf 2021:

*  Na [palestra principal](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa), explicamos como o React 18 se encaixa em nossa missão de facilitar para os desenvolvedores a criação de ótimas experiências de usuário
*  [Shruti Kapoor](https://twitter.com/shrutikapoor08) [demonstrou como usar os novos recursos do React 18](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
*  [Shaundai Person](https://twitter.com/shaundai) nos deu uma visão geral da [renderização do lado do servidor com Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

A seguir, uma visão geral completa do que esperar nesta versão, começando com a renderização concorrente.

<Note>

Para usuários do React Native, o React 18 será lançado no React Native com a Nova Arquitetura React Native. Para mais informações, consulte a [palestra principal do React Conf aqui](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s).

</Note>

## O que é React Concorrente? {/*what-is-concurrent-react*/}

A adição mais importante no React 18 é algo que esperamos que você nunca precise pensar: concorrência. Achamos que isso é amplamente verdade para desenvolvedores de aplicativos, embora a história possa ser um pouco mais complicada para os mantenedores de bibliotecas.

A concorrência não é uma feature, *per se*. É um novo mecanismo nos bastidores que permite que o React prepare várias versões da sua UI ao mesmo tempo. Você pode pensar na concorrência como um detalhe de implementação — ela é valiosa por causa dos recursos que ela desbloqueia. O React usa técnicas sofisticadas em sua implementação interna, como filas prioritárias e múltiplos *buffering*. Mas você não verá esses conceitos em lugar nenhum em nossas APIs públicas.

Quando projetamos APIs, tentamos esconder os detalhes de implementação dos desenvolvedores. Como desenvolvedor React, você se concentra em *o que* quer que a experiência do usuário pareça, e o React lida com *como* entregar essa experiência. Portanto, não esperamos que os desenvolvedores React saibam como a concorrência funciona por baixo dos panos.

No entanto, o React Concorrente é mais importante do que um detalhe de implementação típico — é uma atualização essencial para o modelo de renderização central do React. Portanto, embora não seja super importante saber como a concorrência funciona, pode valer a pena saber o que ela é em alto nível.

Uma propriedade chave do React Concorrente é que a renderização é interrompível. Quando você atualiza pela primeira vez para o React 18, antes de adicionar quaisquer recursos concorrentes, as atualizações são renderizadas da mesma forma que nas versões anteriores do React — em uma única transação síncrona e ininterrupta. Com a renderização síncrona, assim que uma atualização começa a renderizar, nada pode interrompê-la até que o usuário possa ver o resultado na tela.

Em uma renderização concorrente, este não é sempre o caso. O React pode começar a renderizar uma atualização, pausar no meio e, em seguida, continuar mais tarde. Pode até abandonar uma renderização em andamento completamente. O React garante que a UI parecerá consistente mesmo se uma renderização for interrompida. Para fazer isso, ele espera para realizar mutações DOM até o final, uma vez que toda a árvore tenha sido avaliada. Com essa capacidade, o React pode preparar novas telas em segundo plano sem bloquear a thread principal. Isso significa que a UI pode responder imediatamente à entrada do usuário, mesmo que esteja no meio de uma grande tarefa de renderização, criando uma experiência de usuário fluida.

Outro exemplo é o estado reutilizável. O React Concorrente pode remover seções da UI da tela e, em seguida, adicioná-las de volta mais tarde, reutilizando o estado anterior. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de restaurar a tela anterior no mesmo estado em que estava antes. Em uma próxima versão secundária, estamos planejando adicionar um novo componente chamado `<Offscreen>` que implementa esse padrão. Da mesma forma, você poderá usar o Offscreen para preparar nova UI em segundo plano para que ela esteja pronta antes que o usuário a revele.

A renderização concorrente é uma nova e poderosa ferramenta no React e a maioria de nossos novos recursos são construídos para tirar proveito dela, incluindo Suspense, transições e renderização do servidor por streaming. Mas o React 18 é apenas o começo do que pretendemos construir sobre essa nova base.

## Adotando gradualmente recursos concorrentes {/*gradually-adopting-concurrent-features*/}

Tecnicamente, a renderização concorrente é uma **breaking change**. Como a renderização concorrente é interrompível, os componentes se comportam de maneira ligeiramente diferente quando ela está ativada.

Em nossos testes, atualizamos milhares de componentes para o React 18. O que descobrimos é que quase todos os componentes existentes "simplesmente funcionam" com a renderização concorrente, sem nenhuma alteração. No entanto, alguns deles podem exigir algum esforço de migração adicional. Embora as alterações geralmente sejam pequenas, você ainda terá a capacidade de fazê-las no seu próprio ritmo. O novo comportamento de renderização no React 18 **só é ativado nas partes do seu aplicativo que usam novos recursos.**

A estratégia geral de atualização é fazer com que seu aplicativo seja executado no React 18 sem quebrar o código existente. Então você pode gradualmente começar a adicionar recursos concorrentes no seu próprio ritmo. Você pode usar [`<StrictMode>`](/reference/react/StrictMode) para ajudar a mostrar erros relacionados à concorrência durante o desenvolvimento. O Modo Estrito não afeta o comportamento de produção, mas durante o desenvolvimento ele registrará avisos extras e invocará duas vezes as funções que devem ser idempotentes. Ele não pegará tudo, mas é eficaz para evitar os tipos mais comuns de erros.

Depois de atualizar para o React 18, você poderá começar a usar os recursos concorrentes imediatamente. Por exemplo, você pode usar startTransition para navegar entre as telas sem bloquear a entrada do usuário. Ou use useDeferredValue para limitar as re-renderizações caras.

No entanto, a longo prazo, esperamos que a principal forma de adicionar concorrência ao seu aplicativo seja usando uma biblioteca ou framework habilitado para concorrência. Na maioria dos casos, você não interagirá com APIs concorrentes diretamente. Por exemplo, em vez de os desenvolvedores chamarem startTransition sempre que navegam para uma nova tela, as bibliotecas de roteamento irão automaticamente encapsular as navegações em startTransition.

Pode levar algum tempo para que as bibliotecas sejam atualizadas para serem compatíveis com a concorrência. Fornecemos novas APIs para facilitar que as bibliotecas aproveitem os recursos concorrentes. Nesse meio tempo, seja paciente com os mantenedores, pois trabalhamos para migrar gradualmente todo o ecossistema React.

Para mais informações, consulte nossa postagem anterior: [Como atualizar para o React 18](/blog/2022/03/08/react-18-upgrade-guide).

## Suspense em Frameworks de Dados {/*suspense-in-data-frameworks*/}

No React 18, você pode começar a usar [Suspense](/reference/react/Suspense) para busca de dados em *frameworks* com opiniões como Relay, Next.js, Hydrogen ou Remix. A busca de dados *ad hoc* com Suspense é tecnicamente possível, mas ainda não é recomendada como uma estratégia geral.

No futuro, podemos expor *primitives* adicionais que podem facilitar o acesso aos seus dados com Suspense, talvez sem o uso de um *framework* com opiniões. No entanto, o Suspense funciona melhor quando está profundamente integrado à arquitetura do seu aplicativo: seu roteador, sua camada de dados e seu ambiente de renderização do servidor. Portanto, mesmo a longo prazo, esperamos que as bibliotecas e *frameworks* desempenhem um papel crucial no ecossistema React.

Como nas versões anteriores do React, você também pode usar o Suspense para *code splitting* no cliente com React.lazy. Mas nossa visão para o Suspense sempre foi sobre muito mais do que apenas carregar código — o objetivo é estender o suporte para o Suspense para que, eventualmente, o mesmo *fallback* declarativo do Suspense possa lidar com qualquer operação assíncrona (carregamento de código, dados, imagens, etc).

## Server Components ainda está em desenvolvimento {/*server-components-is-still-in-development*/}

[**Server Components**](/blog/2020/12/21/data-fetching-with-react-server-components) é um recurso futuro que permite que os desenvolvedores construam aplicativos que abrangem o servidor e o cliente, combinando a rica interatividade de aplicativos do lado do cliente com o desempenho aprimorado da renderização tradicional do servidor. Server Components não é inerentemente acoplado ao React Concorrente, mas foi projetado para funcionar melhor com recursos concorrentes como Suspense e renderização do servidor por *streaming*.

Server Components ainda é experimental, mas esperamos lançar uma versão inicial em uma versão secundária 18.x. Enquanto isso, estamos trabalhando com *frameworks* como Next.js, Hydrogen e Remix para avançar na proposta e prepará-la para uma ampla adoção.

## O que há de novo no React 18 {/*whats-new-in-react-18*/}

### Recurso novo: *Batching* automático {/*new-feature-automatic-batching*/}

*Batching* acontece quando o React agrupa várias atualizações de estado em uma única re-renderização para obter um melhor desempenho. Sem o *batching* automático, apenas fazíamos o *batching* de atualizações dentro dos *event handlers* do React. As atualizações dentro de *promises*, `setTimeout`, *event handlers* nativos ou qualquer outro evento não foram agrupadas no React por padrão. Com o *batching* automático, essas atualizações serão agrupadas automaticamente:

```js
// Before: only React events were batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will render twice, once for each state update (no batching)
}, 1000);

// After: updates inside of timeouts, promises,
// native event handlers or any other event are batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

Para mais informações, veja esta postagem [Batching automático para menos renderizações no React 18](https://github.com/reactwg/react-18/discussions/21).

### Recurso novo: Transições {/*new-feature-transitions*/}

Uma transição é um novo conceito no React para distinguir entre atualizações urgentes e não urgentes.

*   **Atualizações urgentes** refletem a interação direta, como digitar, clicar, pressionar e assim por diante.
*   **Atualizações de transição** transitam a UI de uma visualização para outra.

Atualizações urgentes, como digitar, clicar ou pressionar, precisam de resposta imediata para corresponder às nossas intuições sobre como os objetos físicos se comportam. Caso contrário, elas parecem "erradas". No entanto, as transições são diferentes porque o usuário não espera ver todo valor intermediário na tela.

Por exemplo, quando você seleciona um filtro em uma lista suspensa, espera que o botão de filtro em si responda imediatamente quando você clica. No entanto, os resultados reais podem fazer a transição separadamente. Um pequeno atraso seria imperceptível e geralmente esperado. E se você alterar o filtro novamente antes que os resultados sejam renderizados, você só se importa em ver os resultados mais recentes.

Normalmente, para a melhor experiência do usuário, uma única entrada do usuário deve resultar em uma atualização urgente e não urgente. Você pode usar a API startTransition dentro de um evento de entrada para informar ao React quais atualizações são urgentes e quais são "transições":

```js
import { startTransition } from 'react';

// Urgent: Show what was typed
setInputValue(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setSearchQuery(input);
});
```

As atualizações encapsuladas em startTransition são tratadas como não urgentes e serão interrompidas se atualizações mais urgentes, como cliques ou pressionamentos de teclas, entrarem. Se uma transição for interrompida pelo usuário (por exemplo, digitando vários caracteres seguidos), o React descartará o trabalho de renderização obsoleto que não foi concluído e renderizará apenas a atualização mais recente.

*   `useTransition`: um Hook para iniciar transições, incluindo um valor para rastrear o estado pendente.
*   `startTransition`: um método para iniciar transições quando o Hook não pode ser usado.

As transições serão integradas à renderização concorrente, o que permite interromper a atualização. Se o conteúdo for suspenso novamente, as transições também informam ao React para continuar mostrando o conteúdo atual enquanto renderiza o conteúdo da transição em segundo plano (consulte o [Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md) para maiores informações).

[Veja a documentação para transições aqui](/reference/react/useTransition).

### Novas funcionalidades do Suspense {/*new-suspense-features*/}

O Suspense permite que você especifique declarativamente o estado de carregamento para uma parte da árvore de componentes, caso ela ainda não esteja pronta para ser exibida:

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

O Suspense torna o "estado de carregamento da UI" um conceito declarativo de primeira classe no modelo de programação React. Isso nos permite criar recursos de nível superior a partir dele.

Apresentamos uma versão limitada do Suspense há vários anos. No entanto, o único caso de uso suportado era o *code splitting* com React.lazy, e ele não era suportado de forma alguma ao renderizar no servidor.

No React 18, adicionamos suporte para Suspense no servidor e expandimos seus recursos usando recursos de renderização concorrente.

O Suspense no React 18 funciona melhor quando combinado com a API de transição. Se você suspender durante uma transição, o React impedirá que o conteúdo já visível seja substituído por um *fallback*. Em vez disso, o React atrasará a renderização até que dados suficientes sejam carregados para evitar um estado de carregamento ruim.

Para mais informações, veja o RFC para [Suspense no React 18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md).

### Novas APIs de renderização do cliente e do servidor {/*new-client-and-server-rendering-apis*/}

Nesta versão, aproveitamos a oportunidade para redesenhar as APIs que expomos para renderização no cliente e no servidor. Essas alterações permitem que os usuários continuem usando as APIs antigas no modo React 17 enquanto atualizam para as novas APIs no React 18.

#### React DOM Cliente {/*react-dom-client*/}

Essas novas APIs agora são exportadas de `react-dom/client`:

*   `createRoot`: Novo método para criar uma raiz para `renderizar` ou `desmontar`. Use-o em vez de `ReactDOM.render`. Os novos recursos do React 18 não funcionam sem ele.
*   `hydrateRoot`: Novo método para hidratar um aplicativo renderizado no servidor. Use-o em vez de `ReactDOM.hydrate` em conjunto com as novas APIs React DOM Server. Os novos recursos do React 18 não funcionam sem ele.

`createRoot` e `hydrateRoot` aceitam uma nova opção chamada `onRecoverableError` caso você queira ser notificado quando o React se recuperar de erros durante a renderização ou hidratação para registro. Por padrão, o React usará [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError) ou `console.error` em navegadores mais antigos.

[Veja a documentação para React DOM Client aqui](/reference/react-dom/client).

#### React DOM Server {/*react-dom-server-1*/}

Essas novas APIs agora são exportadas de `react-dom/server` e têm suporte total à renderização do Suspense por *streaming* no servidor:

*   `renderToPipeableStream`: para *streaming* em ambientes Node.
*   `renderToReadableStream`: para ambientes de tempo de execução de ponta modernos, como Deno e *workers* do Cloudflare.

O método `renderToString` existente continua funcionando, mas é desencorajado.

[Veja a documentação para React DOM Server aqui](/reference/react-dom/server).

### Novos comportamentos do Modo Estrito {/*new-strict-mode-behaviors*/}

No futuro, gostaríamos de adicionar um recurso que permita ao React adicionar e remover seções da UI enquanto preserva o estado. Por exemplo, quando um usuário sai de uma tela e volta, o React deve ser capaz de mostrar imediatamente a tela anterior. Para fazer isso, o React desmontaria e remontaria árvores usando o mesmo estado do componente de antes.

Esse recurso dará aos aplicativos React um melhor desempenho pronto para uso, mas exige que os componentes sejam resilientes a *effects* sendo montados e destruídos várias vezes. A maioria dos *effects* funcionará sem nenhuma alteração, mas alguns *effects* supõem que eles só são montados ou destruídos uma vez.

Para ajudar a mostrar esses problemas, o React 18 apresenta uma nova verificação apenas para desenvolvimento no Modo Estrito. Essa nova verificação desmontará e remontará automaticamente cada componente, sempre que um componente for montado pela primeira vez, restaurando o estado anterior na segunda montagem.

Antes dessa mudança, o React montava o componente e criava os *effects*:

```
* React mounts the component.
  * Layout effects are created.
  * Effects are created.
```

Com o Modo Estrito no React 18, o React simulará a desmontagem e remontagem do componente no modo de desenvolvimento:

```
* React mounts the component.
  * Layout effects are created.
  * Effects are created.
* React simulates unmounting the component.
  * Layout effects are destroyed.
  * Effects are destroyed.
* React simulates mounting the component with the previous state.
  * Layout effects are created.
  * Effects are created.
```

[Veja a documentação para garantir o estado reutilizável aqui](/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development).

### Novos Hooks {/*new-hooks*/}

#### useId {/*useid*/}

`useId` é um novo Hook para gerar IDs exclusivos no cliente e no servidor, evitando incompatibilidades de hidratação. Ele é principalmente útil para bibliotecas de componentes que se integram a APIs de acessibilidade que exigem IDs exclusivos. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 por causa de como o novo renderizador do servidor de streaming entrega HTML fora de ordem. [Veja a documentação aqui](/reference/react/useId).

> Nota
>
> `useId` **não** serve para gerar [chaves em uma lista](/learn/rendering-lists#where-to-get-your-key). As chaves devem ser geradas a partir de seus dados.

#### useTransition {/*usetransition*/}

`useTransition` e `startTransition` permitem que você marque algumas atualizações de estado como não urgentes. Outras atualizações de estado são consideradas urgentes por padrão. O React permitirá que as atualizações de estado urgentes (por exemplo, atualizar uma entrada de texto) interrompam as atualizações de estado não urgentes (por exemplo, renderizar uma lista de resultados da pesquisa). [Veja a documentação aqui](/reference/react/useTransition).

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` permite que você adie a renderização de uma parte não urgente da árvore. É semelhante ao debouncing, mas tem algumas vantagens em comparação. Não há atraso de tempo fixo, então o React tentará a renderização adiada logo após a refletida na tela. A renderização adiada é interrompível e não bloqueia a entrada do usuário. [Veja a documentação aqui](/reference/react/useDeferredValue).

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` é um novo Hook que permite que lojas externas suportem leituras simultâneas, forçando atualizações para a loja a serem síncronas. Ele remove a necessidade de useEffect ao implementar assinaturas em fontes de dados externas e é recomendado para qualquer biblioteca que se integre ao estado externo ao React. [Veja a documentação aqui](/reference/react/useSyncExternalStore).

> Nota
>
> `useSyncExternalStore` destina-se a ser usado por bibliotecas, não por código de aplicativo.

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` é um novo Hook que permite que bibliotecas CSS-in-JS abordem problemas de desempenho de injeção de estilos na renderização. A menos que você já tenha construído uma biblioteca CSS-in-JS, não esperamos que você use isso. Este Hook será executado depois que o DOM for mutado, mas antes que os *layout effects* leiam o novo *layout*. Isso resolve um problema que já existe no React 17 e versões anteriores, mas é ainda mais importante no React 18 porque o React cede ao navegador durante a renderização concorrente, dando a ele a chance de recalcular o *layout*. [Veja a documentação aqui](/reference/react/useInsertionEffect).

> Nota
>
> `useInsertionEffect` destina-se a ser usado por bibliotecas, não por código de aplicativo.

## Como atualizar {/*how-to-upgrade*/}

Consulte [Como atualizar para o React 18](/blog/2022/03/08/react-18-upgrade-guide) para obter instruções passo a passo e uma lista completa de alterações *breaking* e notáveis.

## Changelog {/*changelog*/}

### React {/*react*/}

*   Adiciona `useTransition` e `useDeferredValue` para separar atualizações urgentes de transições. ([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii) e [@sebmarkbage](https://github.com/sebmarkbage))
*   Adiciona `useId` para gerar IDs exclusivos. ([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) por [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan) e [@sebmarkbage](https://github.com/sebmarkbage))
*   Adiciona `useSyncExternalStore` para ajudar as bibliotecas de armazenamento externas a se integrarem ao React. ([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@drarmstr](https://github.com/drarmstr))
*   Adiciona `startTransition` como uma versão de `useTransition` sem comentários pendentes. ([#19696](https://github.com/facebook/react/pull/19696)  por [@rickhanlonii](https://github.com/rickhanlonii))
*   Adiciona `useInsertionEffect` para bibliotecas CSS-in-JS. ([#21913](https://github.com/facebook/react/pull/21913)  por [@rickhanlonii](https://github.com/rickhanlonii))
*   Faz com que o Suspense remonte os *layout effects* quando o conteúdo reaparece.  ([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) por [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
*   Faz com que `<StrictMode>` re-execute os *effects* para verificar o estado restaurável. ([#19523](https://github.com/facebook/react/pull/19523) , [#21418](https://github.com/facebook/react/pull/21418)  por [@bvaughn](https://github.com/bvaughn) e [@lunaruan](https://github.com/lunaruan))
*   Assume que os Símbolos estão sempre disponíveis. ([#23348](https://github.com/facebook/react/pull/23348)  por [@sebmarkbage](https://github.com/sebmarkbage))
*   Remove o polyfill `object-assign`. ([#23351](https://github.com/facebook/react/pull/23351)  por [@sebmarkbage](https://github.com/sebmarkbage))
*   Remove a API `unstable_changedBits` sem suporte.  ([#20953](https://github.com/facebook/react/pull/20953)  por [@acdlite](https://github.com/acdlite))
*   Permite que os componentes renderizem `undefined`. ([#21869](https://github.com/facebook/react/pull/21869)  por [@rickhanlonii](https://github.com/rickhanlonii))
*   *Flush* `useEffect` resultando de eventos discretos como cliques de forma síncrona. ([#21150](https://github.com/facebook/react/pull/21150)  por [@acdlite](https://github.com/acdlite))
*   Suspense `fallback={undefined}` agora se comporta da mesma forma que `null` e não é ignorado. ([#21854](https://github.com/facebook/react/pull/21854)  por [@rickhanlonii](https://github.com/rickhanlonii))
*   Considera todos os `lazy()` resolvendo o mesmo componente como equivalente. ([#20357](https://github.com/facebook/react/pull/20357)  por [@sebmarkbage](https://github.com/sebmarkbage))
*   Não corrige o console durante a primeira renderização. ([#22308](https://github.com/facebook/react/pull/22308)  por [@lunaruan](https://github.com/lunaruan))
*   Melhora o uso de memória. ([#21039](https://github.com/facebook/react/pull/21039)  por [@bgirard](https://github.com/bgirard))
*   Melhora as mensagens se a coerção de *string* lança (Temporal.*, Symbol, etc.) ([#22064](https://github.com/facebook/react/pull/22064)  por [@justingrant](https://github.com/justingrant))
*   Use `setImmediate` quando disponível, usando o `MessageChannel`. ([#20834](https://github.com/facebook/react/pull/20834)  por [@gaearon](https://github.com/gaearon))
*   Corrige a propagação de contexto dentro de árvores suspensas. ([#23095](https://github.com/facebook/react/pull/23095)  por [@gaearon](https://github.com/gaearon))
*   Corrige `useReducer` observando *props* incorretas removendo o mecanismo de *bailout* *eager*. ([#22445](https://github.com/facebook/react/pull/22445)  por [@josephsavona](https://github.com/josephsavona))
*   Corrige o `setState` sendo ignorado no Safari ao anexar *iframes*. ([#23111](https://github.com/facebook/react/pull/23111)  por [@gaearon](https://github.com/gaearon))
*   Corrige a queda ao renderizar `ZonedDateTime` na árvore. ([#20617](https://github.com/facebook/react/pull/20617)  por [@dimaqq](https://github.com/dimaqq))
*   Corrige uma queda quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695)  por [@SimenB](https://github.com/SimenB))
*   Corrige `onLoad` não sendo acionado quando os recursos concorrentes estão ativados. ([#23316](https://github.com/facebook/react/pull/23316)  por [@gnoff](https://github.com/gnoff))
*   Corrige um aviso quando um seletor retorna `NaN`.  ([#23333](https://github.com/facebook/react/pull/23333)  por [@hachibeeDI](https://github.com/hachibeeDI))
*   Corrige uma queda quando o documento é definido como `null` em testes. ([#22695](https://github.com/facebook/react/pull/22695) por [@SimenB](https://github.com/SimenB))
*   Corrige o cabeçalho de licença gerado. ([#23004](https://github.com/facebook/react/pull/23004)  por [@vitaliemiron](https://github.com/vitaliemiron))
*   Adiciona `package.json` como um dos *entry points*. ([#22954](https://github.com/facebook/react/pull/22954)  por [@Jack](https://github.com/Jack-Works))
*   Permite suspender fora de uma *boundary* Suspense. ([#23267](https://github.com/facebook/react/pull/23267)  por [@acdlite](https://github.com/acdlite))
*   Registra um erro recuperável sempre que a hidratação falha. ([#23319](https://github.com/facebook/react/pull/23319)  por [@acdlite](https://github.com/acdlite))

### React DOM {/*react-dom*/}

*   Adiciona `createRoot` e `hydrateRoot`. ([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/