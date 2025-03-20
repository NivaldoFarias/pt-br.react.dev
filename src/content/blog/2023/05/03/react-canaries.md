---
title: "React Canaries: Habilitando a Implantação de Recursos Incrementais Fora do Meta"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que o design estiver próximo do final, antes de serem lançados em uma versão estável -- semelhante à forma como a Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente compatível. Ele permite que configurações com curadoria, como frameworks, desacoplem a adoção de recursos React individuais do cronograma de lançamento do React.
---

3 de maio de 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar novos recursos individuais assim que o design estiver próximo do final, antes de serem lançados em uma versão estável -- semelhante à forma como a Meta tem usado versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente compatível. Ele permite que configurações com curadoria, como frameworks, desacoplem a adoção de recursos React individuais do cronograma de lançamento do React.

</Intro>

---

## Resumo (Em poucas palavras) {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) oficialmente compatível para React. Como é oficialmente compatível, se quaisquer regressões aparecerem, nós as trataremos com uma urgência semelhante aos erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novos recursos React individuais antes que eles cheguem aos lançamentos semver-estáveis.
* Diferente do canal [Experimental](/community/versioning-policy#experimental-channel), os React Canaries só incluem recursos que acreditamos razoavelmente estarem prontos para adoção. Encorajamos os frameworks a considerar a inclusão de lançamentos Canary React fixados.
* Anunciaremos mudanças que quebram e novos recursos em nosso blog à medida que chegarem nos lançamentos Canary.
* **Como sempre, React continua a seguir o semver para cada lançamento Stable.**

## Como os recursos do React são geralmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passou pelos mesmos estágios:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar este recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes na Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e disponibilizamos o recurso no branch `main` por padrão, que a maioria dos produtos Meta usam. Neste ponto, qualquer equipe na Meta pode usar este recurso.
4. Ao construirmos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de lançar um lançamento de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento React estável.

Este playbook funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta, e adotar novos recursos individuais mais cedo (conforme eles se tornam disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React acabarão chegando em um lançamento Stable.

## Podemos apenas fazer mais lançamentos secundários? {/*can-we-just-do-more-minor-releases*/}

Geralmente, nós *usamos* lançamentos secundários para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interconectados com *outros* novos recursos que ainda não foram totalmente concluídos e que ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas partes que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit fixado específico toda semana. Essa também é a abordagem que os lançamentos do React Native vêm seguindo nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório React. Isso permite que o React Native inclua correções de bugs importantes e adote incrementalmente novos recursos do React em nível de framework sem se acoplar ao cronograma global de lançamento do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações com curadoria. Por exemplo, ele permite que um framework *no topo* do React inclua uma alteração que quebra o React *antes* que essa alteração quebra seja incluída em um lançamento React estável. Isso é particularmente útil porque algumas mudanças que quebram afetam apenas integrações de framework. Isso permite que um framework lance tal mudança em sua própria versão secundária sem quebrar o semver.

Lançamentos contínuos com o canal Canaries nos permitirão ter um loop de feedback mais apertado e garantir que novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho é mais proximo de como o TC39, o comitê de padrões JavaScript, [lida com as mudanças em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes de estarem em um lançamento estável do React, assim como novos recursos do JavaScript são enviados em navegadores antes de serem oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar os [lançamentos Experimentais](/community/versioning-policy#canary-channel), recomendamos que você não os use em produção, pois as APIs experimentais podem sofrer mudanças significativas que quebram no caminho para a estabilização (ou podem até ser removidos por completo). Embora os Canaries também possam conter erros (como em qualquer lançamento), daqui para frente, planejamos anunciar quaisquer mudanças significativas que quebram nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então você geralmente pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e escanear manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração supervisionada (como um framework) queiram continuar usando os lançamentos Stable.** No entanto, se você estiver criando um framework, talvez queira considerar a inclusão de uma versão Canary do React fixada em um commit específico e atualizá-la em seu próprio ritmo. A vantagen disso é que ele permite que você lance recursos e correções de bugs do React individuais e concluídos mais cedo para seus usuários e em seu próprio cronograma de lançamento, semelhante a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria a responsabilidade adicional de revisar quais commits do React estão sendo incluídos e comunicar a seus usuários quais mudanças do React estão incluídas com seus lançamentos.

Se você é um autor de um framework e deseja experimentar essa abordagem, entre em contato conosco.

## Anunciando mudanças que quebram e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento React estável a qualquer momento.

Tradicionalmente, anunciamos as mudanças que quebram somente no *final* do ciclo de lançamento (ao fazer um lançamento principal). Agora que os lançamentos Canary são uma forma oficialmente compatível de consumir o React, planejamos mudar para anunciar mudanças que quebram e novos recursos significativos *conforme eles chegam* nos Canaries. Por exemplo, se fizermos o merge de uma mudança que quebra que será lançada em um Canary, escreveremos uma postagem sobre ela no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de um framework lançando uma versão principal que atualiza o Canary React fixado para incluir essa mudança, você pode vincular nossa postagem do blog de suas notas de lançamento. Finalmente, quando uma versão principal estável do React estiver pronta, vinculará a essas postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rapidamente.

Planejamos documentar as APIs conforme elas chegam nos Canaries -- mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir mudanças que quebram.

## Exemplo: React Server Components {/*example-react-server-components*/}

Como [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos mudanças significativas que quebram relacionadas ao seu contrato de API voltado para o usuário. No entanto, não podemos lançar suporte para React Server Components em uma versão estável do React ainda porque ainda estamos trabalhando em vários recursos interligados apenas para frameworks (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperar mais mudanças que quebram lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento principal do React, a única maneira de um framework adotá-los é lançar uma versão Canary fixada do React. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que enviam com seu framework, e explicar isso a seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas com as versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores da biblioteca testem cada lançamento Canary, pois seria proibitivamente difícil. No entanto, assim como quando [introduzimos originalmente os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes em *ambas* as versões Stable e Canary mais recentes. Se você vir uma mudança no comportamento que não foi anunciada, registre um erro no repositório React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, reduza a quantidade de esforço necessário para atualizar as bibliotecas para novas versões principais do React, pois as regressões acidentais seriam encontradas conforme elas chegam.

<Note>

Falando com rigor, o Canary não é um *novo* canal de lançamento -- costumava ser chamado Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um *novo* canal de lançamento para comunicar as novas expectativas, como os Canaries serem uma forma oficialmente compatível de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis ​​do React.
``