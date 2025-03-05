---
title: "React Labs: No que temos trabalhado – Março de 2023"
author: Joseph Savona, Josh Story, Lauren Tan, Mengdi Chen, Samuel Susla, Sathya Gunasekaran, Sebastian Markbage e Andrew Clark
date: 2023/03/22
description: Nas postagens do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos neles desde nossa última atualização, e gostaríamos de compartilhar o que aprendemos.
---

22 de março de 2023 por [Joseph Savona](https://twitter.com/en_JS), [Josh Story](https://twitter.com/joshcstory), [Lauren Tan](https://twitter.com/potetotes), [Mengdi Chen](https://twitter.com/mengdi_en), [Samuel Susla](https://twitter.com/SamuelSusla), [Sathya Gunasekaran](https://twitter.com/_gsathya), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Nas postagens do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos neles desde nossa [última atualização](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022), e gostaríamos de compartilhar o que aprendemos.

</Intro>

---

## React Server Components {/*react-server-components*/}

React Server Components (ou RSC) é uma nova arquitetura de aplicação projetada pela equipe do React.

Compartilhamos nossa pesquisa sobre RSC em uma [apresentação introdutória](/blog/2020/12/21/data-fetching-with-react-server-components) e um [RFC](https://github.com/reactjs/rfcs/pull/188). Para recapitular, estamos introduzindo um novo tipo de componente - Server Components - que são executados com antecedência e são excluídos do seu bundle JavaScript. Server Components podem rodar durante o build, permitindo que você leia do sistema de arquivos ou busque conteúdo estático. Eles também podem rodar no servidor, permitindo que você acesse sua camada de dados sem precisar construir uma API. Você pode passar dados por props de Server Components para os Client Components interativos no navegador.

RSC combina o modelo mental simples de "request/response" de aplicativos de várias páginas (Multi-Page Apps) centrados no servidor com a interatividade perfeita de aplicativos de página única (Single-Page Apps) centrados no cliente, oferecendo o melhor dos dois mundos.

Desde nossa última atualização, mesclamos o [React Server Components RFC](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md) para ratificar a proposta. Resolvemos problemas pendentes com a proposta [React Server Module Conventions](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md) e chegamos a um consenso com nossos parceiros para usar a convenção `"use client"`. Esses documentos também funcionam como especificação para o que uma implementação compatível com RSC deve suportar.

A maior mudança é que introduzimos [`async` / `await`](https://github.com/reactjs/rfcs/pull/229) como a principal maneira de buscar dados de Server Components. Também planejamos dar suporte ao carregamento de dados do cliente, introduzindo um novo Hook chamado `use` que desempacota Promises. Embora não possamos suportar `async / await` em componentes arbitrários em aplicativos somente cliente, planejamos adicionar suporte a ele quando você estruturar seu aplicativo somente cliente de forma semelhante à forma como os aplicativos RSC são estruturados.

Agora que temos a busca de dados bem organizada, estamos explorando na outra direção: enviar dados do cliente para o servidor, para que você possa executar mutações de banco de dados e implementar formulários. Estamos fazendo isso permitindo que você passe funções Server Action através da fronteira servidor/cliente, que o cliente pode então chamar, fornecendo RPC perfeito. Server Actions também oferecem formulários progressivamente aprimorados antes que o JavaScript seja carregado.

React Server Components foi lançado no [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router). Isso mostra uma profunda integração de um roteador que realmente usa o RSC como um primitivo, mas não é a única maneira de construir um roteador e framework compatível com RSC. Há uma clara separação para recursos fornecidos pela especificação e implementação do RSC. React Server Components foi projetado como uma especificação para componentes que funcionam em frameworks React compatíveis.

Geralmente, recomendamos usar um framework existente, mas se você precisar construir seu próprio framework personalizado, é possível. Criar seu próprio framework compatível com RSC não é tão fácil quanto gostaríamos que fosse, principalmente devido à profunda integração do bundler necessária. A geração atual de bundlers é ótima para uso no cliente, mas não foi projetada com suporte de primeira classe para dividir um único gráfico de módulo entre o servidor e o cliente. É por isso que agora estamos em parceria direta com os desenvolvedores de bundler para obter os primitivos para RSC integrados.

## Carregamento de ativos {/*asset-loading*/}

[Suspense](/reference/react/Suspense) permite que você especifique o que exibir na tela enquanto os dados ou o código para seus componentes ainda estão sendo carregados. Isso permite que seus usuários vejam progressivamente mais conteúdo enquanto a página está carregando, bem como durante as navegações do roteador que carregam mais dados e código. No entanto, da perspectiva do usuário, o carregamento e a renderização de dados não contam toda a história ao considerar se o novo conteúdo está pronto. Por padrão, os navegadores carregam folhas de estilo, fontes e imagens de forma independente, o que pode levar a saltos na UI e mudanças consecutivas de layout.

Estamos trabalhando para integrar totalmente o Suspense com o ciclo de vida de carregamento de folhas de estilo, fontes e imagens, para que o React os leve em consideração para determinar se o conteúdo está pronto para ser exibido. Sem nenhuma alteração na maneira como você cria seus componentes React, as atualizações se comportarão de maneira mais coerente e agradável. Como uma otimização, também forneceremos uma maneira manual de pré-carregar ativos como fontes diretamente dos componentes.

Atualmente, estamos implementando esses recursos e teremos mais para compartilhar em breve.

## Metadados do documento {/*document-metadata*/}

Diferentes páginas e telas em seu aplicativo podem ter metadados diferentes, como a tag `<title>`, descrição e outras tags `<meta>` específicas desta tela. Da perspectiva de manutenção, é mais escalonável manter essa informação perto do componente React para essa página ou tela. No entanto, as tags HTML para esses metadados precisam estar no documento `<head>`, que normalmente é renderizado em um componente na raiz do seu aplicativo.

Hoje, as pessoas resolvem esse problema com uma das duas técnicas.

Uma técnica é renderizar um componente especial de terceiros que move `<title>`, `<meta>` e outras tags dentro dele para o documento `<head>`. Isso funciona para os principais navegadores, mas existem muitos clientes que não executam JavaScript do lado do cliente, como analisadores do Open Graph, e, portanto, essa técnica não é universalmente adequada.

Outra técnica é renderizar a página no servidor em duas partes. Primeiro, o conteúdo principal é renderizado e todas essas tags são coletadas. Em seguida, o `<head>` é renderizado com essas tags. Finalmente, o `<head>` e o conteúdo principal são enviados para o navegador. Essa abordagem funciona, mas impede que você aproveite o [Renderizador de Servidor de Streaming do React 18](/reference/react-dom/server/renderToReadableStream) porque você teria que esperar a renderização de todo o conteúdo antes de enviar o `<head>`.

É por isso que estamos adicionando suporte integrado para renderizar tags `<title>`, `<meta>` e metadados `<link>` em qualquer lugar da sua árvore de componentes, de dentro para fora. Funcionaria da mesma forma em todos os ambientes, incluindo código totalmente do lado do cliente, SSR e, no futuro, RSC. Compartilharemos mais detalhes sobre isso em breve.

## React Optimizing Compiler {/*react-optimizing-compiler*/}

Desde nossa atualização anterior, temos iterado ativamente no design do [React Forget](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#react-compiler), um compilador otimizador para React. Falamos anteriormente sobre ele como um "compilador de auto-memoização", e isso é verdade em certo sentido. Mas construir o compilador nos ajudou a entender ainda mais profundamente o modelo de programação do React. Uma maneira melhor de entender o React Forget é como um compilador de *reatividade* automático.

A ideia central do React é que os desenvolvedores definam sua UI como uma função do estado atual. Você trabalha com valores JavaScript simples — números, strings, arrays, objetos — e usa expressões JavaScript padrão — if/else, for, etc — para descrever a lógica do seu componente. O modelo mental é que o React irá re-renderizar sempre que o estado da aplicação mudar. Acreditamos que esse modelo mental simples e a manutenção próxima da semântica do JavaScript são um princípio importante no modelo de programação do React.

O problema é que o React pode, às vezes, ser *demasiado* reativo: ele pode re-renderizar demais. Por exemplo, em JavaScript não temos maneiras baratas de comparar se dois objetos ou arrays são equivalentes (tendo as mesmas chaves e valores), então criar um novo objeto ou array em cada renderização pode fazer com que o React faça mais trabalho do que o estritamente necessário. Isso significa que os desenvolvedores devem memorizar explicitamente os componentes para não reagir demais às mudanças.

Nosso objetivo com o React Forget é garantir que os aplicativos React tenham a quantidade certa de reatividade por padrão: que os aplicativos re-renderizem apenas quando os valores de estado mudarem *significativamente*. De uma perspectiva de implementação, isso significa memoização automática, mas acreditamos que a estrutura de reatividade é uma maneira melhor de entender o React e o Forget. Uma maneira de pensar sobre isso é que o React atualmente re-renderiza quando a identidade do objeto muda. Com o Forget, o React re-renderiza quando o valor semântico muda — mas sem incorrer no custo de tempo de execução de comparações profundas.

Em termos de progresso concreto, desde nossa última atualização, iteramos substancialmente no projeto do compilador para alinhá-lo a essa abordagem de reatividade automática e para incorporar o feedback do uso do compilador internamente. Após algumas refatorações significativas no compilador no final do ano passado, começamos a usar o compilador em produção em áreas limitadas na Meta. Pretendemos torná-lo de código aberto assim que o provarmos em produção.

Finalmente, muitas pessoas expressaram interesse em como o compilador funciona. Estamos ansiosos para compartilhar muitos mais detalhes quando provarmos o compilador e abri-lo ao código-fonte. Mas há algumas coisas que podemos compartilhar agora:

O núcleo do compilador é quase completamente desacoplado do Babel, e a API principal do compilador é (aproximadamente) AST antigo de entrada, AST novo de saída (enquanto retém dados de localização da fonte). Por baixo dos panos, usamos uma representação de código personalizada e um pipeline de transformação para fazer análise semântica de baixo nível. No entanto, a interface pública primária do compilador será por meio de plugins Babel e outros sistemas de build. Para facilitar os testes, atualmente temos um plugin Babel que é um wrapper muito fino que chama o compilador para gerar uma nova versão de cada função e trocá-la.

Ao refatorar o compilador nos últimos meses, queríamos nos concentrar no refinamento do modelo de compilação principal para garantir que pudéssemos lidar com complexidades como condicionais, loops, reatribuição e mutação. No entanto, o JavaScript tem muitas maneiras de expressar cada um desses recursos: if/else, ternários, for, for-in, for-of, etc. Tentar dar suporte à linguagem completa antecipadamente teria atrasado o ponto em que poderíamos validar o modelo principal. Em vez disso, começamos com um subconjunto pequeno, mas representativo, da linguagem: let/const, if/else, loops for, objetos, arrays, primitivos, chamadas de função e alguns outros recursos. À medida que ganhamos confiança no modelo principal e refinamos nossas abstrações internas, expandimos o subconjunto de linguagem suportado. Também somos explícitos sobre a sintaxe que ainda não suportamos, registrando diagnósticos e ignorando a compilação para entrada não suportada. Temos utilitários para experimentar o compilador nos codebases da Meta e ver quais recursos não suportados são mais comuns para que possamos priorizá-los a seguir. Continuaremos a expandir incrementalmente o suporte a toda a linguagem.

Tornar o JavaScript simples em componentes React reativo requer um compilador com um profundo entendimento da semântica para que ele possa entender exatamente o que o código está fazendo. Ao adotar essa abordagem, estamos criando um sistema para reatividade dentro do JavaScript que permite escrever código de produto de qualquer complexidade com a expressividade total da linguagem, em vez de se limitar a uma linguagem específica de domínio.

## Offscreen Rendering {/*offscreen-rendering*/}

Offscreen rendering é um recurso futuro no React para renderizar telas em segundo plano sem sobrecarga de desempenho adicional. Você pode pensar nisso como uma versão da propriedade CSS [`content-visibility`](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) que funciona não apenas para elementos DOM, mas também para componentes React. Durante nossa pesquisa, descobrimos uma variedade de casos de uso:

- Um roteador pode pré-renderizar telas em segundo plano para que, quando um usuário navegar até elas, elas estejam instantaneamente disponíveis.
- Um componente de troca de guias pode preservar o estado de guias ocultas, para que o usuário possa alternar entre elas sem perder seu progresso.
- Um componente de lista virtualizada pode pré-renderizar linhas adicionais acima e abaixo da janela visível.
- Ao abrir um modal ou popup, o restante do aplicativo pode ser colocado no modo "plano de fundo" para que os eventos e atualizações sejam desabilitados para tudo, exceto o modal.

A maioria dos desenvolvedores React não interage diretamente com as APIs offscreen do React. Em vez disso, a renderização offscreen será integrada a coisas como roteadores e bibliotecas de UI, e então os desenvolvedores que usam essas bibliotecas se beneficiarão automaticamente sem trabalho adicional.

A ideia é que você deve ser capaz de renderizar qualquer árvore do React offscreen sem alterar a maneira como você escreve seus componentes. Quando um componente é renderizado offscreen, ele não é realmente *montado* até que o componente se torne visível — seus efeitos não são disparados. Por exemplo, se um componente usar `useEffect` para registrar análises quando aparecer pela primeira vez, a pré-renderização não afetará a precisão dessas análises. Da mesma forma, quando um componente sai da tela, seus efeitos também são desmontados. Um recurso chave da renderização offscreen é que você pode alternar a visibilidade de um componente sem perder seu estado.

Desde nossa última atualização, testamos uma versão experimental de pré-renderização internamente na Meta em nossos aplicativos React Native no Android e iOS, com resultados de desempenho positivos. Também melhoramos como a renderização offscreen funciona com o Suspense - suspender dentro de uma árvore offscreen não acionará os fallbacks do Suspense. Nosso trabalho restante envolve a finalização dos primitivos que são expostos aos desenvolvedores de bibliotecas. Esperamos publicar um RFC ainda este ano, junto com uma API experimental para teste e feedback.

## Transition Tracing {/*transition-tracing*/}

A API de Transition Tracing permite que você detecte quando as [Transições do React](/reference/react/useTransition) ficam mais lentas e investigue o motivo de sua lentidão. Após nossa última atualização, concluímos o projeto inicial da API e publicamos um [RFC](https://github.com/reactjs/rfcs/pull/238). As funcionalidades básicas também foram implementadas. O projeto está atualmente suspenso. Agradecemos o feedback sobre o RFC e esperamos retomar seu desenvolvimento para fornecer uma ferramenta de medição de desempenho melhor para o React. Isso será particularmente útil com roteadores construídos sobre as Transições do React, como o [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router).

* * *
Além desta atualização, nossa equipe fez aparições recentes em podcasts e transmissões ao vivo da comunidade para falar mais sobre nosso trabalho e responder a perguntas.

* [Dan Abramov](https://bsky.app/profile/danabra.mov) e [Joe Savona](https://twitter.com/en_JS) foram entrevistados por [Kent C. Dodds em seu canal do YouTube](https://www.youtube.com/watch?v=h7tur48JSaw), onde discutiram preocupações em relação aos React Server Components.
* [Dan Abramov](https://bsky.app/profile/danabra.mov) e [Joe Savona](https://twitter.com/en_JS) foram convidados no [podcast JSParty](https://jsparty.fm/267) e compartilharam suas opiniões sobre o futuro do React.

Obrigado a [Andrew Clark](https://twitter.com/acdlite), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Dave McCabe](https://twitter.com/mcc_abe), [Luna Wei](https://twitter.com/lunaleaps), [Matt Carroll](https://twitter.com/mattcarrollcode), [Sean Keegan](https://twitter.com/DevRelSean), [Sebastian Silbermann](https://twitter.com/sebsilbermann), [Seth Webster](https://twitter.com/sethwebster) e [Sophie Alpert](https://twitter.com/sophiebits) por revisar esta postagem.

Obrigado por ler e nos vemos na próxima atualização!