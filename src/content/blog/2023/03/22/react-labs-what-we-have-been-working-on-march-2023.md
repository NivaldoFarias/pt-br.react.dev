---
title: "React Labs: No que estivemos trabalhando – Março de 2023"
author: Joseph Savona, Josh Story, Lauren Tan, Mengdi Chen, Samuel Susla, Sathya Gunasekaran, Sebastian Markbage, and Andrew Clark
date: 2023/03/22
description: Em posts do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos neles desde nossa última atualização e gostaríamos de compartilhar o que aprendemos.
---

22 de março de 2023 por [Joseph Savona](https://twitter.com/en_JS), [Josh Story](https://twitter.com/joshcstory), [Lauren Tan](https://twitter.com/potetotes), [Mengdi Chen](https://twitter.com/mengdi_en), [Samuel Susla](https://twitter.com/SamuelSusla), [Sathya Gunasekaran](https://twitter.com/_gsathya), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Em posts do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos neles desde nossa [última atualização](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022) e gostaríamos de compartilhar o que aprendemos.

</Intro>

---

## Componentes de Servidor React {/*react-server-components*/}

Componentes de Servidor React (ou RSC) é uma nova arquitetura de aplicação projetada pelo time do React.

Nós compartilhamos pela primeira vez nossa pesquisa sobre RSC em uma [palestra introdutória](/blog/2020/12/21/data-fetching-with-react-server-components) e uma [RFC](https://github.com/reactjs/rfcs/pull/188). Para recapitular, estamos introduzindo um novo tipo de componente - Componentes de Servidor - que são executados antes do tempo e são excluídos do seu *bundle* JavaScript. Componentes de Servidor podem executar durante a construção, permitindo que você leia do sistema de arquivos ou busque conteúdo estático. Eles também podem executar no servidor, permitindo que você acesse sua camada de dados sem ter que construir uma API. Você pode passar dados por *props* de Componentes de Servidor para os Componentes de Cliente interativos no navegador.

RSC combina o modelo mental simples de "requisição/resposta" de Aplicações Multi-Página centradas no servidor com a interatividade contínua de Aplicações de Página Única centradas no cliente, oferecendo o melhor dos dois mundos.

Desde nossa última atualização, nós mergeamos a [RFC de Componentes de Servidor React](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md) para ratificar a proposta. Nós resolvemos problemas pendentes com a proposta de [Convenções de Módulo de Servidor React](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md) e chegamos a um consenso com nossos parceiros para seguir com a convenção `"use client"`. Esses documentos também atuam como especificação para o que uma implementação compatível com RSC deve suportar.

A maior mudança é que nós introduzimos [`async` / `await`](https://github.com/reactjs/rfcs/pull/229) como a principal maneira de fazer busca de dados de Componentes de Servidor. Nós também planejamos suportar o carregamento de dados do cliente, introduzindo um novo Hook chamado `use` que desembrulha *Promises*. Embora não possamos suportar `async / await` em componentes arbitrários em aplicações somente cliente, nós planejamos adicionar suporte para isso quando você estruturar sua aplicação somente cliente de forma similar a como as aplicações RSC são estruturadas.

Agora que temos a busca de dados muito bem resolvida, estamos explorando a outra direção: enviar dados do cliente para o servidor, para que você possa executar mutações no banco de dados e implementar formulários. Estamos fazendo isso permitindo que você passe funções de Ação de Servidor através da fronteira servidor/cliente, que o cliente pode então chamar, fornecendo RPC contínuo. Ações de Servidor também fornecem a você formulários aprimorados progressivamente antes que o JavaScript carregue.

Componentes de Servidor React foram enviados em [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router). Isso mostra uma integração profunda de um roteador que realmente compra a ideia de RSC como uma primitiva, mas não é a única maneira de construir um roteador e *framework* compatível com RSC. Há uma separação clara para os recursos fornecidos pela especificação e implementação do RSC. Componentes de Servidor React é destinado a ser uma especificação para componentes que funcionam em *frameworks* React compatíveis.

Nós geralmente recomendamos usar um *framework* existente, mas se você precisar construir seu próprio *framework* customizado, é possível. Construir seu próprio *framework* compatível com RSC não é tão fácil quanto gostaríamos que fosse, principalmente devido à profunda integração do *bundler* necessária. A geração atual de *bundlers* é ótima para uso no cliente, mas eles não foram projetados com suporte de primeira classe para dividir um único grafo de módulo entre o servidor e o cliente. É por isso que agora estamos fazendo parcerias diretamente com desenvolvedores de *bundler* para obter as primitivas para RSC embutidas.

## Carregamento de Recursos (Assets) {/*asset-loading*/}

[Suspense](/reference/react/Suspense) permite que você especifique o que exibir na tela enquanto os dados ou código para seus componentes ainda estão sendo carregados. Isso permite que seus usuários vejam progressivamente mais conteúdo enquanto a página está carregando, bem como durante as navegações do roteador que carregam mais dados e código. No entanto, da perspectiva do usuário, o carregamento e a *renderização* de dados não contam toda a história ao considerar se o novo conteúdo está pronto. Por padrão, os navegadores carregam folhas de estilo, fontes e imagens independentemente, o que pode levar a saltos de UI e mudanças de *layout* consecutivas.

Estamos trabalhando para integrar totalmente o Suspense com o ciclo de vida de carregamento de folhas de estilo, fontes e imagens, para que o React os leve em consideração para determinar se o conteúdo está pronto para ser exibido. Sem qualquer alteração na maneira como você cria seus componentes React, as atualizações se comportarão de uma maneira mais coerente e agradável. Como uma otimização, também forneceremos uma maneira manual de pré-carregar recursos como fontes diretamente dos componentes.

Atualmente, estamos implementando esses recursos e teremos mais para compartilhar em breve.

## Metadados do Documento {/*document-metadata*/}

Diferentes páginas e telas em seu aplicativo podem ter diferentes metadados, como a tag `<title>`, descrição e outras tags `<meta>` específicas para esta tela. Da perspectiva da manutenção, é mais escalável manter essas informações próximas ao componente React para essa página ou tela. No entanto, as *tags* HTML para esses metadados precisam estar no `<head>` do documento, que normalmente é *renderizado* em um componente na raiz do seu aplicativo.

Hoje, as pessoas resolvem esse problema com uma das duas técnicas.

Uma técnica é *renderizar* um componente especial de terceiros que move `<title>`, `<meta>` e outras *tags* dentro dele para o `<head>` do documento. Isso funciona para os principais navegadores, mas existem muitos clientes que não executam JavaScript do lado do cliente, como *parsers* do Open Graph, e, portanto, essa técnica não é universalmente adequada.

Outra técnica é *renderizar* a página no servidor em duas partes. Primeiro, o conteúdo principal é *renderizado* e todas as *tags* são coletadas. Então, o `<head>` é *renderizado* com essas *tags*. Finalmente, o `<head>` e o conteúdo principal são enviados ao navegador. Essa abordagem funciona, mas impede que você aproveite o [Streaming Server Renderer do React 18](/reference/react-dom/server/renderToReadableStream) porque você teria que esperar que todo o conteúdo fosse *renderizado* antes de enviar o `<head>`.

É por isso que estamos adicionando suporte interno para *renderizar* *tags* `<title>`, `<meta>` e `<link>` de metadados em qualquer lugar em sua árvore de componentes *out of the box*. Ele funcionaria da mesma forma em todos os ambientes, incluindo código totalmente do lado do cliente, SSR e, no futuro, RSC. Compartilharemos mais detalhes sobre isso em breve.

## Otimizando o Compilador React {/*react-optimizing-compiler*/}

Desde nossa atualização anterior, temos iterado ativamente no design do [React Forget](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#react-compiler), um compilador de otimização para React. Já falamos sobre ele como um "compilador de auto-memorização", e isso é verdade em certo sentido. Mas construir o compilador nos ajudou a entender o modelo de programação do React ainda mais profundamente. Uma maneira melhor de entender o React Forget é como um compilador de *reatividade* automático.

A ideia central do React é que os desenvolvedores definam sua UI como uma função do *state* atual. Você trabalha com valores JavaScript simples — números, *strings*, *arrays*, objetos — e usa expressões idiomáticas JavaScript padrão — if/else, for, etc. — para descrever a lógica do seu componente. O modelo mental é que o React irá *renderizar* novamente sempre que o *state* da aplicação mudar. Acreditamos que este modelo mental simples e manter-se próximo à semântica JavaScript é um princípio importante no modelo de programação do React.

O problema é que o React às vezes pode ser *reativo demais*: ele pode *renderizar* novamente demais. Por exemplo, em JavaScript não temos maneiras baratas de comparar se dois objetos ou *arrays* são equivalentes (tendo as mesmas chaves e valores), então criar um novo objeto ou *array* em cada *renderização* pode fazer com que o React faça mais trabalho do que estritamente necessário. Isso significa que os desenvolvedores devem explicitaramente *memoizar* componentes para não reagir em excesso às mudanças.

Nosso objetivo com o React Forget é garantir que as aplicações React tenham a quantidade certa de reatividade por padrão: que as aplicações *renderizem* novamente apenas quando os valores do *state* mudarem *significativamente*. De uma perspectiva de implementação, isso significa *memoizar* automaticamente, mas acreditamos que o enquadramento da reatividade é uma maneira melhor de entender o React e o Forget. Uma maneira de pensar sobre isso é que o React atualmente *renderiza* novamente quando a identidade do objeto muda. Com o Forget, o React *renderiza* novamente quando o valor semântico muda — mas sem incorrer no custo de tempo de execução de comparações profundas.

Em termos de progresso concreto, desde nossa última atualização, iteramos substancialmente no design do compilador para alinhar com esta abordagem de reatividade automática e para incorporar o *feedback* do uso do compilador internamente. Após algumas refatorações significativas do compilador a partir do final do ano passado, agora começamos a usar o compilador em produção em áreas limitadas do Meta. Planejamos torná-lo *open-source* assim que o provarmos em produção.

Finalmente, muitas pessoas expressaram interesse em como o compilador funciona. Estamos ansiosos para compartilhar muito mais detalhes quando provarmos o compilador e o tornarmos *open-source*. Mas há algumas partes que podemos compartilhar agora:

O núcleo do compilador é quase completamente desacoplado do Babel, e a API do compilador central é (grosseiramente) AST antigo de entrada, novo AST de saída (enquanto retém os dados de localização da fonte). Nos bastidores, usamos uma representação de código customizada e *pipeline* de transformação para fazer análise semântica de baixo nível. No entanto, a interface primária pública para o compilador será via Babel e outros *plugins* do sistema de construção. Para facilitar os testes, atualmente temos um *plugin* Babel que é um *wrapper* muito fino que chama o compilador para gerar uma nova versão de cada função e trocá-la.

Enquanto refatorávamos o compilador nos últimos meses, queríamos nos concentrar em refinar o modelo de compilação central para garantir que pudéssemos lidar com complexidades como condicionais, *loops*, reatribuição e mutação. No entanto, JavaScript tem muitas maneiras de expressar cada um desses recursos: if/else, ternários, for, for-in, for-of, etc. Tentar suportar a linguagem completa antecipadamente teria atrasado o ponto em que poderíamos validar o modelo central. Em vez disso, começamos com um subconjunto pequeno, mas representativo, da linguagem: let/const, if/else, *loops* for, objetos, *arrays*, primitivos, chamadas de função e alguns outros recursos. À medida que ganhamos confiança no modelo central e refinamos nossas abstrações internas, expandimos o subconjunto de linguagem suportado. Também somos explícitos sobre a sintaxe que ainda não suportamos, registrando diagnósticos e ignorando a compilação para entrada não suportada. Temos utilitários para experimentar o compilador nas bases de código do Meta e ver quais recursos não suportados são mais comuns para que possamos priorizar os próximos. Continuaremos expandindo incrementalmente para suportar toda a linguagem.

Tornar o JavaScript simples em componentes React reativo requer um compilador com um profundo entendimento da semântica para que ele possa entender exatamente o que o código está fazendo. Ao adotar essa abordagem, estamos criando um sistema para reatividade dentro do JavaScript que permite que você escreva código de produto de qualquer complexidade com toda a expressividade da linguagem, em vez de ser limitado a uma linguagem específica de domínio.

## *Renderização* Fora da Tela {/*offscreen-rendering*/}

*Renderização* fora da tela é uma capacidade futura no React para *renderizar* telas em segundo plano sem sobrecarga de desempenho adicional. Você pode pensar nisso como uma versão da [propriedade CSS `content-visibility`](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) que funciona não apenas para elementos DOM, mas também para componentes React. Durante nossa pesquisa, descobrimos uma variedade de casos de uso:

- Um roteador pode pré-*renderizar* telas em segundo plano para que, quando um usuário navega até elas, elas estejam instantaneamente disponíveis.
- Um componente de troca de aba pode preservar o *state* de abas ocultas, para que o usuário possa alternar entre elas sem perder seu progresso.
- Um componente de lista virtualizada pode pré-*renderizar* linhas adicionais acima e abaixo da janela visível.
- Ao abrir um *modal* ou *popup*, o restante do aplicativo pode ser colocado no modo "segundo plano" para que eventos e atualizações sejam desativados para tudo, exceto o *modal*.

A maioria dos desenvolvedores React não interagirá diretamente com as APIs *offscreen* do React. Em vez disso, a *renderização offscreen* será integrada a coisas como roteadores e bibliotecas de UI, e então os desenvolvedores que usam essas bibliotecas se beneficiarão automaticamente sem trabalho adicional.

A ideia é que você deve ser capaz de *renderizar* qualquer árvore React fora da tela sem mudar a maneira como você escreve seus componentes. Quando um componente é *renderizado* fora da tela, ele na verdade não é *montado* até que o componente se torne visível — seus efeitos não são disparados. Por exemplo, se um componente usa `useEffect` para registrar *analytics* quando ele aparece pela primeira vez, a pré-*renderização* não prejudicará a precisão desses *analytics*. Da mesma forma, quando um componente sai da tela, seus efeitos também são desmontados. Um recurso chave da *renderização offscreen* é que você pode alternar a visibilidade de um componente sem perder seu *state*.

Desde nossa última atualização, testamos uma versão experimental de pré-*renderização* internamente no Meta em nossos aplicativos React Native no Android e iOS, com resultados positivos de desempenho. Também melhoramos como a *renderização offscreen* funciona com Suspense — suspender dentro de uma árvore *offscreen* não acionará *fallbacks* de Suspense. Nosso trabalho restante envolve finalizar as primitivas que são expostas aos desenvolvedores de biblioteca. Esperamos publicar uma RFC ainda este ano, juntamente com uma API experimental para testes e *feedback*.

## Rastreamento de Transição {/*transition-tracing*/}

A API de Rastreamento de Transição permite que você detecte quando as [Transições React](/reference/react/useTransition) ficam mais lentas e investigue por que elas podem estar lentas. Após nossa última atualização, concluímos o design inicial da API e publicamos uma [RFC](https://github.com/reactjs/rfcs/pull/238). Os recursos básicos também foram implementados. O projeto está atualmente em espera. Agradecemos o *feedback* sobre a RFC e esperamos retomar seu desenvolvimento para fornecer uma ferramenta de medição de desempenho melhor para React. Isso será particularmente útil com roteadores construídos em cima de Transições React, como o [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router).

* * *
Além desta atualização, nossa equipe fez aparições recentes como convidados em *podcasts* e *livestreams* da comunidade para falar mais sobre nosso trabalho e responder a perguntas.

* [Dan Abramov](https://bsky.app/profile/danabra.mov) e [Joe Savona](https://twitter.com/en_JS) foram entrevistados por [Kent C. Dodds em seu canal no YouTube](https://www.youtube.com/watch?v=h7tur48JSaw), onde discutiram preocupações sobre Componentes de Servidor React.
* [Dan Abramov](https://bsky.app/profile/danabra.mov) e [Joe Savona](https://twitter.com/en_JS) foram convidados no [*podcast* JSParty](https://jsparty.fm/267) e compartilharam seus pensamentos sobre o futuro do React.

Obrigado a [Andrew Clark](https://twitter.com/acdlite), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Dave McCabe](https://twitter.com/mcc_abe), [Luna Wei](https://twitter.com/lunaleaps), [Matt Carroll](https://twitter.com/mattcarrollcode), [Sean Keegan](https://twitter.com/DevRelSean), [Sebastian Silbermann](https://twitter.com/sebsilbermann), [Seth Webster](https://twitter.com/sethwebster) e [Sophie Alpert](https://twitter.com/sophiebits) por revisar este *post*.

Obrigado por ler e até a próxima atualização!
