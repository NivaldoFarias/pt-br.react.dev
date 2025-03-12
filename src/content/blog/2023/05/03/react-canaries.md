---
title: "React Canaries: Habilitando a implantação incremental de recursos fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, e Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver quase finalizado, antes que sejam lançados em uma versão estável - similar ao que o Meta usa há muito tempo versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas como frameworks desacoplem a adoção de recursos React individuais do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver quase finalizado, antes que sejam lançados em uma versão estável - similar ao que o Meta usa há muito tempo versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas como frameworks desacoplem a adoção de recursos React individuais do cronograma de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como é oficialmente suportado, se alguma regressão acontecer, iremos tratá-las com uma urgência similar aos erros em lançamentos estáveis.
* Canaries permitem que você comece a usar recursos React individuais antes que eles cheguem aos lançamentos semver-estáveis.
* Ao contrário do canal [Experimental](/community/versioning-policy#experimental-channel), React Canaries só incluem recursos que acreditamos razoavelmente estarem prontos para adoção. Encorajamos os frameworks a considerar a junção de lançamentos Canary React fixados.
* Anunciaremos breaking changes e novos recursos em nosso blog à medida que chegarem aos lançamentos Canary.
* **Como sempre, o React continua a seguir o semver para cada lançamento Stable.**

## Como os recursos React são normalmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso React passou pelos mesmos estágios:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe no Meta disposta a nos ajudar a testar esse recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes no Meta para testá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e disponibilizamos o recurso no branch `main` por padrão, que a maioria dos produtos Meta usam. Neste ponto, qualquer equipe no Meta pode usar esse recurso.
4. À medida que construímos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos prestes a cortar um lançamento de código aberto, escrevemos documentação para o recurso e finalmente lançamos o recurso em um lançamento React estável.

Este manual funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem do Meta e adotar novos recursos individuais mais cedo (à medida que ficam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos React eventualmente chegarão a um lançamento Stable.

## Podemos apenas fazer mais lançamentos menores? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos menores para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos são interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e que ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

No Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit fixado específico toda semana. Esta também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos React no nível do framework sem ficar acoplado ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *em cima* do React inclua uma breaking change relacionada ao React *antes* que essa breaking change seja incluída em um lançamento React estável. Isso é particularmente útil porque algumas breaking changes afetam apenas as integrações do framework. Isso permite que um framework lance tal alteração em sua própria versão menor sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um ciclo de feedback mais apertado e garantir que os novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho está mais próximo de como o TC39, o comitê de padrões JavaScript, [lidam com as mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos JavaScript são enviados em navegadores antes que sejam oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos Experimental](/community/versioning-policy#canary-channel), nós recomendamos que você não os use em produção porque as APIs experimentais podem passar por mudanças significativas que estouram a compatibilidade em seu caminho para a estabilização (ou podem até ser removidas inteiramente). Enquanto Canaries também podem conter erros (como em qualquer lançamento), daqui para frente planejamos anunciar quaisquer alterações importantes nas Canaries em nosso blog. Canaries são os mais próximos do código que o Meta executa internamente, então você geralmente pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, talvez queira considerar a junção de uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. O benefício disso é que ele permite que você envie recursos React individuais concluídos e correções de bugs mais cedo para seus usuários e em seu próprio cronograma de lançamento, similar a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de analisar quais commits React estão sendo incluídos e comunicar aos seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você é um autor de framework e deseja experimentar essa abordagem, entre em contato conosco.

## Anunciando breaking changes e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Lançamentos Canary representam nossa melhor estimativa do que estará no próximo lançamento React estável em um determinado momento.

Tradicionalmente, anunciamos breaking changes apenas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar breaking changes e novos recursos significativos *à medida que eles chegam* nas Canaries. Por exemplo, se nós mesclarmos uma breaking change que sairá em um Canary, escreveremos uma postagem sobre isso no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework cortando um lançamento principal que atualiza o canary React fixado para incluir essa mudança, você pode vincular à nossa postagem do blog em suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, iremos vincular a essas postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rápido.

Planejamos documentar as APIs à medida que elas chegam nas Canaries - mesmo que essas APIs ainda não estejam disponíveis fora delas. As APIs que estão disponíveis apenas nas Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu app ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir breaking changes.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos breaking changes significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos interligados apenas para o framework (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais breaking changes lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary do React fixada. (Para evitar a junção de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso para seus usuários. Como exemplo, isso é o que o Next.js App Router faz.)

## Testando bibliotecas em versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes em *ambas* as versões Stable mais recentes e Canary mais recentes. Se você vir uma mudança no comportamento que não foi anunciada, registre um erro no repositório do React para que possamos ajudá-lo a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessário para atualizar as bibliotecas para novas versões principais do React, uma vez que as regressões acidentais seriam encontradas à medida que chegam.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como Canaries sendo uma forma oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.