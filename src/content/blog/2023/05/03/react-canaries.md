---
title: "React Canaries: Habilitando a Implantação Incremental de Recursos Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar recursos novos e individuais assim que seu design estiver próximo de finalizado, antes que sejam lançados em uma versão estável - semelhante a como a Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar recursos novos e individuais assim que seu design estiver próximo de finalizado, antes que sejam lançados em uma versão estável - semelhante a como a Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como é oficialmente suportado, se quaisquer regressões surgirem, trataremos elas com uma urgência semelhante a bugs em lançamentos estáveis.
* Canaries permitem que você comece a usar recursos individuais do novo React antes de eles chegarem nos lançamentos semver-estáveis.
* Diferente do canal [Experimental](/community/versioning-policy#experimental-channel), React Canaries incluem apenas recursos que acreditamos razoavelmente estar prontos para adoção. Encorajamos frameworks a considerar a inclusão de lançamentos de Canary React fixados.
* Anunciaremos mudanças significativas e novos recursos em nosso blog conforme eles chegarem nos lançamentos Canary.
* **Como sempre, React continua a seguir semver para cada lançamento Stable.**

## Como os recursos do React são normalmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passou pelos mesmos estágios:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso só está disponível no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar esse recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. Conforme o recurso se torna mais estável, trabalhamos com mais equipes da Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e tornamos o recurso disponível no branch `main` por padrão, que a maioria dos produtos da Meta usa. Neste ponto, qualquer equipe da Meta pode usar este recurso.
4. Conforme construímos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de lançar uma versão de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento estável do React.

Este manual funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novos recursos individuais mais cedo (conforme eles se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React acabarão chegando em um lançamento Stable.

## Podemos apenas fazer mais lançamentos secundários? {/*can-we-just-do-more-minor-releases*/}

Geralmente, nós *usamos* lançamentos secundários para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e que ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit fixado específico a cada semana. Essa também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native está fixado a um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React no nível do framework sem ficar acoplado ao cronograma de lançamento global do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, permite que um framework *em cima do* React inclua uma mudança significativa relacionada ao React *antes* que essa mudança significativa seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas mudanças significativas afetam apenas as integrações do framework. Isso permite que um framework lance tal mudança em sua própria versão secundária sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que os novos recursos recebam testes abrangentes na comunidade. Esse fluxo de trabalho está mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com as mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes que sejam oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar os [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos não usá-los em produção porque as APIs experimentais podem passar por grandes mudanças significativas em sua jornada para a estabilização (ou podem até ser removidas completamente). Embora os Canaries também possam conter erros (como em qualquer lançamento), no futuro planejamos anunciar quaisquer mudanças significativas nos Canaries em nosso blog. Canaries são o mais próximo do código que a Meta executa internamente, então você pode geralmente esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queiram continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, talvez queira considerar a inclusão de uma versão Canary do React fixada a um commit específico e atualizá-la em seu próprio ritmo. A vantagem disso é que permite que você envie recursos e correções de bugs individuais do React concluídos mais cedo para seus usuários e em seu próprio cronograma de lançamento, de forma semelhante a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo inseridos e comunicar aos seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você é um autor de framework e deseja experimentar esta abordagem, entre em contato conosco.

## Anunciando mudanças significativas e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, só anunciamos mudanças significativas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar mudanças significativas e novos recursos *conforme eles chegam* nos Canaries. Por exemplo, se mesclarmos uma mudança significativa que sairá em um Canary, escreveremos uma postagem sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework lançando uma versão principal que atualiza o canary React fixado para incluir essa mudança, você pode vincular à nossa postagem de blog em suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, faremos um link para essas postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rapidamente.

Planejamos documentar as APIs conforme elas chegam nos Canaries - mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que só estão disponíveis nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229), e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir mudanças significativas.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções React Server Components foram finalizadas e não esperamos mudanças significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos interligados apenas para o framework (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais mudanças significativas lá.

Isso significa que o React Server Components está pronto para ser adotado por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary fixada do React. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso a seus usuários. Como exemplo, é isso que Next.js App Router faz.)

## Testando bibliotecas em relação às versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem toda e qualquer versão Canary, pois seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes *tanto* nos últimos lançamentos Stable quanto nos últimos lançamentos Canary. Se você vir uma mudança no comportamento que não foi anunciada, registre um bug no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessário para atualizar bibliotecas para novas versões principais do React, uma vez que as regressões acidentais seriam encontradas conforme chegam.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como Canaries sendo uma forma oficialmente suportada de usar o React.

</Note>

## Os lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.