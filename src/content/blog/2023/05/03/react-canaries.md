---
title: "React Canaries: Habilitando o lançamento incremental de recursos fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seus projetos estiverem próximos do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa versões de ponta do React internamente há muito tempo. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações com curadoria, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), and [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seus projetos estiverem próximos do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa versões de ponta do React internamente há muito tempo. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações com curadoria, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) para React oficialmente suportado. Como ele é oficialmente suportado, se alguma regressão for detectada, iremos tratá-las com urgência semelhante a erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novos recursos individuais do React antes que eles cheguem aos lançamentos semver-estáveis.
* Ao contrário do canal [Experimental](/community/versioning-policy#experimental-channel), React Canaries incluem apenas recursos que acreditamos razoavelmente estarem prontos para adoção. Incentivamos os frameworks a considerar a vinculação de lançamentos Canary do React.
* Anunciaremos alterações importantes e novos recursos em nosso blog à medida que eles chegarem aos lançamentos Canary.
* **Como sempre, o React continua a seguir o semver para cada lançamento Stable.**

## Como os recursos do React são normalmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, todo recurso do React passou pelas mesmas etapas:

1. Desenvolvemos uma versão inicial e prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso seja alterado significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar este recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes na Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no projeto. Removemos o prefixo do nome da API e tornamos o recurso disponível no branch `main` por padrão, que a maioria dos produtos Meta usam. Neste ponto, qualquer equipe na Meta pode usar este recurso.
4. À medida que construímos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o projeto funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos prestes a cortar um lançamento de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento estável do React.

Este roteiro funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novos recursos individuais mais cedo (assim que estiverem disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React acabarão chegando a um lançamento Stable.

## Podemos apenas fazer mais lançamentos menores? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos menores para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e que ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações são relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versões principais, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit específico fixo toda semana. Essa também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React no nível do framework sem ficar acoplado ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar esse fluxo de trabalho para outros frameworks e configurações com curadoria. Por exemplo, ele permite que um framework *no topo* do React inclua uma mudança importante relacionada ao React *antes* que essa mudança seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas mudanças importantes afetam apenas as integrações do framework. Isso permite que um framework lance essa alteração em sua própria versão secundária sem quebrar o semver.

Os lançamentos contínuos com o canal Canaries nos permitirão ter um ciclo de feedback mais restrito e garantir que os novos recursos recebam testes abrangentes na comunidade. Esse fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes de estarem em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes de serem oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos não usá-los em produção porque as APIs experimentais podem sofrer alterações significativas em sua estabilização (ou podem até ser removidas inteiramente). Embora os Canaries também possam conter erros (como em qualquer lançamento), planejamos anunciar quaisquer alterações importantes nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então você pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração com curadoria (como um framework) queiram continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, talvez queira considerar a vinculação de uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. O benefício disso é que ele permite que você envie recursos e correções de bugs do React concluídos individualmente mais cedo para seus usuários e em seu próprio cronograma de lançamento, semelhante à forma como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo extraídos e comunicar aos seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você é autor de um framework e deseja tentar essa abordagem, entre em contato conosco.

## Anunciando alterações importantes e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React a qualquer momento.

Tradicionalmente, anunciamos mudanças importantes apenas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar alterações importantes e novos recursos *assim que eles chegarem* nos Canaries. Por exemplo, se unirmos uma alteração importante que será lançada no Canary, escreveremos uma postagem sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Em seguida, se você for um autor de framework criando um lançamento principal que atualiza o canary do React fixado para incluir essa alteração, poderá vincular à postagem do nosso blog em suas notas de lançamento. Por fim, quando uma versão principal estável do React estiver pronta, faremos um link para essas postagens do blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rápido.

Planejamos documentar as APIs à medida que elas chegam nos Canaries - mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e alguns outros (como `cache` e `createServerContext`) para os quais enviaremos RFCs.

## Canaries deve ser fixado {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se sempre de fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir alterações importantes.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos mudanças importantes relacionadas ao contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos apenas do framework interligados (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais mudanças importantes lá.

Isso significa que os React Server Components estão prontos para serem adotados pelos frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary do React fixada. (Para evitar a vinculação de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso aos seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas em versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes em *ambas* as versões mais recentes Stable e Canary. Se você vir uma mudança no comportamento que não foi anunciada, por favor, registre um erro no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessário para atualizar as bibliotecas para novas versões principais do React, pois as regressões acidentais seriam encontradas à medida que chegam.

<Note>

Falando estritamente, o Canary não é um canal de lançamento *novo* - costumava se chamar Next. No entanto, decidimos renomeá-lo para evitar confusão com Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como os Canaries serem uma forma oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.
``