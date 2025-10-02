---
title: "React Canaries: Habilitando o lançamento incremental de funcionalidades fora da Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage e Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novas funcionalidades individuais assim que seu design estiver próximo do final, antes que sejam lançadas em uma versão estável -- semelhante a como a Meta há muito tempo usa versões de ponta do React internamente. Estamos introduzindo um novo [Canary release channel](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas como frameworks desacoplem a adoção de funcionalidades individuais do React do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novas funcionalidades individuais assim que seu design estiver próximo do final, antes que sejam lançadas em uma versão estável -- semelhante a como a Meta há muito tempo usa versões de ponta do React internamente. Estamos introduzindo um novo [Canary release channel](/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações selecionadas como frameworks desacoplem a adoção de funcionalidades individuais do React do cronograma de lançamento do React.

</Intro>

---

## tl;dr {/*tldr*/}

*   Estamos introduzindo um [Canary release channel](/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como é oficialmente suportado, se alguma regressão ocorrer, nós as trataremos com uma urgência semelhante a erros em lançamentos estáveis.
*   Os Canaries permitem que você comece a usar novas funcionalidades individuais do React antes que elas cheguem aos lançamentos semver-estáveis.
*   Ao contrário do canal [Experimental](/community/versioning-policy#experimental-channel), os React Canaries incluem apenas funcionalidades que acreditamos razoavelmente estarem prontas para adoção. Encorajamos os frameworks a considerar o agrupamento de lançamentos Canary React fixados.
*   Anunciaremos breaking changes e novas funcionalidades em nosso blog assim que chegarem aos lançamentos Canary.
*   **Como sempre, o React continua a seguir o semver para cada lançamento Stable.**

## Como as funcionalidades do React são geralmente desenvolvidas {/*how-react-features-are-usually-developed*/}

Normalmente, cada funcionalidade do React passou pelas mesmas etapas:

1.  Nós desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. A funcionalidade está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que a funcionalidade mude significativamente.
2.  Encontramos uma equipe na Meta disposta a nos ajudar a testar esta funcionalidade e fornecer feedback sobre ela. Isso leva a uma rodada de mudanças. À medida que a funcionalidade se torna mais estável, trabalhamos com mais equipes na Meta para experimentá-la.
3.  Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e tornamos a funcionalidade disponível no branch `main` por padrão, que a maioria dos produtos Meta usam. Neste ponto, qualquer equipe na Meta pode usar esta funcionalidade.
4.  À medida que construímos confiança na direção, também postamos um RFC para a nova funcionalidade. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5.  Quando estamos perto de cortar um lançamento de código aberto, escrevemos documentação para a funcionalidade e, finalmente, lançamos a funcionalidade em um lançamento React estável.

Este manual funciona bem para a maioria das funcionalidades que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando a funcionalidade está geralmente pronta para uso (etapa 3) e quando é lançada em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novas funcionalidades individuais mais cedo (à medida que se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todas as funcionalidades do React acabarão chegando a um lançamento Stable.

## Podemos apenas fazer mais lançamentos minor? {/*can-we-just-do-more-minor-releases*/}

Geralmente, nós *usamos* lançamentos minor para introduzir novas funcionalidades.

No entanto, isso nem sempre é possível. Às vezes, novas funcionalidades estão interconectadas com *outras* novas funcionalidades que ainda não foram totalmente concluídas e nas quais ainda estamos iterando ativamente. Não podemos lançá-las separadamente porque suas implementações estão relacionadas. Não podemos versioná-las separadamente porque afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas partes que não estão prontas sem uma série de lançamentos de versão major, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos este problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit fixado específico a cada semana. Esta é também a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório React. Isso permite que o React Native inclua correções de erros importantes e adote incrementalmente novas funcionalidades do React no nível do framework sem ficar acoplado ao cronograma de lançamento global do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *em cima do* React inclua uma breaking change relacionada ao React *antes* que esta breaking change seja incluída em um lançamento React estável. Isso é particularmente útil porque algumas breaking changes afetam apenas as integrações do framework. Isso permite que um framework lance tal mudança em sua própria versão minor sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que novas funcionalidades recebam testes abrangentes na comunidade. Este fluxo de trabalho é mais próximo de como o TC39, o comitê de padrões JavaScript, [lida com mudanças em estágios numerados](https://tc39.es/process-document/). Novas funcionalidades do React podem estar disponíveis em frameworks construídos em React antes de estarem em um lançamento estável do React, assim como novas funcionalidades JavaScript são lançadas em navegadores antes de serem oficialmente ratificadas como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos experimentais](/community/versioning-policy#canary-channel), recomendamos não usá-los em produção porque as APIs experimentais podem sofrer breaking changes significativas em seu caminho para a estabilização (ou podem até ser removidas completamente). Embora os Canaries também possam conter erros (como em qualquer lançamento), daqui para frente planejamos anunciar quaisquer breaking changes significativas nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então você pode geralmente esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, pode querer considerar agrupar uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. O benefício disso é que ele permite que você envie funcionalidades e correções de erros individuais do React mais cedo para seus usuários e em seu próprio cronograma de lançamento, semelhante a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo puxados e comunicar aos seus usuários quais mudanças do React estão incluídas em seus lançamentos.

Se você é um autor de framework e quer experimentar esta abordagem, entre em contato conosco.

## Anunciando breaking changes e novas funcionalidades antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento React estável a qualquer momento.

Tradicionalmente, anunciamos breaking changes apenas no *final* do ciclo de lançamento (ao fazer um lançamento major). Agora que os lançamentos Canary são uma maneira oficialmente suportada de consumir React, planejamos mudar para anunciar breaking changes e novas funcionalidades significativas *à medida que chegam* nos Canaries. Por exemplo, se mergearmos uma breaking change que será lançada em um Canary, escreveremos uma postagem sobre isso no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework cortando um lançamento major que atualiza o Canary React fixado para incluir essa mudança, você pode linkar para nossa postagem no blog a partir de suas notas de lançamento. Finalmente, quando uma versão major estável do React estiver pronta, linkaremos para essas postagens de blog já publicadas, o que esperamos que ajude nossa equipe a progredir mais rápido.

Planejamos documentar as APIs à medida que chegam nos Canaries -- mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir breaking changes.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos breaking changes significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em várias funcionalidades entrelaçadas apenas para framework (como [carregamento de assets](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais breaking changes lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento major do React, a única maneira de um framework adotá-los é enviar uma versão Canary fixada do React. (Para evitar agrupar duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework, e explicar isso para seus usuários. Como um exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas em relação às versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [originalmente introduzimos os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), encorajamos as bibliotecas a executar testes em relação *tanto* às versões Stable quanto às Canary mais recentes. Se você vir uma mudança no comportamento que não foi anunciada, por favor, registre um bug no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que esta prática se torne amplamente adotada, ela reduza a quantidade de esforço necessária para atualizar as bibliotecas para novas versões major do React, já que as regressões acidentais seriam encontradas à medida que chegam.

<Note>

Estritamente falando, Canary não é um canal de lançamento *novo* -- costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como os Canaries serem uma maneira oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma mudança nos lançamentos React estáveis.
