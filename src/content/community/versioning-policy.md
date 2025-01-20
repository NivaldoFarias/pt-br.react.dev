---
title: Política de Versionamento
---

<Intro>

Todas as compilações estáveis do React passam por um alto nível de testes e seguem o versionamento semântico (semver). O React também oferece canais de lançamento instáveis para incentivar o feedback precoce sobre recursos experimentais. Esta página descreve o que você pode esperar dos lançamentos do React.

</Intro>

Para uma lista de lançamentos anteriores, consulte a página [Versões](/versions).

## Lançamentos Estáveis {/*stable-releases*/}

Lançamentos estáveis do React (também conhecidos como canal de lançamento "Mais Recente") seguem os princípios do [versionamento semântico (semver)](https://semver.org/).

Isso significa que com um número de versão **x.y.z**:

* Ao lançar **correções de bugs críticas**, fazemos um **lançamento de patch** alterando o número **z** (ex: 15.6.2 para 15.6.3).
* Ao lançar **novos recursos** ou **correções não críticas**, fazemos um **lançamento menor** alterando o número **y** (ex: 15.6.2 para 15.7.0).
* Ao lançar **mudanças quebradoras**, fazemos um **lançamento maior** alterando o número **x** (ex: 15.6.2 para 16.0.0).

Lançamentos maiores podem também conter novos recursos, e qualquer lançamento pode incluir correções de bugs.

Lançamentos menores são o tipo mais comum de lançamento.

### Mudanças Quebradoras {/*breaking-changes*/}

Mudanças quebradoras são inconvenientes para todos, então tentamos minimizar o número de lançamentos maiores – por exemplo, o React 15 foi lançado em abril de 2016, o React 16 foi lançado em setembro de 2017, e o React 17 foi lançado em outubro de 2020.

Em vez disso, lançamos novos recursos em versões menores. Isso significa que lançamentos menores são frequentemente mais interessantes e atraentes do que os maiores, apesar de seu nome modesto.

### Compromisso com a Estabilidade {/*commitment-to-stability*/}

À medida que mudamos o React ao longo do tempo, tentamos minimizar o esforço necessário para aproveitar novos recursos. Quando possível, manteremos uma API mais antiga funcionando, mesmo que isso signifique colocá-la em um pacote separado. Por exemplo, [mixins têm sido desencorajados há anos](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html), mas eles são suportados até hoje [via create-react-class](https://legacy.reactjs.org/docs/react-without-es6.html#mixins) e muitos códigos continuam a usá-los em código legado estável.

Mais de um milhão de desenvolvedores usam o React, mantendo coletivamente milhões de componentes. A base de código do Facebook sozinha tem mais de 50.000 componentes React. Isso significa que precisamos tornar o mais fácil possível atualizar para novas versões do React; se fizermos grandes mudanças sem um caminho de migração, as pessoas ficarão presas a versões antigas. Testamos esses caminhos de atualização no próprio Facebook – se nossa equipe de menos de 10 pessoas pode atualizar mais de 50.000 componentes sozinha, esperamos que a atualização seja administrável para qualquer um que use o React. Em muitos casos, escrevemos [scripts automatizados](https://github.com/reactjs/react-codemod) para atualizar a sintaxe dos componentes, que depois incluímos na versão de código aberto para que todos possam usar.

### Atualizações Gradativas via Avisos {/*gradual-upgrades-via-warnings*/}

Compilações de desenvolvimento do React incluem muitos avisos úteis. Sempre que possível, adicionamos avisos em preparação para futuras mudanças quebradoras. Dessa forma, se seu aplicativo não tiver avisos na versão mais recente, ele será compatível com o próximo lançamento maior. Isso permite que você atualize seus aplicativos um componente de cada vez.

Os avisos de desenvolvimento não afetarão o comportamento em tempo de execução do seu aplicativo. Assim, você pode ter certeza de que seu aplicativo se comportará da mesma forma entre as compilações de desenvolvimento e produção – as únicas diferenças são que a compilação de produção não registrará os avisos e que é mais eficiente. (Se você notar o contrário, por favor, registre um problema.)

### O que conta como uma mudança quebradora? {/*what-counts-as-a-breaking-change*/}

Em geral, *não* aumentamos o número da versão maior para mudanças em:

* **Avisos de desenvolvimento.** Como esses não afetam o comportamento de produção, podemos adicionar novos avisos ou modificar avisos existentes entre versões maiores. Na verdade, isso é o que nos permite avisar de forma confiável sobre mudanças quebradoras futuras.
* **APIs que começam com `unstable_`.** Essas são fornecidas como recursos experimentais cujas APIs ainda não temos confiança. Ao lançá-las com um prefixo `unstable_`, podemos iterar mais rápido e chegar a uma API estável mais cedo.
* **Versões Alpha e Canary do React.** Fornecemos versões alpha do React como uma forma de testar novos recursos cedo, mas precisamos da flexibilidade para fazer alterações com base no que aprendemos no período alpha. Se você usar essas versões, observe que as APIs podem mudar antes do lançamento estável.
* **APIs não documentadas e estruturas de dados internas.** Se você acessar nomes de propriedades internas como `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` ou `__reactInternalInstance$uk43rzhitjg`, não há garantia. Você está por conta própria.

Esta política é projetada para ser pragmática: certamente, não queremos causar dores de cabeça para você. Se aumentássemos a versão maior para todas essas mudanças, acabaríamos lançando mais versões maiores e, em última instância, causando mais dores de versionamento para a comunidade. Isso também significaria que não podemos avançar na melhoria do React tão rapidamente quanto gostaríamos.

Dito isso, se esperarmos que uma mudança nesta lista cause problemas amplos na comunidade, faremos o nosso melhor para fornecer um caminho de migração gradual.

### Se um lançamento menor não incluir novos recursos, por que não é um patch? {/*if-a-minor-release-includes-no-new-features-why-isnt-it-a-patch*/}

É possível que um lançamento menor não inclua novos recursos. [Isso é permitido pelo semver](https://semver.org/#spec-item-7), que afirma **"[uma versão menor] PODE ser incrementada se funcionalidades novas ou melhorias substanciais forem introduzidas no código privado. PODE incluir mudanças de nível de patch."**

No entanto, isso levanta a questão de por que esses lançamentos não são versionados como patches.

A resposta é que qualquer mudança no React (ou em outro software) carrega algum risco de quebrar de maneiras inesperadas. Imagine um cenário em que um lançamento de patch que corrige um bug acidentalmente introduz um bug diferente. Isso não só seria disruptivo para os desenvolvedores, mas também prejudicaria sua confiança em futuros lançamentos de patch. É especialmente lamentável se a correção original for para um bug que raramente é encontrado na prática.

Temos um bom histórico em manter os lançamentos do React livres de bugs, mas lançamentos de patch têm um padrão ainda mais alto de confiabilidade, pois a maioria dos desenvolvedores assume que podem ser adotados sem consequências adversas.

Por essas razões, reservamos lançamentos de patch apenas para os bugs mais críticos e vulnerabilidades de segurança.

Se um lançamento incluir mudanças não essenciais – como refatorações internas, alterações em detalhes de implementação, melhorias de desempenho ou correções de bugs menores – nós aumentaremos a versão menor, mesmo quando não houver novos recursos.

## Todos os Canais de Lançamento {/*all-release-channels*/}

O React depende de uma comunidade de código aberto vibrante para registrar relatórios de bugs, abrir pull requests e [submeter RFCs](https://github.com/reactjs/rfcs). Para incentivar o feedback, às vezes compartilhamos compilações especiais do React que incluem recursos não lançados.

<Note>

Esta seção será mais relevante para desenvolvedores que trabalham em frameworks, bibliotecas ou ferramentas de desenvolvimento. Desenvolvedores que usam React principalmente para construir aplicações voltadas para o usuário não devem se preocupar com nossos canais de pré-lançamento.

</Note>

Cada um dos canais de lançamento do React é projetado para um caso de uso distinto:

- [**Mais Recente**](#latest-channel) é para lançamentos estáveis e semver do React. É o que você obtém ao instalar o React via npm. Este é o canal que você já está usando hoje. **Aplicações voltadas para o usuário que consomem o React diretamente usam este canal.**
- [**Canary**](#canary-channel) acompanha o branch principal do repositório de código-fonte do React. Pense nisso como candidatos a lançamento para a próxima versão semver. **[Frameworks ou outras configurações curadas podem optar por usar este canal com uma versão fixa do React.](/blog/2023/05/03/react-canaries) Você também pode usar Canaries para testes de integração entre React e projetos de terceiros.**
- [**Experimental**](#experimental-channel) inclui APIs e recursos experimentais que não estão disponíveis nos lançamentos estáveis. Estes também acompanham o branch principal, mas com flags de recurso adicionais ativadas. Use isso para experimentar recursos futuros antes que sejam lançados.

Todos os lançamentos são publicados no npm, mas apenas o Mais Recente utiliza o versionamento semântico. Pré-lançamentos (aqueles nos canais Canary e Experimental) têm versões geradas a partir de um hash de seu conteúdo e da data de commit, por exemplo, `18.3.0-canary-388686f29-20230503` para Canary e `0.0.0-experimental-388686f29-20230503` para Experimental.

**Tanto os canais Mais Recente quanto Canary são oficialmente suportados para aplicações voltadas para o usuário, mas com expectativas diferentes**:

* Lançamentos Mais Recentes seguem o modelo tradicional de semver.
* Lançamentos Canary [devem ser fixados](/blog/2023/05/03/react-canaries) e podem incluir mudanças quebradoras. Eles existem para configurações curadas (como frameworks) que desejam lançar gradualmente novos recursos e correções de bugs do React em seu próprio cronograma de lançamentos.

Os lançamentos Experimentais são fornecidos apenas para fins de teste, e não oferecemos garantias de que o comportamento não mudará entre os lançamentos. Eles não seguem o protocolo semver que usamos para os lançamentos do Mais Recente.

Ao publicar pré-lançamentos no mesmo registro que usamos para lançamentos estáveis, conseguimos aproveitar as muitas ferramentas que suportam o fluxo de trabalho do npm, como [unpkg](https://unpkg.com) e [CodeSandbox](https://codesandbox.io).

### Canal Mais Recente {/*latest-channel*/}

O Canal Mais Recente é o canal usado para lançamentos estáveis do React. Ele corresponde ao tag `latest` no npm. É o canal recomendado para todos os aplicativos React que são enviados para usuários reais.

**Se você não tem certeza de qual canal deve usar, é o Mais Recente.** Se você estiver usando o React diretamente, é isso que você já está usando. Você pode esperar que as atualizações para o Mais Recente sejam extremamente estáveis. As versões seguem o esquema de versionamento semântico, como [descrito anteriormente.](#stable-releases)

### Canal Canary {/*canary-channel*/}

O canal Canary é um canal de pré-lançamento que acompanha o branch principal do repositório do React. Usamos pré-lançamentos no canal Canary como candidatos a lançamentos para o canal Mais Recente. Você pode pensar no Canary como um superconjunto do Mais Recente que é atualizado com mais frequência.

O grau de mudança entre o lançamento mais recente do Canary e o lançamento Mais Recente mais recente é aproximadamente o mesmo que você encontraria entre dois lançamentos menores semver. No entanto, **o canal Canary não está em conformidade com o versionamento semântico.** Você deve esperar mudanças quebradoras ocasionais entre lançamentos sucessivos no canal Canary.

**Não use pré-lançamentos em aplicações voltadas para o usuário diretamente, a menos que você esteja seguindo o [fluxo de trabalho Canary](/blog/2023/05/03/react-canaries).**

Os lançamentos no Canary são publicados com a tag `canary` no npm. As versões são geradas a partir de um hash do conteúdo da compilação e da data de commit, por exemplo, `18.3.0-canary-388686f29-20230503`.

#### Usando o canal canary para testes de integração {/*using-the-canary-channel-for-integration-testing*/}

O canal Canary também suporta testes de integração entre o React e outros projetos.

Todas as mudanças no React passam por extensos testes internos antes de serem lançadas ao público. No entanto, há uma miríade de ambientes e configurações usados em todo o ecossistema React, e não é possível para nós testar todos eles.

Se você é o autor de um framework, biblioteca, ferramenta de desenvolvedor de terceiros ou projeto de infraestrutura semelhante, pode nos ajudar a manter o React estável para seus usuários e para toda a comunidade React executando periodicamente seu conjunto de testes contra as mudanças mais recentes. Se você estiver interessado, siga estas etapas:

- Configure um cron job usando sua plataforma de integração contínua preferida. Cron jobs são suportados tanto pelo [CircleCI](https://circleci.com/docs/2.0/triggers/#scheduled-builds) quanto pelo [Travis CI](https://docs.travis-ci.com/user/cron-jobs/).
- No cron job, atualize seus pacotes do React para o mais recente lançamento do React no canal Canary, usando a tag `canary` no npm. Usando a CLI do npm:

  ```console
  npm update react@canary react-dom@canary
  ```

  Ou yarn:

  ```console
  yarn upgrade react@canary react-dom@canary
  ```
- Execute seu conjunto de testes contra os pacotes atualizados.
- Se tudo passar, ótimo! Você pode esperar que seu projeto funcione com o próximo lançamento menor do React.
- Se algo quebrar inesperadamente, por favor, nos avise registrando um [problema](https://github.com/facebook/react/issues).

Um projeto que usa esse fluxo de trabalho é o Next.js. Você pode se referir à [configuração do CircleCI deles](https://github.com/zeit/next.js/blob/c0a1c0f93966fe33edd93fb53e5fafb0dcd80a9e/.circleci/config.yml) como exemplo.

### Canal Experimental {/*experimental-channel*/}

Assim como o Canary, o canal Experimental é um canal de pré-lançamento que acompanha o branch principal do repositório do React. Diferentemente do Canary, os lançamentos Experimentais incluem recursos e APIs adicionais que não estão prontos para um lançamento mais amplo.

Geralmente, uma atualização para o Canary é acompanhada por uma atualização correspondente para o Experimental. Elas são baseadas na mesma revisão de código-fonte, mas são construídas usando um conjunto diferente de flags de recurso.

Lançamentos Experimentais podem ser significativamente diferentes dos lançamentos do Canary e do Mais Recente. **Não use lançamentos Experimentais em aplicações voltadas para o usuário.** Você deve esperar mudanças quebradoras frequentes entre lançamentos no canal Experimental.

Os lançamentos no Experimental são publicados com a tag `experimental` no npm. As versões são geradas a partir de um hash do conteúdo da compilação e da data de commit, por exemplo, `0.0.0-experimental-68053d940-20210623`.

#### O que entra em um lançamento experimental? {/*what-goes-into-an-experimental-release*/}

Recursos experimentais são aqueles que não estão prontos para serem lançados ao público mais amplo e podem mudar drasticamente antes de serem finalizados. Alguns experimentos podem nunca ser finalizados – a razão pela qual temos experimentos é testar a viabilidade das mudanças propostas.

Por exemplo, se o canal Experimental tivesse existido quando anunciamos Hooks, teríamos lançado Hooks no canal Experimental semanas antes de estarem disponíveis no Mais Recente.

Você pode achar valioso executar testes de integração contra os Experimentais. Isso fica a seu critério. No entanto, esteja avisado de que o Experimental é ainda menos estável que o Canary. **Não garantimos nenhuma estabilidade entre lançamentos Experimentais.**

#### Como posso aprender mais sobre recursos experimentais? {/*how-can-i-learn-more-about-experimental-features*/}

Recursos experimentais podem ou não ser documentados. Geralmente, os experimentos não são documentados até que estejam próximos de serem enviados no Canary ou no Mais Recente.

Se um recurso não estiver documentado, ele pode estar acompanhado de um [RFC](https://github.com/reactjs/rfcs).

Postaremos no [blog do React](/blog) quando estivermos prontos para anunciar novos experimentos, mas isso não significa que iremos publicizar todos os experimentos.

Você sempre pode consultar o [histórico](https://github.com/facebook/react/commits/main) do nosso repositório público no GitHub para uma lista abrangente de mudanças.