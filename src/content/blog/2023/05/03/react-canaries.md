---
title: "React Canaries: Habilitando a Implantação Incremental de Recursos Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, e Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar recursos individuais novos assim que seu design estiver perto de finalizado, antes que sejam lançados em uma versão estável - semelhante à forma como o Meta usa há muito tempo versões de ponta do React internamente. Estamos apresentando um novo [Canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar recursos individuais novos assim que seu design estiver perto de finalizado, antes que sejam lançados em uma versão estável — semelhante à forma como o Meta usa há muito tempo versões de ponta do React internamente. Estamos apresentando um novo [Canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.

</Intro>

---

## Resumo rápido {/*tldr*/}

* Estamos apresentando um [Canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como ele é oficialmente suportado, se houver alguma regressão, vamos tratá-la com urgência semelhante a erros em versões estáveis.
* Canaries permitem que você comece a usar recursos individuais do React antes que eles cheguem aos lançamentos semver-estáveis.
* Ao contrário do canal [Experimental](/community/versioning-policy#experimental-channel), os React Canaries incluem apenas recursos que acreditamos razoavelmente estarem prontos para adoção. Incentivamos os frameworks a considerar a inclusão de lançamentos Canary do React fixados.
* Anunciaremos *breaking changes* e novos recursos em nosso blog assim que chegarem aos lançamentos Canary.
* **Como sempre, o React continua a seguir o semver para cada lançamento Stable.**

## Como os recursos do React são geralmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passou pelos mesmos estágios:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso só está disponível no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe no Meta disposta a nos ajudar a testar esse recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes no Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e disponibilizamos o recurso na branch `main` por padrão, que a maioria dos produtos Meta usa. Neste ponto, qualquer equipe no Meta pode usar esse recurso.
4. À medida que construímos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de cortar um lançamento de código aberto, escrevemos documentação para o recurso e finalmente o lançamos em um lançamento estável do React.

Este manual funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem do Meta e adotar novos recursos individuais mais cedo (assim que estiverem disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React eventualmente farão parte de um lançamento Stable.

## Podemos apenas fazer mais lançamentos secundários? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos secundários para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos são interligados com *outros* novos recursos que ainda não foram totalmente concluídos e que ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

No Meta, resolvemos esse problema construindo o React a partir da `main` branch e atualizando-o manualmente para um commit específico fixado toda semana. Essa também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico da `main` branch do repositório React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React no nível do framework sem se acoplar ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *no topo* do React inclua uma *breaking change* relacionada ao React *antes* que essa *breaking change* seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas *breaking changes* afetam apenas as integrações do framework. Isso permite que um framework lance essa mudança em sua própria versão secundária sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um ciclo de feedback mais apertado e garantir que os novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com as mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes de serem oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar os [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos que você não os use em produção porque as APIs experimentais podem sofrer *breaking changes* significativas a caminho da estabilização (ou podem até ser removidas inteiramente). Embora os Canaries também possam conter erros (como em qualquer lançamento), no futuro planejamos anunciar quaisquer *breaking changes* significativas em Canaries em nosso blog. Os Canaries são os mais próximos do código que o Meta executa internamente, então você pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e analisar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver criando um framework, convém considerar a inclusão de uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. A vantagem disso é que permite que você envie recursos e correções de bugs individuais do React concluídos mais cedo para seus usuários e em seu próprio cronograma de lançamento, semelhante à forma como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo incluídos e comunicar aos seus usuários quais mudanças do React estão incluídas em seus lançamentos.

Se você é um autor de framework e quer experimentar essa abordagem, entre em contato conosco.

## Anunciando *breaking changes* e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, só anunciamos *breaking changes* no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar *breaking changes* e novos recursos significativos *conforme eles chegam* nos Canaries. Por exemplo, se mesclarmos uma *breaking change* que sairá em um Canary, escreveremos uma postagem sobre isso no blog do React, incluindo *codemods* e instruções de migração, se necessário. Então, se você é um autor de framework que está lançando uma versão principal que atualiza o Canary do React fixado para incluir essa mudança, você pode vincular à nossa postagem do blog de suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, faremos um link para essas postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rapidamente.

Planejamos documentar as APIs à medida que elas chegam nos Canaries — mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão apenas disponíveis nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229), e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir *breaking changes*.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos *breaking changes* significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos interligados apenas para o framework (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais *breaking changes* lá.

Isso significa que o React Server Components está pronto para ser adotado por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary fixada do React. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso a seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas em versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [apresentamos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes em *ambas* as versões mais recentes Stable e Canary. Se você vir uma mudança no comportamento que não foi anunciada, registre um bug no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se tornar amplamente adotada, ela reduza a quantidade de esforço necessário para atualizar bibliotecas para novas versões principais do React, pois as regressões acidentais seriam encontradas assim que chegassem.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* — costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como Canaries sendo uma forma oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.