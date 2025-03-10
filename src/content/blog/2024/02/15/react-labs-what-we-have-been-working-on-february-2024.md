---
title: "React Labs: No que temos trabalhado – Fevereiro de 2024"
author: Joseph Savona, Ricky Hanlon, Andrew Clark, Matt Carroll e Dan Abramov
date: 2024/02/15
description: Nas publicações do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos desde nossa última atualização e gostaríamos de compartilhar nosso progresso.
---

15 de fevereiro de 2024 por [Joseph Savona](https://twitter.com/en_JS), [Ricky Hanlon](https://twitter.com/rickhanlonii), [Andrew Clark](https://twitter.com/acdlite), [Matt Carroll](https://twitter.com/mattcarrollcode) e [Dan Abramov](https://bsky.app/profile/danabra.mov).

---

<Intro>

Nas publicações do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativos. Fizemos progressos significativos desde nossa [última atualização](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023) e gostaríamos de compartilhar nosso progresso.

</Intro>

<Note>

A React Conf 2024 está agendada para 15 a 16 de maio em Henderson, Nevada! Se você estiver interessado em participar da React Conf pessoalmente, pode [se inscrever na loteria de ingressos](https://forms.reform.app/bLaLeE/react-conf-2024-ticket-lottery/1aRQLK) até 28 de fevereiro.

Para obter mais informações sobre ingressos, streaming gratuito, patrocínio e muito mais, consulte [o site da React Conf](https://conf.react.dev).

</Note>

---

## React Compiler {/*react-compiler*/}

React Compiler não é mais um projeto de pesquisa: o compilador agora impulsiona instagram.com em produção, e estamos trabalhando para lançar o compilador em outras plataformas no Meta e preparar o primeiro lançamento de código aberto.

Como discutido em nossa [postagem anterior](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-optimizing-compiler), o React *às vezes* pode renderizar demais quando o state muda. Desde os primeiros dias do React, nossa solução para esses casos tem sido a memoização manual. Em nossas APIs atuais, isso significa aplicar as APIs [`useMemo`](/reference/react/useMemo), [`useCallback`](/reference/react/useCallback) e [`memo`](/reference/react/memo) para ajustar manualmente o quanto o React renderiza novamente nas mudanças de state. Mas a memoização manual é um compromisso. Ela confunde nosso código, é fácil de errar e exige trabalho extra para mantê-lo atualizado.

A memoização manual é um compromisso razoável, mas não estávamos satisfeitos. Nossa visão é que o React *automaticamente* renderize novamente apenas as partes certas da UI quando o state muda, *sem comprometer o modelo mental principal do React*. Acreditamos que a abordagem do React — UI como uma função simples do state, com valores e idiomas padrão do JavaScript — é uma parte fundamental do motivo pelo qual o React tem sido acessível para tantos desenvolvedores. É por isso que investimos na construção de um compilador otimizador para React.

JavaScript é uma linguagem notoriamente desafiadora de otimizar, graças às suas regras soltas e natureza dinâmica. O React Compiler é capaz de compilar código com segurança, modelando as regras do JavaScript *e* as "regras do React". Por exemplo, os componentes React devem ser idempotentes — retornando o mesmo valor com as mesmas entradas — e não podem mutar props ou valores de state. Essas regras limitam o que os desenvolvedores podem fazer e ajudam a criar um espaço seguro para o compilador otimizar.

Obviamente, entendemos que os desenvolvedores às vezes contornam um pouco as regras, e nosso objetivo é fazer com que o React Compiler funcione imediatamente na maior quantidade possível de código. O compilador tenta detectar quando o código não segue estritamente as regras do React e compilará o código onde for seguro ou pulará a compilação se não for seguro. Estamos testando em relação à grande e variada base de código do Meta para ajudar a validar essa abordagem.

Para os desenvolvedores que estão curiosos sobre como garantir que seu código segue as regras do React, recomendamos [habilitar o Strict Mode](/reference/react/StrictMode) e [configurar o plugin ESLint do React](/learn/editor-setup#linting). Essas ferramentas podem ajudar a detectar bugs sutis no seu código React, melhorando a qualidade de seus aplicativos hoje e tornando seus aplicativos à prova de futuro para recursos futuros, como o React Compiler. Também estamos trabalhando em uma documentação consolidada das regras do React e atualizações para nosso plugin ESLint para ajudar as equipes a entender e aplicar essas regras para criar aplicativos mais robustos.

Para ver o compilador em ação, você pode conferir nossa [palestra do último outono](https://www.youtube.com/watch?v=qOQClO3g8-Y). Na época da palestra, tínhamos dados experimentais iniciais de testar o React Compiler em uma página de instagram.com. Desde então, lançamos o compilador em produção em todo o instagram.com. Também expandimos nossa equipe para acelerar a implantação em outras plataformas no Meta e para código aberto. Estamos entusiasmados com o caminho a seguir e teremos mais para compartilhar nos próximos meses.

## Actions {/*actions*/}

Nós [compartilhamos anteriormente](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components) que estávamos explorando soluções para enviar dados do cliente para o servidor com Server Actions, para que você possa executar mutações de banco de dados e implementar formulários. Durante o desenvolvimento de Server Actions, estendemos essas APIs para suportar o tratamento de dados em aplicativos somente no cliente também.

Referimo-nos a esta coleção mais ampla de recursos simplesmente como "Actions." Actions permitem que você passe uma função para elementos DOM como [`<form/>`](/reference/react-dom/components/form):

```js
<form action={search}>
  <input name="query" />
  <button type="submit">Search</button>
</form>
```

A função `action` pode operar de forma síncrona ou assíncrona. Você pode defini-las no lado do cliente usando JavaScript padrão ou no servidor com a diretiva [`'use server'`](/reference/rsc/use-server). Ao usar uma action, o React gerenciará o ciclo de vida do envio de dados para você, fornecendo hooks como [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) e [`useActionState`](/reference/react/useActionState) para acessar o estado atual e a resposta da form action.

Por padrão, as Actions são enviadas em uma [transição](/reference/react/useTransition), mantendo a página atual interativa enquanto a action está sendo processada. Como as Actions suportam funções assíncronas, também adicionamos a capacidade de usar `async/await` em transições. Isso permite que você mostre a UI pendente com o estado `isPending` de uma transição quando uma solicitação assíncrona como `fetch` é iniciada e mostre a UI pendente até que a atualização seja aplicada.

Junto com as Actions, estamos introduzindo um recurso chamado [`useOptimistic`](/reference/react/useOptimistic) para gerenciar atualizações de estado otimistas. Com este hook, você pode aplicar atualizações temporárias que são revertidas automaticamente assim que o estado final é confirmado. Para as Actions, isso permite que você defina otimisticamente o estado final dos dados no cliente, assumindo que o envio seja bem-sucedido, e reverta para o valor dos dados recebidos do servidor. Ele funciona usando `async`/`await` regular, então funciona da mesma forma se você estiver usando `fetch` no cliente ou uma Server Action do servidor.

Os autores de bibliotecas podem implementar as próprias propriedades `action={fn}` personalizadas em seus componentes com `useTransition`. Nossa intenção é que as bibliotecas adotem o padrão Actions ao projetar suas APIs de componentes, para fornecer uma experiência consistente para os desenvolvedores React. Por exemplo, se sua biblioteca fornecer um componente `<Calendar onSelect={eventHandler}>`, considere também expor uma API `<Calendar selectAction={action}>`.

Embora tenhamos focado inicialmente em Server Actions para transferência de dados cliente-servidor, nossa filosofia para React é fornecer o mesmo modelo de programação em todas as plataformas e ambientes. Quando possível, se apresentarmos um recurso no cliente, nosso objetivo é fazê-lo funcionar também no servidor e vice-versa. Essa filosofia nos permite criar um único conjunto de APIs que funcionam, independentemente de onde seu aplicativo é executado, tornando mais fácil a atualização para diferentes ambientes posteriormente.

As Actions já estão disponíveis no canal Canary e serão lançadas na próxima versão do React.

## Novos recursos no React Canary {/*new-features-in-react-canary*/}

Apresentamos [React Canaries](/blog/2023/05/03/react-canaries) como uma opção para adotar novos recursos estáveis individuais assim que seu design estiver quase finalizado, antes de serem lançados em uma versão semver estável.

Canaries são uma mudança na forma como desenvolvemos o React. Anteriormente, os recursos seriam pesquisados e construídos de forma privada dentro do Meta, de modo que os usuários só veriam o produto final polido quando lançado para Stable. Com Canaries, estamos construindo em público com a ajuda da comunidade para finalizar os recursos que compartilhamos na série de blogs React Labs. Isso significa que você ouve sobre novos recursos mais cedo, à medida que eles estão sendo finalizados, em vez de depois de serem concluídos.

React Server Components, Asset Loading, Document Metadata e Actions chegaram no React Canary, e adicionamos documentação para esses recursos no react.dev:

- **Diretivas**: [`"use client"`](/reference/rsc/use-client) e [`"use server"`](/reference/rsc/use-server) são recursos de bundler projetados para frameworks React full-stack. Elas marcam os "pontos de divisão" entre os dois ambientes: `"use client"` instrui o bundler a gerar uma tag `<script>` (como [Astro Islands](https://docs.astro.build/en/concepts/islands/#creating-an-island)), enquanto `"use server"` diz ao bundler para gerar um endpoint POST (como [tRPC Mutations](https://trpc.io/docs/concepts)). Juntas, elas permitem que você escreva componentes reutilizáveis que compõem a interatividade do lado do cliente com a lógica relacionada do lado do servidor.

- **Document Metadata**: adicionamos suporte integrado para renderizar tags [`<title>`](/reference/react-dom/components/title), [`<meta>`](/reference/react-dom/components/meta) e metadados [`<link>`](/reference/react-dom/components/link) em qualquer lugar em sua árvore de componentes. Elas funcionam da mesma forma em todos os ambientes, incluindo código totalmente do lado do cliente, SSR e RSC. Isso fornece suporte integrado para recursos pioneiros de bibliotecas como [React Helmet](https://github.com/nfl/react-helmet).

- **Asset Loading**: integramos Suspense com o ciclo de vida de carregamento de recursos, como folhas de estilo, fontes e scripts, para que o React os leve em consideração para determinar se o conteúdo em elementos como [`<style>`](/reference/react-dom/components/style), [`<link>`](/reference/react-dom/components/link) e [`<script>`](/reference/react-dom/components/script) estão prontos para serem exibidos. Também adicionamos novas [APIs de Carregamento de Recursos](/reference/react-dom#resource-preloading-apis), como `preload` e `preinit`, para fornecer maior controle sobre quando um recurso deve ser carregado e inicializado.

- **Actions**: Como compartilhado acima, adicionamos Actions para gerenciar o envio de dados do cliente para o servidor. Você pode adicionar `action` a elementos como [`<form/>`](/reference/react-dom/components/form), acessar o status com [`useFormStatus`](/reference/react-dom/hooks/useFormStatus), lidar com o resultado com [`useActionState`](/reference/react/useActionState) e atualizar otimisticamente a UI com [`useOptimistic`](/reference/react/useOptimistic).

Como todos esses recursos funcionam juntos, é difícil lançá-los no canal Stable individualmente. Lançar Actions sem os hooks complementares para acessar os estados do formulário limitaria a usabilidade prática das Actions. A introdução de React Server Components sem a integração de Server Actions complicaria a modificação de dados no servidor.

Antes de podermos lançar um conjunto de recursos para o canal Stable, precisamos garantir que eles funcionem de forma coesa e que os desenvolvedores tenham tudo o que precisam para usá-los em produção. React Canaries nos permite desenvolver esses recursos individualmente e lançar as APIs estáveis incrementalmente até que todo o conjunto de recursos seja concluído.

O conjunto atual de recursos no React Canary está completo e pronto para lançamento.

## A próxima versão principal do React {/*the-next-major-version-of-react*/}

Depois de alguns anos de iteração, `react@canary` agora está pronto para ser lançado em `react@latest`. Os novos recursos mencionados acima são compatíveis com qualquer ambiente em que seu aplicativo é executado, fornecendo tudo o que é necessário para uso em produção. Como o Asset Loading e o Document Metadata podem ser uma alteração de última hora para alguns aplicativos, a próxima versão do React será uma versão principal: **React 19**.

Ainda há mais a ser feito para se preparar para o lançamento. No React 19, também estamos adicionando melhorias solicitadas há muito tempo que exigem alterações de última hora, como suporte para Web Components. Nosso foco agora é concretizar essas alterações, preparar o lançamento, finalizar a documentação para novos recursos e publicar anúncios sobre o que está incluído.

Compartilharemos mais informações sobre tudo o que o React 19 inclui, como adotar os novos recursos do cliente e como criar suporte para React Server Components nos próximos meses.

## Offscreen (renomeado para Activity). {/*offscreen-renamed-to-activity*/}

Desde nossa última atualização, renomeamos um recurso que estamos pesquisando de "Offscreen" para "Activity". O nome "Offscreen" implicava que ele se aplicava apenas a partes do aplicativo que não eram visíveis, mas durante a pesquisa do recurso, percebemos que é possível que partes do aplicativo sejam visíveis e inativas, como conteúdo atrás de um modal. O novo nome reflete mais de perto o comportamento de marcar certas partes do aplicativo como "ativas" ou "inativas".

Activity ainda está em pesquisa e nosso trabalho restante é finalizar os primitivos que são expostos aos desenvolvedores de bibliotecas. Despriorizamos essa área enquanto nos concentramos no lançamento de recursos mais completos.

* * *

Além desta atualização, nossa equipe apresentou em conferências e fez aparições em podcasts para falar mais sobre nosso trabalho e responder a perguntas.

- [Sathya Gunasekaran](/community/team#sathya-gunasekaran) falou sobre o React Compiler na conferência [React India](https://www.youtube.com/watch?v=kjOacmVsLSE)

- [Dan Abramov](/community/team#dan-abramov) fez uma palestra no [RemixConf](https://www.youtube.com/watch?v=zMf_xeGPn6s) intitulada "React from Another Dimension", que explora uma história alternativa de como React Server Components e Actions poderiam ter sido criados

- [Dan Abramov](/community/team#dan-abramov) foi entrevistado no [podcast JS Party do Changelog](https://changelog.com/jsparty/311) sobre React Server Components

- [Matt Carroll](/community/team#matt-carroll) foi entrevistado no [podcast Front-End Fire](https://www.buzzsprout.com/2226499/14462424-interview-the-two-reacts-with-rachel-nabors-evan-bacon-and-matt-carroll), onde ele discutiu [The Two Reacts](https://overreacted.io/the-two-reacts/)

Obrigado [Lauren Tan](https://twitter.com/potetotes), [Sophie Alpert](https://twitter.com/sophiebits), [Jason Bonta](https://threads.net/someextent), [Eli White](https://twitter.com/Eli_White) e [Sathya Gunasekaran](https://twitter.com/_gsathya) por revisar esta publicação.

Obrigado por ler e [vejo você na React Conf](https://conf.react.dev/)!