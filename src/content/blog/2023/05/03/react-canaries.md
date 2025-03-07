---
title: "React Canaries: Habilitando a Implantação Incremental de Recursos Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa versões de ponta do React internamente há muito tempo. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React da programação de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa versões de ponta do React internamente há muito tempo. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React da programação de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como é oficialmente suportado, se quaisquer regressões ocorrerem, trataremos elas com uma urgência semelhante aos erros nas versões estáveis.
* Canaries permitem que você comece a usar novos recursos individuais do React antes de serem lançados nas versões semver-stable.
* Diferente do canal [Experimental](/community/versioning-policy#experimental-channel), React Canaries só inclui recursos que acreditamos razoavelmente estar prontos para adoção. Incentivamos frameworks a considerar a inclusão de lançamentos React Canary fixados.
* Anunciaremos as mudanças radicais e novos recursos em nosso blog à medida que forem lançados nas versões Canary.
* **Como sempre, o React continua seguindo o semver para cada lançamento Estável.**

## Como os recursos do React costumam ser desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passou pelas mesmas etapas:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar este recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. Conforme o recurso se torna mais estável, trabalhamos com mais equipes da Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes com o design. Removemos o prefixo do nome da API e disponibilizamos o recurso no branch `main` por padrão, que a maioria dos produtos da Meta usa. Neste ponto, qualquer equipe da Meta pode usar este recurso.
4. À medida que construímos a confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos prestes a lançar uma versão de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento estável do React.

Este manual funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre o momento em que o recurso está geralmente pronto para uso (etapa 3) e o momento em que é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novos recursos individuais mais cedo (conforme eles se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React acabarão chegando a um lançamento Estável.

## Podemos simplesmente fazer mais lançamentos secundários? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos secundários para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interligados com *outros* novos recursos que ainda não foram totalmente concluídos e nos quais ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit específico fixado toda semana. Esta também é a abordagem que os lançamentos do React Native vêm seguindo nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React no nível do framework sem se acoplar à programação de lançamento global do React.

Gostaríamos de disponibilizar este fluxo de trabalho a outros frameworks e configurações selecionadas. Por exemplo, permite que um framework *acima* do React inclua uma alteração radical relacionada ao React *antes* que essa alteração radical seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas alterações radicais afetam apenas as integrações do framework. Isso permite que um framework lance essa alteração em sua própria versão secundária sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que os novos recursos obtenham testes abrangentes na comunidade. Este fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com as mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes de serem oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos que você não os use em produção, pois as APIs experimentais podem sofrer mudanças radicais significativas a caminho da estabilização (ou podem até ser removidas completamente). Embora os Canaries também possam conter erros (como em qualquer lançamento), no futuro planejamos anunciar quaisquer alterações radicais significativas nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então geralmente você pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queiram continuar usando os lançamentos Estáveis.** No entanto, se você estiver construindo um framework, pode considerar a inclusão de uma versão Canary do React fixada em um commit específico e atualizá-la no seu próprio ritmo. O benefício disso é que permite que você envie recursos e correções de bugs individuais do React concluídos mais cedo para seus usuários e em sua própria programação de lançamento, semelhante a como o React Native vem fazendo nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de analisar quais commits do React estão sendo extraídos e comunicar aos seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você é um autor de framework e deseja tentar esta abordagem, entre em contato conosco.

## Anunciando mudanças radicais e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, anunciamos as mudanças radicais apenas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma maneira oficialmente suportada de consumir o React, planejamos mudar para anunciar as mudanças radicais e novos recursos significativos *à medida que eles chegam* nos Canaries. Por exemplo, se fizermos a merge de uma mudança radical que será lançada em um Canary, escreveremos um post sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework que está lançando uma versão principal que atualiza o canary React fixado para incluir essa alteração, pode vincular à nossa postagem do blog em suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, faremos a ligação a essas postagens de blog já publicadas, o que esperamos que ajude nossa equipe a progredir mais rápido.

Planejamos documentar as APIs à medida que elas chegam nos Canaries - mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir alterações radicais.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos mudanças radicais significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, ainda não podemos lançar suporte para React Server Components em uma versão estável do React porque ainda estamos trabalhando em vários recursos somente para framework interligados (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais mudanças radicais lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviando uma versão Canary fixada do React. (Para evitar a inclusão de duas cópias do React, frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso a seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas em versões Estáveis e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary desde que seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes nos *dois* versões Estáveis e Canary mais recentes. Se você vir uma mudança de comportamento que não foi anunciada, registre um erro no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessária para atualizar bibliotecas para novas versões principais do React, uma vez que as regressões acidentais seriam encontradas à medida que fossem lançadas.

<Note>

Falando estritamente, o Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como os Canaries sendo uma maneira oficialmente suportada de usar o React.

</Note>

## Lançamentos estáveis funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.