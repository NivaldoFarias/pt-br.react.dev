---
title: "React Labs: No que estamos trabalhando – Junho de 2022"
author:  Andrew Clark, Dan Abramov, Jan Kassens, Joseph Savona, Josh Story, Lauren Tan, Luna Ruan, Mengdi Chen, Rick Hanlon, Robert Zhang, Sathya Gunasekaran, Sebastian Markbage, and Xuan Huang
date: 2022/06/15
description: O React 18 levou anos para ser produzido e, com ele, trouxe lições valiosas para a equipe do React. Seu lançamento foi o resultado de muitos anos de pesquisa e da exploração de muitos caminhos. Alguns desses caminhos foram bem-sucedidos; muitos outros foram becos sem saída que levaram a novos insights. Uma lição que aprendemos é que é frustrante para a comunidade esperar por novos recursos sem ter uma visão desses caminhos que estamos explorando.
---

15 de junho de 2022 por [Andrew Clark](https://twitter.com/acdlite), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Jan Kassens](https://twitter.com/kassens), [Joseph Savona](https://twitter.com/en_JS), [Josh Story](https://twitter.com/joshcstory), [Lauren Tan](https://twitter.com/potetotes), [Luna Ruan](https://twitter.com/lunaruan), [Mengdi Chen](https://twitter.com/mengdi_en), [Rick Hanlon](https://twitter.com/rickhanlonii), [Robert Zhang](https://twitter.com/jiaxuanzhang01), [Sathya Gunasekaran](https://twitter.com/_gsathya), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Xuan Huang](https://twitter.com/Huxpro)

---

<Intro>

[React 18](/blog/2022/03/29/react-v18) levou anos para ser produzido e, com ele, trouxe lições valiosas para a equipe React. Seu lançamento foi o resultado de muitos anos de pesquisa e da exploração de muitos caminhos. Alguns desses caminhos foram bem-sucedidos; muitos outros foram becos sem saída que levaram a novos insights. Uma lição que aprendemos é que é frustrante para a comunidade esperar por novos recursos sem ter uma visão desses caminhos que estamos explorando.

</Intro>

---

Normalmente, temos vários projetos sendo trabalhados a qualquer momento, variando dos mais experimentais aos claramente definidos. Olhando para o futuro, gostaríamos de começar a compartilhar regularmente mais sobre o que temos trabalhado com a comunidade em todos esses projetos.

Para definir as expectativas, esta não é uma planilha com cronogramas claros. Muitos desses projetos estão sob pesquisa ativa e são difíceis de colocar datas de lançamento concretas. Eles podem possivelmente nunca serem lançados em sua iteração atual, dependendo do que aprendermos. Em vez disso, queremos compartilhar com você os espaços de problema em que estamos pensando ativamente e o que aprendemos até agora.

## Server Components {/*server-components*/}

Anunciamos uma [demo experimental de React Server Components](https://legacy.reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) (RSC) em dezembro de 2020. Desde então, estamos terminando suas dependências no React 18 e trabalhando em mudanças inspiradas pelo feedback experimental.

Em particular, estamos abandonando a ideia de ter bibliotecas de I/O bifurcadas (ex: react-fetch) e, em vez disso, adotando um modelo async/await para melhor compatibilidade. Isso tecnicamente não bloqueia o lançamento do RSC porque você também pode usar roteadores para buscar dados. Outra mudança é que também estamos nos afastando da abordagem de extensão de arquivo em favor da [anotação de limites](https://github.com/reactjs/rfcs/pull/189#issuecomment-1116482278).

Estamos trabalhando em conjunto com a Vercel e a Shopify para unificar o suporte ao bundler para semântica compartilhada em Webpack e Vite. Antes do lançamento, queremos garantir que a semântica dos RSCs seja a mesma em todo o ecossistema React. Este é o principal impeditivo para alcançar a estabilidade.

## Asset Loading {/*asset-loading*/}

Atualmente, ativos como scripts, estilos externos, fontes e imagens são normalmente pré-carregados e carregados usando sistemas externos. Isso pode dificultar a coordenação em novos ambientes como streaming, Server Components e muito mais.
Estamos analisando a possibilidade de adicionar APIs para pré-carregar e carregar ativos externos desduplicados por meio de APIs React que funcionam em todos os ambientes React.

Também estamos analisando a possibilidade de que estes suportem Suspense para que você possa ter imagens, CSS e fontes que bloqueiam a exibição até que sejam carregadas, mas não bloqueiam o streaming e a renderização concorrente. Isso pode ajudar a evitar [“popcorning“](https://twitter.com/sebmarkbage/status/1516852731251724293), pois os visuais surgem e a disposição muda.

## Static Server Rendering Optimizations {/*static-server-rendering-optimizations*/}

Static Site Generation (SSG) e Incremental Static Regeneration (ISR) são ótimas maneiras de obter desempenho para páginas armazenáveis em cache, mas achamos que podemos adicionar recursos para melhorar o desempenho do Server Side Rendering (SSR) dinâmico – especialmente quando a maior parte do conteúdo é armazenável em cache, mas não todo. Estamos explorando maneiras de otimizar a renderização do servidor, utilizando compilação e passes estáticos.

## React Optimizing Compiler {/*react-compiler*/}

Demos uma [prévia antecipada](https://www.youtube.com/watch?v=lGEMwh32soc) do React Forget na React Conf 2021. É um compilador que gera automaticamente o equivalente a chamadas `useMemo` e `useCallback` para minimizar o custo da re-renderização, mantendo o modelo de programação do React.

Recentemente, terminamos uma reescrita do compilador para torná-lo mais confiável e capaz. Essa nova arquitetura nos permite analisar e memorizar padrões mais complexos, como o uso de [mutações locais](/learn/keeping-components-pure#local-mutation-your-components-little-secret), e abre muitas novas oportunidades de otimização em tempo de compilação, além de apenas ser compatível com os Hooks de memorização.

Também estamos trabalhando em um playground para explorar muitos aspectos do compilador. Embora o objetivo do playground seja facilitar o desenvolvimento do compilador, achamos que ele facilitará a experimentação e a construção de intuições sobre o que o compilador faz. Ele revela vários insights sobre como ele funciona por baixo dos panos e renderiza ao vivo as saídas do compilador conforme você digita. Isso será lançado junto com o compilador quando for lançado.

## Offscreen {/*offscreen*/}

Hoje, se você quiser ocultar e exibir um componente, você tem duas opções. Uma é adicioná-lo ou removê-lo completamente da árvore. O problema com esta abordagem é que o estado da sua UI é perdido cada vez que você desmonta, incluindo o estado armazenado no DOM, como a posição da rolagem.

A outra opção é manter o componente montado e alternar a aparência visualmente usando CSS. Isso preserva o estado da sua UI, mas tem um custo de desempenho, porque o React deve continuar renderizando o componente oculto e todos os seus filhos sempre que recebe novas atualizações.

Offscreen apresenta uma terceira opção: oculte a UI visualmente, mas despriorize seu conteúdo. A ideia é semelhante em espírito à propriedade CSS `content-visibility`: quando o conteúdo é oculto, ele não precisa ficar em sincronia com o restante da UI. O React pode adiar o trabalho de renderização até que o restante do aplicativo esteja ocioso ou até que o conteúdo se torne visível novamente.

Offscreen é uma capacidade de baixo nível que desbloqueia recursos de alto nível. Semelhante a outros recursos concorrentes do React, como `startTransition`, na maioria dos casos você não interagirá com a API Offscreen diretamente, mas sim por meio de um framework com opinião para implementar padrões como:

*   **Transições instantâneas.** Alguns frameworks de roteamento já buscam dados antecipadamente para acelerar as navegações subsequentes, como ao passar o mouse sobre um link. Com o Offscreen, eles também poderão pré-renderizar a próxima tela em segundo plano.
*   **Estado reutilizável.** Da mesma forma, ao navegar entre rotas ou guias, você pode usar o Offscreen para preservar o estado da tela anterior para que você possa voltar e continuar de onde parou.
*   **Renderização de lista virtualizada.** Ao exibir grandes listas de itens, os frameworks de lista virtualizada pré-renderizarão mais linhas do que estão atualmente visíveis. Você pode usar o Offscreen para pré-renderizar as linhas ocultas com uma prioridade menor do que os itens visíveis na lista.
*   **Conteúdo em segundo plano.** Também estamos explorando um recurso relacionado para despriorizar o conteúdo em segundo plano sem ocultá-lo, como ao exibir uma sobreposição modal.

## Transition Tracing {/*transition-tracing*/}

Atualmente, o React tem duas ferramentas de perfilamento. O [Profiler original](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) mostra uma visão geral de todos os commits em uma sessão de perfilamento. Para cada commit, ele também mostra todos os componentes que renderizaram e a quantidade de tempo que levou para eles renderizarem. Também temos uma versão beta de um [Timeline Profiler](https://github.com/reactwg/react-18/discussions/76) introduzida no React 18 que mostra quando os componentes agendam atualizações e quando o React trabalha nessas atualizações. Ambos os profilers ajudam os desenvolvedores a identificar problemas de desempenho em seu código.

Percebemos que os desenvolvedores não acham tão útil saber sobre commits ou componentes lentos individuais fora de contexto. É mais útil saber sobre o que realmente causa os commits lentos. E que os desenvolvedores querem ser capazes de rastrear interações específicas (por exemplo, um clique de botão, um carregamento inicial ou uma navegação de página) para observar regressões de desempenho e entender por que uma interação foi lenta e como corrigi-la.

Tentamos solucionar esse problema anteriormente criando uma [API de Rastreamento de Interação](https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16), mas ela tinha algumas falhas de design fundamentais que reduziam a precisão do rastreamento do porquê uma interação era lenta e, às vezes, resultavam em interações que nunca terminavam. Acabamos [removendo esta API](https://github.com/facebook/react/pull/20037) por causa desses problemas.

Estamos trabalhando em uma nova versão para a API de Rastreamento de Interação (provisoriamente chamada de Rastreamento de Transição porque é iniciada via `startTransition`) que resolve esses problemas.

## New React Docs {/*new-react-docs*/}

No ano passado, anunciamos a versão beta do novo site de documentação do React ([posteriormente lançado como react.dev](/blog/2023/03/16/introducing-react-dev)) do novo site de documentação do React. Os novos materiais de aprendizado ensinam Hooks em primeiro lugar e têm novos diagramas, ilustrações, bem como muitos exemplos e desafios interativos. Fizemos uma pausa nesse trabalho para nos concentrarmos no lançamento do React 18, mas agora que o React 18 foi lançado, estamos trabalhando ativamente para finalizar e lançar a nova documentação.

Atualmente, estamos escrevendo uma seção detalhada sobre efeitos, pois ouvimos dizer que esse é um dos tópicos mais desafiadores para usuários novos e experientes do React. [Sincronizando com Efeitos](/learn/synchronizing-with-effects) é a primeira página publicada na série, e haverá mais nas próximas semanas. Quando começamos a escrever uma seção detalhada sobre efeitos, percebemos que muitos padrões de efeitos comuns podem ser simplificados adicionando uma nova primitiva ao React. Compartilhamos algumas ideias iniciais sobre isso no [useEvent RFC](https://github.com/reactjs/rfcs/pull/220). Ele está atualmente em pesquisa inicial e ainda estamos iterando sobre a ideia. Agradecemos os comentários da comunidade sobre o RFC até agora, bem como o [feedback](https://github.com/reactjs/react.dev/issues/3308) e as contribuições para a reformulação da documentação em andamento. Gostaríamos de agradecer especificamente a [Harish Kumar](https://github.com/harish-sethuraman) por enviar e revisar muitas melhorias para a nova implementação do site.

*Obrigado a [Sophie Alpert](https://twitter.com/sophiebits) por revisar esta publicação do blog!*