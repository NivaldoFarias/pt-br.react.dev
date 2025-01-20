---
title: "React Canaries: Habilitando o Rollout Incremental de Recursos Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage e Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes de serem lançados em uma versão estável - similar à forma como o Meta há muito utiliza versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](https://reactjs.org/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações curadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamentos do React.
---

3 de maio de 2023 por [Dan Abramov](https://twitter.com/dan_abramov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage) e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que seu design estiver próximo do final, antes de serem lançados em uma versão estável - similar à forma como o Meta há muito utiliza versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](https://reactjs.org/community/versioning-policy#canary-channel) oficialmente suportado. Ele permite que configurações curadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamentos do React.

</Intro>

---

## tl;dr {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](https://reactjs.org/community/versioning-policy#canary-channel) oficialmente suportado para o React. Como é oficialmente suportado, se qualquer regressão ocorrer, iremos tratá-las com a mesma urgência que erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novos recursos individuais do React antes que eles estejam disponíveis nas versões estáveis semver.
* Ao contrário do canal [Experimental](https://reactjs.org/community/versioning-policy#experimental-channel), os Canaries do React incluem apenas recursos que acreditamos razoavelmente estar prontos para adoção. Encorajamos frameworks a considerar empacotar versões Canary do React fixas.
* Anunciaremos mudanças quebradoras e novos recursos em nosso blog à medida que forem lançados nas versões Canary.
* **Como sempre, o React continua a seguir semver para cada lançamento estável.**

## Como os recursos do React são geralmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passa pelas mesmas etapas:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Nesse ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe no Meta disposta a nos ajudar a testar esse recurso e fornecer feedback. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes do Meta para testá-lo.
3. Eventualmente, sentimos confiança no design. Removemos o prefixo do nome da API e tornamos o recurso disponível por padrão na branch `main`, que a maioria dos produtos do Meta utiliza. Nesse ponto, qualquer equipe no Meta pode usar esse recurso.
4. À medida que construímos confiança na direção, também publicamos um RFC para o novo recurso. Nesse ponto, sabemos que o design funciona para um conjunto amplo de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos próximos de cortar um lançamento de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em uma versão estável do React.

Este plano funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver um gap significativo entre quando o recurso está geralmente pronto para uso (etapa 3) e quando é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem do Meta e adotar novos recursos individuais mais cedo (à medida que se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React eventualmente farão parte de um lançamento estável.

## Podemos apenas fazer mais lançamentos menores? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *fazemos* lançamentos menores para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e nos quais ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar sobre as partes que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

No Meta, resolvemos esse problema construindo o React a partir da branch `main` e atualizando manualmente para um commit fixo específico a cada semana. Esta também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native está vinculado a um commit específico da branch `main` do repositório do React. Isso permite que o React Native inclua correções de bugs importantes e adote novos recursos do React de forma incremental ao nível do framework, sem ficar acoplado ao cronograma de lançamentos do React global.

Gostaríamos de tornar esse fluxo de trabalho disponível para outros frameworks e configurações curadas. Por exemplo, isso permite que um framework *em cima do* React inclua uma mudança quebradora relacionada ao React *antes* que essa mudança quebradora seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas mudanças quebradoras afetam apenas integrações de framework. Isso permite que um framework lance tal mudança em sua própria versão menor sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um ciclo de feedback mais próximo e garantir que novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho está mais próximo de como o TC39, o comitê de padrões do JavaScript, [lida com mudanças em etapas numeradas](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos sobre o React antes de estarem em um lançamento estável do React, assim como novos recursos do JavaScript são disponibilizados nos navegadores antes de serem oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar [lançamentos Experimentais](https://reactjs.org/community/versioning-policy#canary-channel), recomendamos não utilizá-los em produção porque APIs experimentais podem passar por mudanças quebradoras significativas no caminho para estabilização (ou podem até ser removidas completamente). Embora Canaries também possam conter erros (como qualquer lançamento), no futuro, planejamos anunciar quaisquer mudanças quebradoras significativas em Canaries em nosso blog. Canaries são o mais próximo do código que o Meta executa internamente, então você pode geralmente esperar que sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e analisar manualmente o log de commits do GitHub ao atualizar entre os commits fixos.

**Esperamos que a maioria das pessoas usando o React fora de uma configuração curada (como um framework) queira continuar usando os lançamentos Estáveis.** No entanto, se você está construindo um framework, pode querer considerar empacotar uma versão Canary do React fixada em um commit particular e atualizá-la em seu próprio ritmo. O benefício disso é que isso permite que você implemente individualmente recursos concluídos do React e correções de bugs mais cedo para seus usuários e em seu próprio cronograma de lançamentos, similar à forma como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria uma responsabilidade adicional para revisar quais commits do React estão sendo incluídos e comunicar aos seus usuários quais mudanças do React estão incluídas em seus lançamentos.

Se você é um autor de framework e deseja tentar essa abordagem, entre em contato conosco.

## Anunciando mudanças quebradoras e novos recursos cedo {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que irá para o próximo lançamento estável do React a qualquer momento.

Tradicionalmente, anunciamos mudanças quebradoras apenas no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para anunciar mudanças quebradoras e novos recursos significativos *à medida que forem lançados* nos Canaries. Por exemplo, se mesclarmos uma mudança quebradora que será lançada em um Canary, escreveremos um post sobre isso em nosso blog do React, incluindo codemods e instruções de migração se necessário. Então, se você é um autor de framework cortando um lançamento principal que atualiza o React canary fixo para incluir essa mudança, você pode vincular o nosso post do blog nas notas do seu lançamento. Finalmente, quando uma versão estável maior do React estiver pronta, vincularemos às postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rápido.

Planejamos documentar APIs à medida que forem lançadas nos Canaries - mesmo que essas APIs ainda não estejam disponíveis fora deles. APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229), e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a *exata* versão do Canary que você está usando. Como os Canaries são pré-lançamentos, eles podem ainda incluir mudanças quebradoras.

## Exemplo: Componentes de Servidor do React {/*example-react-server-components*/}

Conforme [anunciamos em março](https://reactjs.org/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções dos Componentes de Servidor do React foram finalizadas, e não esperamos mudanças quebradoras significativas relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos ainda lançar suporte para Componentes de Servidor do React em uma versão estável do React porque ainda estamos trabalhando em várias funcionalidades interligadas apenas para frameworks (como [carregamento de ativos](https://reactjs.org/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais mudanças quebradoras lá.

Isso significa que os Componentes de Servidor do React estão prontos para serem adotados por frameworks. Contudo, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é enviar uma versão Canary fixa do React. (Para evitar empacotar duas cópias do React, frameworks que desejam fazer isso precisariam impor a resolução do `react` e `react-dom` para o Canary fixo com o qual enviam seu framework, e explicar isso aos seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas contra versões Estáveis e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois isso seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), encorajamos as bibliotecas a executar testes contra *tanto* a versão Estável mais recente quanto a versão Canary mais recente. Se você notar uma mudança de comportamento que não foi anunciada, por favor, relate um erro no repositório do React para que possamos ajudar a diagnosticar. Esperamos que, à medida que essa prática se torne amplamente adotada, reduzirá o esforço necessário para atualizar bibliotecas para novas versões principais do React, uma vez que regressões acidentais seriam encontradas à medida que surgissem.

<Note>

Estritamente falando, Canary não é um canal de lançamento *novo* - costumava ser chamado de Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um *novo* canal de lançamento para comunicar as novas expectativas, como Canaries sendo uma maneira oficialmente suportada de usar o React.

</Note>

## Lançamentos estáveis funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma mudança nos lançamentos estáveis do React.