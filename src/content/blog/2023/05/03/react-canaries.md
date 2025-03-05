```
---
title: "React Canaries: Habilitando a Implementação Incremental de Recursos Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa há muito tempo versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes que sejam lançados em uma versão estável - semelhante à forma como a Meta usa há muito tempo versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como ele é oficialmente suportado, se alguma regressão ocorrer, vamos tratá-la com urgência semelhante à de erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novos recursos individuais do React antes que eles cheguem aos lançamentos semver-estáveis.
* Diferente do canal [Experimental](/community/versioning-policy#experimental-channel), os React Canaries incluem apenas recursos que acreditamos razoavelmente que estão prontos para adoção. Incentivamos frameworks a considerar a inclusão de lançamentos Canary do React fixados.
* Anunciaremos mudanças radicais e novos recursos em nosso blog conforme eles chegam aos lançamentos Canary.
* **Como sempre, o React continua seguindo o semver para cada lançamento Stable.**

## Como os recursos do React são normalmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, todo recurso do React passou pelas mesmas etapas:

1. Desenvolvemos uma versão inicial e prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar esse recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes na Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e tornamos o recurso disponível no branch `main` por padrão, que a maioria dos produtos Meta usa. Neste ponto, qualquer equipe na Meta pode usar esse recurso.
4. À medida que construímos confiança na direção, também publicamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de fazer um lançamento de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento estável do React.

Este manual de instruções funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novos recursos individuais mais cedo (conforme eles estiverem disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React acabarão chegando a um lançamento Stable.

## Podemos apenas fazer mais lançamentos secundários? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos secundários para apresentar novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e nos quais ainda estamos trabalhando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit específico fixado toda semana. Essa também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React em nível de framework sem ser acoplado ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *sobre* o React inclua uma alteração com quebra do React *antes* que essa alteração com quebra seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas alterações com quebra afetam apenas integrações de frameworks. Isso permite que um framework lance essa alteração em sua própria versão secundária sem quebrar o semver.

Os lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com as mudanças nos estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes que sejam oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar os [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos que você não os use em produção porque as APIs experimentais podem passar por mudanças com quebra significativas em seu caminho para a estabilização (ou podem até mesmo ser removidas por completo). Embora os Canaries também possam conter erros (como em qualquer lançamento), no futuro planejamos anunciar quaisquer alterações com quebra significativas nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então você geralmente pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commit do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, talvez queira considerar a inclusão de uma versão Canary do React fixada em um commit em particular e atualizá-la em seu próprio ritmo. A vantagem disso é que permite que você envie recursos e correções de bugs individuais do React mais cedo para seus usuários e em seu próprio cronograma de lançamento, semelhante à maneira como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo recebidos e comunicar aos seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você for um autor de framework e quiser experimentar essa abordagem, entre em contato conosco.

## Anunciando alterações com quebra e novos recursos com antecedência {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, só anunciamos mudanças com quebra no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma maneira oficialmente suportada de consumir o React, planejamos mudar para anunciar mudanças com quebra e novos recursos significativos *conforme eles chegam* aos Canaries. Por exemplo, se mesclarmos uma alteração com quebra que será lançada em um Canary, escreveremos uma publicação sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você for um autor de framework lançando uma versão principal que atualiza o Canary do React fixado para incluir essa alteração, você pode vincular à nossa postagem do blog a partir de suas notas de lançamento. Por fim, quando uma versão principal estável do React estiver pronta, faremos link para essas postagens do blog já publicadas, o que esperamos que ajude nossa equipe a progredir mais rapidamente.

Planejamos documentar as APIs conforme elas chegam aos Canaries - mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir mudanças com quebra.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos mudanças com quebra significativas relacionadas ao seu contrato de API voltada para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos de framework somente interligados (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais mudanças com quebra lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary fixada do React. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que enviam com seu framework e explicar isso a seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas contra versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [apresentamos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes contra as versões *Stable* e *Canary* mais recentes. Se você vir uma alteração no comportamento que não foi anunciada, registre um erro no repositório do React para que possamos ajudá-lo a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessária para atualizar as bibliotecas para as novas versões principais do React, pois as regressões acidentais seriam encontradas conforme elas chegam.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como Canaries sendo uma maneira oficialmente suportada de usar o React.

</Note>

## Os lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.
```