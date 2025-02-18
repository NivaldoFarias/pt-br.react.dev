May 3, 2023 por [Dan Abramov](https://bsky.app/profile/danabra.mov), [Sophie Alpert](https://twitter.com/sophiebits), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

Gostaríamos de oferecer à comunidade React uma opção para adotar recursos individuais novos assim que seu design estiver próximo do final, antes que sejam lançados em uma versão estável -- semelhante a como a Meta usa há muito tempo versões de ponta do React internamente. Estamos introduzindo um novo [canal de lançamento Canary](/community/versioning-policy#canary-channel) com suporte oficial. Ele permite que configurações selecionadas, como frameworks, desacoplem a adoção de recursos individuais do React do cronograma de lançamento do React.

</Intro>

---

## tl;dr {/*tldr*/}

* Estamos introduzindo um [canal de lançamento Canary](/community/versioning-policy#canary-channel) com suporte oficial para o React. Como ele é oficialmente suportado, se houver alguma regressão, trataremos com uma urgência semelhante a erros em lançamentos estáveis.
* Canaries permitem que você comece a usar novos recursos individuais do React antes que eles cheguem aos lançamentos estáveis de semver.
* Ao contrário do canal [Experimental](/community/versioning-policy#experimental-channel), React Canaries só inclui recursos que acreditamos razoavelmente estarem prontos para adoção. Incentivamos os frameworks a considerar a inclusão de lançamentos Canary do React fixos.
* Anunciaremos alterações interruptivas e novos recursos em nosso blog assim que chegarem aos lançamentos Canary.
* **Como sempre, o React continua a seguir o semver para cada lançamento Stable.**

## Como os recursos do React são normalmente desenvolvidos {/*how-react-features-are-usually-developed*/}

Normalmente, cada recurso do React passou pelos mesmos estágios:

1. Desenvolvemos uma versão inicial e a prefixamos com `experimental_` ou `unstable_`. O recurso está disponível apenas no canal de lançamento `experimental`. Neste ponto, espera-se que o recurso mude significativamente.
2. Encontramos uma equipe na Meta disposta a nos ajudar a testar este recurso e fornecer feedback sobre ele. Isso leva a uma rodada de mudanças. À medida que o recurso se torna mais estável, trabalhamos com mais equipes da Meta para experimentá-lo.
3. Eventualmente, nos sentimos confiantes no design. Removemos o prefixo do nome da API e tornamos o recurso disponível no branch `main` por padrão, que a maioria dos produtos da Meta usa. Neste ponto, qualquer equipe da Meta pode usar este recurso.
4. À medida que construímos confiança na direção, também postamos um RFC para o novo recurso. Neste ponto, sabemos que o design funciona para um amplo conjunto de casos, mas podemos fazer alguns ajustes de última hora.
5. Quando estamos perto de lançar uma versão de código aberto, escrevemos a documentação para o recurso e finalmente lançamos o recurso em um lançamento estável do React.

Este manual de instruções funciona bem para a maioria dos recursos que lançamos até agora. No entanto, pode haver uma lacuna significativa entre quando o recurso está geralmente pronto para uso (etapa 3) e quando ele é lançado em código aberto (etapa 5).

**Gostaríamos de oferecer à comunidade React uma opção para seguir a mesma abordagem da Meta e adotar novos recursos individuais mais cedo (assim que estiverem disponíveis) sem ter que esperar pelo próximo ciclo de lançamento do React.**

Como sempre, todos os recursos do React eventualmente chegarão a um lançamento Stable.

## Podemos simplesmente fazer mais lançamentos menores? {/*can-we-just-do-more-minor-releases*/}

Geralmente, *usamos* lançamentos menores para introduzir novos recursos.

No entanto, isso nem sempre é possível. Às vezes, novos recursos estão interligados com *outros* novos recursos que ainda não foram totalmente concluídos e nos quais ainda estamos iterando ativamente. Não podemos lançá-los separadamente porque suas implementações estão relacionadas. Não podemos versioná-los separadamente porque eles afetam os mesmos pacotes (por exemplo, `react` e `react-dom`). E precisamos manter a capacidade de iterar nas peças que não estão prontas sem uma enxurrada de lançamentos de versão principal, o que o semver exigiria que fizéssemos.

Na Meta, resolvemos esse problema construindo o React a partir do branch `main` e atualizando-o manualmente para um commit específico fixo toda semana. Essa também é a abordagem que os lançamentos do React Native têm seguido nos últimos anos. Cada lançamento *estável* do React Native é fixado em um commit específico do branch `main` do repositório do React. Isso permite que o React Native inclua correções de erros importantes e adote incrementalmente novos recursos do React no nível do framework, sem ser acoplado ao cronograma global de lançamentos do React.

Gostaríamos de disponibilizar este fluxo de trabalho para outros frameworks e configurações selecionadas. Por exemplo, ele permite que um framework *sobre* o React inclua uma alteração interruptiva relacionada ao React *antes* que essa alteração interruptiva seja incluída em um lançamento estável do React. Isso é particularmente útil porque algumas alterações interruptivas afetam apenas integrações de framework. Isso permite que um framework lance tal alteração em sua própria versão secundária sem quebrar o semver.

Os lançamentos contínuos com o canal Canaries nos permitirão ter um ciclo de feedback mais apertado e garantir que os novos recursos recebam testes abrangentes na comunidade. Este fluxo de trabalho está mais próximo de como o TC39, o comitê de padrões JavaScript, [gerencia alterações em estágios numerados](https://tc39.es/process-document/). Novos recursos do React podem estar disponíveis em frameworks construídos no React antes que estejam em um lançamento estável do React, assim como novos recursos do JavaScript são lançados em navegadores antes que sejam oficialmente ratificados como parte da especificação.

## Por que não usar lançamentos experimentais em vez disso? {/*why-not-use-experimental-releases-instead*/}

Embora você *possa* tecnicamente usar os [lançamentos Experimental](/community/versioning-policy#canary-channel), recomendamos não usá-los em produção porque as APIs experimentais podem sofrer alterações interruptivas significativas a caminho da estabilização (ou podem até ser removidas inteiramente). Embora os Canaries também possam conter erros (como em qualquer lançamento), no futuro planejamos anunciar quaisquer alterações interruptivas significativas nos Canaries em nosso blog. Os Canaries são os mais próximos do código que a Meta executa internamente, então você geralmente pode esperar que eles sejam relativamente estáveis. No entanto, você *precisa* manter a versão fixada e verificar manualmente o log de commits do GitHub ao atualizar entre os commits fixados.

**Esperamos que a maioria das pessoas que usam o React fora de uma configuração selecionada (como um framework) queira continuar usando os lançamentos Stable.** No entanto, se você estiver construindo um framework, convém considerar a inclusão de uma versão Canary do React fixada em um commit específico e atualizá-la no seu próprio ritmo. A vantagem disso é que permite que você envie recursos individuais concluídos e correções de bugs do React mais cedo para seus usuários e em seu próprio cronograma de lançamento, de forma semelhante a como o React Native tem feito nos últimos anos. A desvantagem é que você assumiria responsabilidades adicionais para revisar quais commits do React estão sendo puxados e comunicar a seus usuários quais alterações do React estão incluídas em seus lançamentos.

Se você é um autor de framework e deseja experimentar essa abordagem, entre em contato conosco.

## Anunciando mudanças interruptivas e novos recursos antecipadamente {/*announcing-breaking-changes-and-new-features-early*/}

Os lançamentos Canary representam nossa melhor estimativa do que entrará no próximo lançamento estável do React em um determinado momento.

Tradicionalmente, só anunciamos alterações interruptivas no *final* do ciclo de lançamento (ao fazer um lançamento major). Agora que os lançamentos Canary são uma forma oficialmente suportada de consumir o React, planejamos mudar para o anúncio de alterações interruptivas e novos recursos significativos *à medida que chegam* aos Canaries. Por exemplo, se mesclarmos uma alteração interruptiva que será lançada em um Canary, escreveremos uma postagem sobre isso no blog do React, incluindo codemods e instruções de migração, se necessário. Então, se você é um autor de framework que está lançando uma versão major que atualiza o canary React fixo para incluir essa alteração, você pode vincular à postagem do nosso blog de suas notas de lançamento. Finalmente, quando uma versão major estável do React estiver pronta, vincularemos a essas postagens de blog já publicadas, o que esperamos ajudar nossa equipe a progredir mais rapidamente.

Planejamos documentar as APIs à medida que chegam aos Canaries -- mesmo que essas APIs ainda não estejam disponíveis fora deles. As APIs que estão disponíveis apenas nos Canaries serão marcadas com uma nota especial nas páginas correspondentes. Isso incluirá APIs como [`use`](https://github.com/reactjs/rfcs/pull/229) e algumas outras (como `cache` e `createServerContext`) para as quais enviaremos RFCs.

## Canaries devem ser fixados {/*canaries-must-be-pinned*/}

Se você decidir adotar o fluxo de trabalho Canary para seu aplicativo ou framework, certifique-se de sempre fixar a versão *exata* do Canary que você está usando. Como os Canaries são pré-lançamentos, eles ainda podem incluir alterações interruptivas.

## Exemplo: React Server Components {/*example-react-server-components*/}

Conforme [anunciamos em março](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), as convenções do React Server Components foram finalizadas e não esperamos alterações interruptivas significativas relacionadas ao contrato de API voltado para o usuário. No entanto, ainda não podemos lançar suporte para React Server Components em uma versão estável do React porque ainda estamos trabalhando em vários recursos interligados apenas para framework (como [carregamento de ativos](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)) e esperamos mais alterações interruptivas lá.

Isso significa que os React Server Components estão prontos para serem adotados por frameworks. No entanto, até o próximo lançamento major do React, a única maneira de um framework adotá-los é enviar uma versão Canary do React fixa. (Para evitar a inclusão de duas cópias do React, os frameworks que desejam fazer isso precisariam impor a resolução de `react` e `react-dom` para o Canary fixado que eles enviam com seu framework e explicar isso a seus usuários. Como exemplo, é isso que o Next.js App Router faz.)

## Testando bibliotecas em versões Stable e Canary {/*testing-libraries-against-both-stable-and-canary-versions*/}

Não esperamos que os autores de bibliotecas testem cada lançamento Canary, pois isso seria demasiadamente difícil. No entanto, assim como quando [originalmente introduzimos os diferentes canais de pré-lançamento do React há três anos](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html), incentivamos as bibliotecas a executar testes em *ambas* as versões Stable e Canary mais recentes. Se você vir uma alteração no comportamento que não foi anunciada, registre um bug no repositório do React para que possamos ajudar a diagnosticá-lo. Esperamos que, à medida que essa prática se torne amplamente adotada, reduza a quantidade de esforço necessária para atualizar bibliotecas para novas versões major do React, uma vez que as regressões acidentais seriam encontradas à medida que chegam.

<Note>

Falando estritamente, Canary não é um canal de lançamento *novo* -- costumava ser chamado Next. No entanto, decidimos renomeá-lo para evitar confusão com o Next.js. Estamos anunciando-o como um canal de lançamento *novo* para comunicar as novas expectativas, como os Canaries sendo uma forma oficialmente suportada de usar o React.

</Note>

## Lançamentos Stable funcionam como antes {/*stable-releases-work-like-before*/}

Não estamos introduzindo nenhuma alteração nos lançamentos estáveis do React.