---
title: "React Labs: No que temos trabalhado – Março de 2023"
author: Joseph Savona, Josh Story, Lauren Tan, Mengdi Chen, Samuel Susla, Sathya Gunasekaran, Sebastian Markbage, e Andrew Clark
date: 2023/03/22
description: Nas postagens do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativo. Fizemos progressos significativos desde nossa última atualização e gostaríamos de compartilhar o que aprendemos.
---

22 de março de 2023 por [Joseph Savona](https://twitter.com/en_JS), [Josh Story](https://twitter.com/joshcstory), [Lauren Tan](https://twitter.com/potetotes), [Mengdi Chen](https://twitter.com/mengdi_en), [Samuel Susla](https://twitter.com/SamuelSusla), [Sathya Gunasekaran](https://twitter.com/_gsathya), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Nas postagens do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativo. Fizemos progressos significativos desde nossa [última atualização](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022) e gostaríamos de compartilhar o que aprendemos.

</Intro>

---

## React Server Components {/*react-server-components*/}

React Server Components (ou RSC) é uma nova arquitetura de aplicação projetada pela equipe do React.

Primeiramente, compartilhamos nossa pesquisa sobre RSC em uma [palestra introdutória](/blog/2020/12/21/data-fetching-with-react-server-components) e um [RFC](https://github.com/reactjs/rfcs/pull/188). Para recapitular, estamos introduzindo um novo tipo de componente—Componentes do Servidor—que são executados antecipadamente e são excluídos do seu pacote JavaScript. Componentes do Servidor podem ser executados durante a construção, permitindo que você leia do sistema de arquivos ou busque conteúdo estático. Eles também podem ser executados no servidor, permitindo que você acesse sua camada de dados sem precisar construir uma API. Você pode passar dados por props dos Componentes do Servidor para os Componentes Interativos do Cliente no navegador.

RSC combina o simples modelo mental de "solicitação/resposta" de Aplicativos Multi-Página centrados no servidor com a interatividade sem costura de Aplicativos de Página Única centrados no cliente, proporcionando o melhor dos dois mundos.

Desde nossa última atualização, nós mesclamos o [RFC dos Componentes do Servidor do React](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md) para ratificar a proposta. Resolvemos questões pendentes com a proposta das [Convenções de Módulos do Servidor do React](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md) e chegamos a um consenso com nossos parceiros para adotar a convenção `"use client"`. Esses documentos também atuam como especificação para o que uma implementação compatível com RSC deve suportar.

A maior mudança é que introduzimos [`async` / `await`](https://github.com/reactjs/rfcs/pull/229) como a principal forma de fazer a busca de dados a partir dos Componentes do Servidor. Também planejamos suportar a carga de dados do cliente ao introduzir um novo Hook chamado `use` que desembrulha Promises. Embora não possamos suportar `async / await` em componentes arbitrários em aplicativos apenas do cliente, planejamos adicionar suporte a isso quando você estruturar seu aplicativo apenas do cliente de forma semelhante à estrutura dos aplicativos RSC.

Agora que temos a busca de dados bem resolvida, estamos explorando a outra direção: enviar dados do cliente para o servidor, para que você possa executar mutações de banco de dados e implementar formulários. Estamos fazendo isso permitindo que você passe funções de Ação do Servidor através da fronteira servidor/cliente, que o cliente pode então chamar, proporcionando RPC sem costura. Ações do Servidor também oferecem formulários aprimorados progressivamente antes que o JavaScript seja carregado.

Os Componentes do Servidor do React foram lançados no [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router). Isso demonstra uma profunda integração de um roteador que realmente incorpora RSC como um primitivo, mas não é a única maneira de construir um roteador e framework compatível com RSC. Há uma separação clara para recursos fornecidos pela especificação RSC e pela implementação. Os Componentes do Servidor do React são destinados como uma especificação para componentes que funcionam em frameworks compatíveis com React.

Recomendamos geralmente o uso de um framework existente, mas se você precisar construir seu próprio framework personalizado, isso é possível. Construir seu próprio framework compatível com RSC não é tão fácil quanto gostaríamos que fosse, principalmente devido à profunda integração necessária com o bundler. A geração atual de bundlers é ótima para uso no cliente, mas não foi projetada com suporte de primeira classe para dividir um único gráfico de módulos entre o servidor e o cliente. É por isso que agora estamos colaborando diretamente com os desenvolvedores de bundlers para obter os primitivos para RSC embutidos.

## Carregamento de Recursos {/*asset-loading*/}

[A Suspense](/reference/react/Suspense) permite que você especifique o que exibir na tela enquanto os dados ou o código para seus componentes ainda estão sendo carregados. Isso permite que seus usuários vejam progressivamente mais conteúdo enquanto a página está sendo carregada, assim como durante as navegações do roteador que carregam mais dados e código. No entanto, da perspectiva do usuário, o carregamento de dados e a renderização não contam toda a história ao considerar se o novo conteúdo está pronto. Por padrão, os navegadores carregam folhas de estilo, fontes e imagens de forma independente, o que pode levar a saltos de UI e alterações de layout consecutivas.

Estamos trabalhando para integrar completamente a Suspense ao ciclo de carregamento de folhas de estilo, fontes e imagens, para que o React as leve em conta para determinar se o conteúdo está pronto para ser exibido. Sem qualquer mudança na forma como você cria seus componentes React, as atualizações se comportarão de maneira mais coerente e agradável. Como uma otimização, também forneceremos uma maneira manual de pré-carregar recursos, como fontes, diretamente dos componentes.

Estamos atualmente implementando esses recursos e teremos mais para compartilhar em breve.

## Metadados do Documento {/*document-metadata*/}

Diferentes páginas e telas em seu aplicativo podem ter metadados diferentes, como a tag `<title>`, descrição e outras tags `<meta>` específicas para essa tela. Do ponto de vista da manutenção, é mais escalável manter essas informações próximas ao componente React daquela página ou tela. No entanto, as tags HTML para esses metadados precisam estar no `<head>` do documento, que é tipicamente renderizado em um componente na raiz do seu aplicativo.

Hoje, as pessoas resolvem esse problema com uma das duas técnicas.

Uma técnica é renderizar um componente especial de terceiros que move `<title>`, `<meta>` e outras tags dentro dele para o `<head>` do documento. Isso funciona para os principais navegadores, mas há muitos clientes que não executam JavaScript do lado do cliente, como analisadores Open Graph, e assim essa técnica não é universalmente adequada.

Outra técnica é renderizar a página no servidor em duas partes. Primeiro, o conteúdo principal é renderizado e todas essas tags são coletadas. Em seguida, o `<head>` é renderizado com essas tags. Finalmente, o `<head>` e o conteúdo principal são enviados ao navegador. Essa abordagem funciona, mas impede que você aproveite o [React 18's Streaming Server Renderer](/reference/react-dom/server/renderToReadableStream) porque você teria que esperar que todo o conteúdo fosse renderizado antes de enviar o `<head>`.

É por isso que estamos adicionando suporte embutido para renderizar as tags `<title>`, `<meta>` e de metadados `<link>` em qualquer lugar da sua árvore de componentes, de forma automática. Isso funcionará da mesma forma em todos os ambientes, incluindo código totalmente do lado do cliente, SSR e, no futuro, RSC. Compartilharemos mais detalhes sobre isso em breve.

## Compilador Otimizador do React {/*react-optimizing-compiler*/}

Desde a nossa atualização anterior, estivemos ativamente iterando no design do [React Forget](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#react-compiler), um compilador otimizador para o React. Já falamos anteriormente sobre ele como um "compilador de memoização automática", e isso é verdade em certo sentido. Mas construir o compilador nos ajudou a entender ainda mais profundamente o modelo de programação do React. Uma maneira melhor de entender o React Forget é como um compilador automático de *reatividade*.

A ideia central do React é que os desenvolvedores definem sua UI como uma função do estado atual. Você trabalha com valores JavaScript comuns — números, strings, arrays, objetos — e usa idiossincrasias JavaScript padrão — if/else, for, etc — para descrever a lógica do seu componente. O modelo mental é que o React será re-renderizado sempre que o estado da aplicação mudar. Acreditamos que esse modelo mental simples e a proximidade com a semântica do JavaScript é um princípio importante no modelo de programação do React.

O problema é que o React pode ser *demais* reativo: ele pode re-renderizar demais. Por exemplo, em JavaScript não temos maneiras baratas de comparar se dois objetos ou arrays são equivalentes (tendo as mesmas chaves e valores), então criar um novo objeto ou array em cada renderização pode fazer com que o React faça mais trabalho do que realmente precisa. Isso significa que os desenvolvedores têm que memoizar explicitamente componentes para não reagir excessivamente a mudanças.

Nosso objetivo com o React Forget é garantir que os aplicativos React tenham a quantidade certa de reatividade por padrão: que os aplicativos re-renderizem apenas quando os valores do estado *mudam significativamente*. Do ponto de vista da implementação, isso significa memoização automática, mas acreditamos que a estrutura de reatividade é uma maneira melhor de entender o React e o Forget. Uma maneira de pensar sobre isso é que o React atualmente re-renderiza quando a identidade do objeto muda. Com o Forget, o React re-renderiza quando o valor semântico muda — mas sem incorrer no custo em tempo de execução de comparações profundas.

Em termos de progresso concreto, desde nossa última atualização, substancialmente iteramos no design do compilador para alinhar com essa abordagem automática de reatividade e incorporar feedback do uso do compilador internamente. Após algumas refatorações significativas do compilador a partir do final do ano passado, agora começamos a usar o compilador em produção em áreas limitadas na Meta. Planejamos open-source uma vez que tenhamos comprovado isso em produção.

Finalmente, muitas pessoas expressaram interesse em como o compilador funciona. Estamos ansiosos para compartilhar muitos mais detalhes quando provarmos o compilador e o tornarmos open-source. Mas há alguns pontos que podemos compartilhar agora:

O núcleo do compilador está quase completamente desacoplado do Babel, e a API central do compilador é (aproximadamente) AST antigo de entrada, novo AST de saída (mantendo os dados de localização de origem). Internamente, usamos uma representação e pipeline de transformação de código personalizados para fazer a análise semântica de baixo nível. No entanto, a principal interface pública para o compilador será via Babel e outros plugins de sistema de construção. Para facilitar os testes, atualmente temos um plugin Babel que é um envoltório muito fino que chama o compilador para gerar uma nova versão de cada função e trocá-la.

À medida que refatoramos o compilador nos últimos meses, queríamos nos concentrar em refinar o modelo de compilação central para garantir que pudéssemos lidar com complexidades como condicionais, loops, reatribuições e mutações. No entanto, o JavaScript possui muitas maneiras de expressar cada um desses recursos: if/else, ternários, for, for-in, for-of, etc. Tentar suportar a linguagem completa desde o início teria atrasado o ponto em que poderíamos validar o modelo central. Em vez disso, começamos com um subconjunto pequeno, mas representativo, da linguagem: let/const, if/else, loops for, objetos, arrays, primitivos, chamadas de função e alguns outros recursos. À medida que ganhamos confiança no modelo central e refinamos nossas abstrações internas, expandimos o subconjunto da linguagem suportado. Também somos explícitos sobre a sintaxe que ainda não suportamos, registrando diagnósticos e pulando a compilação para entradas não suportadas. Temos utilitários para testar o compilador em bases de código da Meta e ver quais recursos não suportados são mais comuns para que possamos priorizar esses próximos. Continuaremos expandindo incrementamente em direção ao suporte para toda a linguagem.

Tornar o JavaScript comum em componentes React reativo requer um compilador com uma compreensão profunda da semântica, para que ele possa entender exatamente o que o código está fazendo. Ao adotar essa abordagem, estamos criando um sistema de reatividade dentro do JavaScript que permite escrever código de produção de qualquer complexidade com toda a expressividade da linguagem, em vez de ser limitado a uma linguagem específica de domínio.

## Renderização Offscreen {/*offscreen-rendering*/}

A renderização offscreen é uma capacidade em desenvolvimento no React para renderizar telas em segundo plano sem sobrecarga adicional de desempenho. Você pode pensar nisso como uma versão da propriedade CSS [`content-visibility`](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) que funciona não apenas para elementos DOM, mas também para componentes React. Durante nossa pesquisa, descobrimos uma variedade de casos de uso:

- Um roteador pode pré-renderizar telas em segundo plano para que, quando um usuário navega até elas, estejam instantaneamente disponíveis.
- Um componente de troca de guias pode preservar o estado das guias ocultas, para que o usuário possa alternar entre elas sem perder seu progresso.
- Um componente de lista virtualizada pode pré-renderizar linhas adicionais acima e abaixo da janela visível.
- Ao abrir um modal ou popup, o resto do aplicativo pode ser colocado em modo "fundo" para que eventos e atualizações sejam desativados para tudo, exceto o modal.

A maioria dos desenvolvedores React não interagirá diretamente com as APIs offscreen do React. Em vez disso, a renderização offscreen será integrada em coisas como roteadores e bibliotecas de UI, e então os desenvolvedores que usam essas bibliotecas automaticamente se beneficiarão sem trabalho adicional.

A ideia é que você deve ser capaz de renderizar qualquer árvore React offscreen sem mudar a forma como você escreve seus componentes. Quando um componente é renderizado offscreen, ele não *monta* realmente até que o componente se torne visível — seus efeitos não são disparados. Por exemplo, se um componente usa `useEffect` para registrar análises quando aparece pela primeira vez, a pré-renderização não afetará a precisão dessas análises. Da mesma forma, quando um componente sai de offscreen, seus efeitos são desmontados também. Um recurso chave da renderização offscreen é que você pode alternar a visibilidade de um componente sem perder seu estado.

Desde nossa última atualização, testamos uma versão experimental de pré-renderização internamente na Meta em nossos aplicativos React Native no Android e iOS, com resultados de desempenho positivos. Também melhoramos como a renderização offscreen funciona com a Suspense — suspender dentro de uma árvore offscreen não acionará os fallback da Suspense. Nosso trabalho restante envolve finalizar os primitivos que são expostos aos desenvolvedores de bibliotecas. Esperamos publicar um RFC ainda este ano, juntamente com uma API experimental para testes e feedback.

## Rastreio de Transições {/*transition-tracing*/}

A API de Rastreio de Transições permite detectar quando [Transições do React](/reference/react/useTransition) se tornam mais lentas e investigar por que elas podem estar lentas. Após nossa última atualização, completamos o design inicial da API e publicamos um [RFC](https://github.com/reactjs/rfcs/pull/238). As capacidades básicas também foram implementadas. O projeto está atualmente pausado. Agradecemos feedback sobre o RFC e esperamos retomar seu desenvolvimento para fornecer uma melhor ferramenta de medição de desempenho para o React. Isso será particularmente útil com roteadores construídos sobre as Transições do React, como o [Next.js App Router](/learn/start-a-new-react-project#nextjs-app-router).

* * *
Além desta atualização, nossa equipe fez recentes aparições como convidados em podcasts e transmissões ao vivo da comunidade para falar mais sobre nosso trabalho e responder perguntas.

* [Dan Abramov](https://twitter.com/dan_abramov) e [Joe Savona](https://twitter.com/en_JS) foram entrevistados por [Kent C. Dodds em seu canal do YouTube](https://www.youtube.com/watch?v=h7tur48JSaw), onde discutiram preocupações em torno dos Componentes do Servidor do React.
* [Dan Abramov](https://twitter.com/dan_abramov) e [Joe Savona](https://twitter.com/en_JS) foram convidados no [podcast JSParty](https://jsparty.fm/267) e compartilharam seus pensamentos sobre o futuro do React.

Agradecemos a [Andrew Clark](https://twitter.com/acdlite), [Dan Abramov](https://twitter.com/dan_abramov), [Dave McCabe](https://twitter.com/mcc_abe), [Luna Wei](https://twitter.com/lunaleaps), [Matt Carroll](https://twitter.com/mattcarrollcode), [Sean Keegan](https://twitter.com/DevRelSean), [Sebastian Silbermann](https://twitter.com/sebsilbermann), [Seth Webster](https://twitter.com/sethwebster), e [Sophie Alpert](https://twitter.com/sophiebits) pela revisão desta postagem.

Obrigado por ler, e nos vemos na próxima atualização!