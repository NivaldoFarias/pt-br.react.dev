---
title: "React Canaries: Habilitando a implementação incremental de recursos fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage e Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar as novas funcionalidades individuais assim que seu design estiver quase finalizado, antes que elas sejam lançadas em uma versão estável - semelhante à forma como o Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de funcionalidades individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novas funcionalidades individuais assim que seu design estiver quase finalizado, antes que elas sejam lançadas em uma versão estável - semelhante à forma como o Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de funcionalidades individuais do React do cronograma de lançamento do React.

</Intro>

---

## Resumo {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como ele é oficialmente suportado, se alguma regressão ocorrer, nós a trataremos com uma urgência semelhante a erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novas funcionalidades individuais do React antes que elas cheguem aos lançamentos semver-estáveis.
* Diferente do canal [Experimental](/community/versioning-policy#experimental-channel), os React Canaries incluem apenas funcionalidades que acreditamos razoavelmente estarem prontas para adoção. Nós encorajamos frameworks a considerar a inclusão de lançamentos Canary do React fixados.
* Nós anunciaremos breaking changes e novas funcionalidades em nosso blog assim que elas chegarem nos lançamentos Canary.
* **Como sempre, o React continua a seguir semver para cada lançamento Stable.**

## Como as funcionalidades do React são geralmente desenvolvidas {/*how-react-features-are-usually-developed*/}

Tipicamente, cada funcionalidade do React passou pelos mesmos estágios:

1. Nós desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. A funcionalidade está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que a funcionalidade mude significativamente.
2. Nós encontramos uma equipe no Meta disposta a nos ajudar a testar essa funcionalidade e fornecer feedback sobre ela. Isso leva a uma rodada de mudanças. À medida que a funcionalidade se torna mais estável, trabalhamos com mais equipes no Meta para experimentá-la.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e disponibilizamos a funcionalidade no branch `main` por padrão, que a maioria dos produtos do Meta usam. Neste ponto, qualquer equipe do Meta pode usar essa funcionalidade.
4. À medida que construímos confiança na direção, também postamos uma RFC para a nova funcionalidade. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de lançar uma versão de código aberto, escrevemos a documentação para a funcionalidade e, finalmente, lançamos a funcionalidade em um lançamento estável do React.

Este playbook funciona bem para a maioria das funcionalidades que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando a funcionalidade está geralmente pronta para uso (etapa 3) e quando ela é lançada em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem do Meta e adotar novas funcionalidades individuais mais cedo (à medida que se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todas as funcionalidades do React eventualmente chegarão a um lançamento Stable.

## Podemos apenas fazer mais lançamentos de menor importância? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos de menor importância para introduzir novas funcionalidades.

No entanto, isso nem sempre é possível. Às vezes, novas funcionalidades estão interconectadas com *outras* novas funcionalidades que ainda não foram totalmente concluídas e que ainda estamos iterando ativamente. Não podemos lançá-las separadamente porque suas implementações são relacionadas. Não podemos versioná-las separadamente porque elas afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas partes que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que semver exigiria que fizéssemos.

No Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit específico fixado a cada semana. Essa também é a abordagem que os lançamentos do React Native vêm seguindo nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novas funcionalidades do React no nível do framework sem ficar acoplado ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar esse fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *em cima* do React inclua uma breaking change relacionada ao React *antes* que essa breaking change seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas breaking changes afetam apenas integrações de frameworks. Isso permite que um framework lance essa alteração em sua própria versão de menor importância sem quebrar o semver.

Os lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que as novas funcionalidades recebam testes abrangentes na comunidade. Esse fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com as alterações em estágios numerados](https://tc39.es/process-document/). Novas funcionalidades do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novas funcionalidades do JavaScript são lançadas nos navegadores antes de serem oficialmente ratificadas como parte da especificação.

## Por que não usar lançamentos experimental em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos Experimental](/community/versioning-policy#canary-channel), nós recomendamos não usá-los em produção porque as APIs experimentais podem passar por breaking changes significativas a caminho da estabilização (ou podem até ser removidas por completo). Embora Canaries também possam conter erros (como em qualquer lançamento), daqui para frente, planejamos anunciar quaisquer breaking changes significativas em Canaries em nosso blog. Canaries são os mais próximos do código que o Meta executa internamente, para que você possa esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, talvez queira considerar a inclusão de uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. O benefício disso é que ele permite que você envie funcionalidades e correções de bugs individuais do React concluídas mais cedo para seus usuários e em seu próprio cronograma de lançamento, de forma semelhante a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo extraídos e comunicar a seus usuários quais mudanças do React estão incluídas em seus lançamentos.

Se você é um autor de framework e quer tentar essa abordagem, entre em contato conosco.

## Anunciando breaking changes e novas funcionalidades antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, anunciamos breaking changes apenas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar breaking changes e novas funcionalidades significativas *à medida que elas chegam* em Canaries. Por exemplo, se mesclarmos uma breaking change que sairá em um Canary, escreveremos uma publicação sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework lançando uma versão principal que atualiza o Canary do React fixo para incluir essa alteração, você pode vincular à nossa postagem do blog em suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, nós iremos vincular a essas postagens do blog já publicadas, que esperamos ajudar nossa equipe a progredir mais rapidamente.

Nós planejamos documentar as APIs à medida que elas chegam em Canaries - mesmo que essas APIs ainda não estejam disponíveis fora delas. As APIs que estão disponíveis apenas em Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229), e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu app ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como Canaries são pré-lançamentos, eles ainda podem incluir breaking changes.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas, e não esperamos breaking changes significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar o suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos interligados apenas para frameworks (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais breaking changes lá.

Isso significa que o React Server Components está pronto para ser adotado por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary do React fixada. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework, e explicar isso a seus usuários. Como exemplo, isso é o que o Next.js App Router faz.)

## Testando bibliotecas em versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary único, pois seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), nós encorajamos as bibliotecas a executar testes em *ambas* as versões Stable e Canary mais recentes. Se você vir uma mudança no comportamento que não foi anunciada, registre um erro no repositório do React para que possamos ajudá-lo a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, ela reduza a quantidade de esforço necessária para atualizar bibliotecas para novas versões principais do React, pois as regressões acidentais seriam encontradas à medida que surgem.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como Canaries sendo uma forma oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis ​​do React.