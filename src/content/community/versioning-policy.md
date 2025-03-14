---
title: Política de Versionamento
---

<Intro>

Todos as versões estáveis do React passam por um alto nível de testes e seguem o versionamento semântico (semver). O React também oferece canais de lançamento instáveis para incentivar feedback inicial sobre recursos experimentais. Esta página descreve o que você pode esperar dos lançamentos do React.

</Intro>

Esta política de versionamento descreve nossa abordagem para números de versão de pacotes como `react` e `react-dom`. Para uma lista de lançamentos anteriores, consulte a página [Versions](/versions).

## Lançamentos estáveis {/*stable-releases*/}

Os lançamentos estáveis do React (também conhecidos como canal de lançamento "Latest") seguem os princípios de [versionamento semântico (semver)](https://semver.org/).

Isso significa que com um número de versão **x.y.z**:

* Ao lançar **correções de bugs críticas**, fazemos um **lançamento de patch** alterando o número **z** (ex: 15.6.2 para 15.6.3).
* Ao lançar **novos recursos** ou **correções não críticas**, fazemos um **lançamento secundário** alterando o número **y** (ex: 15.6.2 para 15.7.0).
* Ao lançar **mudanças de última hora**, fazemos um **lançamento principal** alterando o número **x** (ex: 15.6.2 para 16.0.0).

Os lançamentos principais também podem conter novos recursos, e qualquer lançamento pode incluir correções de bugs.

Lançamentos secundários são o tipo mais comum de lançamento.

Sabemos que nossos usuários continuam usando versões antigas do React em produção. Se soubermos de uma vulnerabilidade de segurança no React, lançamos uma correção retroportada para todas as versões principais afetadas pela vulnerabilidade.

### Mudanças de última hora {/*breaking-changes*/}

Mudanças de última hora são inconvenientes para todos, por isso tentamos minimizar o número de lançamentos principais - por exemplo, o React 15 foi lançado em abril de 2016 e o React 16 foi lançado em setembro de 2017, e o React 17 foi lançado em outubro de 2020.

Em vez disso, lançamos novos recursos em versões secundárias. Isso significa que os lançamentos secundários são frequentemente mais interessantes e atraentes do que os principais, apesar de seu nome modesto.

### Compromisso com a estabilidade {/*commitment-to-stability*/}

À medida que alteramos o React ao longo do tempo, tentamos minimizar o esforço necessário para aproveitar novos recursos. Sempre que possível, manteremos uma API mais antiga funcionando, mesmo que isso signifique colocá-la em um pacote separado. Por exemplo, [mixins foram desencorajados por anos](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html), mas são suportados até hoje [via create-react-class](https://legacy.reactjs.org/docs/react-without-es6.html#mixins) e muitas bases de código continuam a usá-los em código estável e herdado.

Mais de um milhão de desenvolvedores usam o React, mantendo coletivamente milhões de componentes. A base de código do Facebook sozinha tem mais de 50.000 componentes React. Isso significa que precisamos tornar o mais fácil possível a atualização para novas versões do React. Se fizermos grandes alterações sem um caminho de migração, as pessoas ficarão presas em versões antigas. Testamos esses caminhos de atualização no próprio Facebook - se nossa equipe de menos de 10 pessoas pode atualizar mais de 50.000 componentes sozinhas, esperamos que a atualização seja gerenciável para qualquer pessoa que use o React. Em muitos casos, escrevemos [scripts automatizados](https://github.com/reactjs/react-codemod) para atualizar a sintaxe do componente, que incluímos no lançamento de código aberto para todos usarem.

### Atualizações graduais via avisos {/*gradual-upgrades-via-warnings*/}

As versões de desenvolvimento do React incluem muitos avisos úteis. Sempre que possível, adicionamos avisos em preparação para futuras mudanças de última hora. Dessa forma, se seu aplicativo não tiver avisos na versão mais recente, ele será compatível com a próxima versão principal. Isso permite que você atualize seus aplicativos um componente por vez.

Os avisos de desenvolvimento não afetarão o comportamento de tempo de execução do seu aplicativo. Dessa forma, você pode ter certeza de que seu aplicativo se comportará da mesma maneira entre as versões de desenvolvimento e produção - as únicas diferenças são que a versão de produção não registrará os avisos e será mais eficiente. (Se você notar o contrário, por favor, [abra um issue](https://github.com/facebook/react/issues).)

### O que conta como uma mudança de última hora? {/*what-counts-as-a-breaking-change*/}

Em geral, *não* aumentamos o número da versão principal para alterações em:

*   **Avisos de desenvolvimento.** Como eles não afetam o comportamento de produção, podemos adicionar novos avisos ou modificar os avisos existentes entre as versões principais. Na verdade, é isso que nos permite avisar com segurança sobre as próximas mudanças de última hora.
*   **APIs começando com `unstable_`.** Estes são fornecidos como recursos experimentais cujas APIs ainda não temos confiança. Ao lançá-los com um prefixo `unstable_`, podemos iterar mais rapidamente e obter uma API estável mais cedo.
*   **Versões Alpha e Canary do React.** Fornecemos versões alfa do React como uma forma de testar novos recursos antecipadamente, mas precisamos da flexibilidade para fazer alterações com base no que aprendemos no período alfa. Se você usar essas versões, observe que as APIs podem mudar antes do lançamento estável.
*   **APIs não documentadas e estruturas de dados internas.** Se você acessar nomes de propriedades internas como `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` ou `__reactInternalInstance$uk43rzhitjg`, não há garantia. Você está por sua conta.

Esta política foi projetada para ser pragmática: certamente, não queremos causar dores de cabeça para você. Se aumentássemos a versão principal para todas essas alterações, acabaríamos lançando mais versões principais e, em última análise, causando mais problemas de versionamento para a comunidade. Isso também significaria que não podemos progredir na melhoria do React tão rápido quanto gostaríamos.

Dito isso, se esperamos que uma alteração nesta lista cause problemas generalizados na comunidade, ainda faremos o possível para fornecer um caminho de migração gradual.

### Se um lançamento secundário não incluir novos recursos, por que ele não é um patch? {/*if-a-minor-release-includes-no-new-features-why-isnt-it-a-patch*/}

É possível que um lançamento secundário não inclua novos recursos. [Isso é permitido por semver](https://semver.org/#spec-item-7), que afirma que **"[uma versão secundária] PODE ser incrementada se uma nova funcionalidade substancial ou melhorias forem introduzidas no código privado. PODE incluir alterações no nível do patch."**

No entanto, isso levanta a questão de por que esses lançamentos não são versionados como patches.

A resposta é que qualquer alteração no React (ou outro software) acarreta algum risco de quebra de maneiras inesperadas. Imagine um cenário em que um lançamento de patch que corrige um erro introduza acidentalmente um erro diferente. Isso não seria apenas disruptivo para os desenvolvedores, mas também prejudicaria sua confiança em futuros lançamentos de patch. É especialmente lamentável se a correção original for para um erro que raramente é encontrado na prática.

Temos um bom histórico de manter os lançamentos do React livres de erros, mas os lançamentos de patch têm um padrão ainda maior de confiabilidade porque a maioria dos desenvolvedores assume que eles podem ser adotados sem consequências adversas.

Por essas razões, reservamos os lançamentos de patch apenas para os bugs e vulnerabilidades de segurança mais críticos.

Se um lançamento incluir alterações não essenciais - como refatorações internas, alterações nos detalhes de implementação, melhorias de desempenho ou pequenas correções de bugs - aumentaremos a versão secundária, mesmo quando não houver novos recursos.

## Todos os canais de lançamento {/*all-release-channels*/}

O React conta com uma próspera comunidade de código aberto para registrar relatórios de erros, abrir pull requests e [enviar RFCs](https://github.com/reactjs/rfcs). Para incentivar o feedback, às vezes compartilhamos versões especiais do React que incluem recursos não lançados.

<Note>

Esta seção será mais relevante para desenvolvedores que trabalham em frameworks, bibliotecas ou ferramentas de desenvolvedor. Desenvolvedores que usam o React principalmente para construir aplicativos voltados para o usuário não devem se preocupar com nossos canais de pré-lançamento.

</Note>

Cada um dos canais de lançamento do React é projetado para um caso de uso distinto:

-   [**Latest**](#latest-channel) é para lançamentos estáveis do React semver. É o que você obtém ao instalar o React do npm. Este é o canal que você já está usando hoje. **Aplicativos voltados para o usuário que consomem React diretamente usam este canal.**
-   [**Canary**](#canary-channel) rastreia a branch principal do repositório de código-fonte do React. Pense nisso como candidatos a lançamento para a próxima versão semver. **[Frameworks ou outras configurações selecionadas podem optar por usar este canal com uma versão fixa do React.](/blog/2023/05/03/react-canaries) Você também pode usar Canaries para testes de integração entre React e projetos de terceiros.**
-   [**Experimental**](#experimental-channel) inclui APIs e recursos experimentais que não estão disponíveis nas versões estáveis. Estes também rastreiam a branch principal, mas com sinalizadores de recursos adicionais ativados. Use-o para experimentar os próximos recursos antes que eles sejam lançados.

Todos os lançamentos são publicados no npm, mas apenas o Latest usa o versionamento semântico. Os pré-lançamentos (aqueles nos canais Canary e Experimental) têm versões geradas a partir de um hash de seus conteúdos e da data do commit, por exemplo, `18.3.0-canary-388686f29-20230503` para Canary e `0.0.0-experimental-388686f29-20230503` para Experimental.

**Os canais Latest e Canary são oficialmente suportados para aplicativos voltados para o usuário, mas com expectativas diferentes**:

*   Os lançamentos mais recentes seguem o modelo semver tradicional.
*   Os lançamentos Canary [devem ser fixados](/blog/2023/05/03/react-canaries) e podem incluir mudanças de última hora. Eles existem para configurações selecionadas (como frameworks) que desejam lançar gradualmente novos recursos e correções de bugs do React em sua própria programação de lançamento.

Os lançamentos experimentais são fornecidos apenas para fins de teste e não fornecemos nenhuma garantia de que o comportamento não mudará entre os lançamentos. Eles não seguem o protocolo semver que usamos para lançamentos do Latest.

Ao publicar pré-lançamentos no mesmo registro que usamos para lançamentos estáveis, podemos aproveitar as muitas ferramentas que suportam o fluxo de trabalho npm, como [unpkg](https://unpkg.com) e [CodeSandbox](https://codesandbox.io).

### Canal mais recente {/*latest-channel*/}

Latest é o canal usado para lançamentos estáveis do React. Ele corresponde à tag `latest` no npm. É o canal recomendado para todos os aplicativos React que são enviados para usuários reais.

**Se você não tiver certeza de qual canal deve usar, é Latest.** Se você estiver usando o React diretamente, é isso que você já está usando. Você pode esperar que as atualizações para Latest sejam extremamente estáveis. As versões seguem o esquema de versionamento semântico, conforme [descrito anteriormente](#stable-releases).

### Canal Canary {/*canary-channel*/}

O canal Canary é um canal de pré-lançamento que rastreia a branch principal do repositório do React. Usamos pré-lançamentos no canal Canary como candidatos a lançamento para o canal Latest. Você pode pensar no Canary como um superconjunto do Latest que é atualizado com mais frequência.

O grau de mudança entre a versão Canary mais recente e a versão Latest mais recente é aproximadamente o mesmo que você encontraria entre duas versões secundárias semver. No entanto, **o canal Canary não está em conformidade com o versionamento semântico.** Você deve esperar mudanças de última hora ocasionais entre os lançamentos sucessivos no canal Canary.

**Não use pré-lançamentos em aplicativos voltados para o usuário diretamente, a menos que você esteja seguindo o [fluxo de trabalho Canary](/blog/2023/05/03/react-canaries).**

Os lançamentos no Canary são publicados com a tag `canary` no npm. As versões são geradas a partir de um hash do conteúdo da compilação e da data do commit, por exemplo, `18.3.0-canary-388686f29-20230503`.

#### Usando o canal canary para testes de integração {/*using-the-canary-channel-for-integration-testing*/}

O canal Canary também suporta testes de integração entre React e outros projetos.

Todas as alterações no React passam por extensos testes internos antes de serem lançadas ao público. No entanto, há uma miríade de ambientes e configurações usadas em todo o ecossistema React, e não é possível testar contra todos eles.

Se for o autor de um framework React de terceiros, biblioteca, ferramenta de desenvolvedor ou projeto de tipo de infraestrutura semelhante, você pode nos ajudar a manter o React estável para seus usuários e toda a comunidade React, executando periodicamente seu conjunto de testes em relação às alterações mais recentes. Se você estiver interessado, siga estas etapas:

*   Configure um job cron usando sua plataforma de integração contínua preferida. Os jobs cron são suportados por [CircleCI](https://circleci.com/docs/2.0/triggers/#scheduled-builds) e [Travis CI](https://docs.travis-ci.com/user/cron-jobs/).
*   No job cron, atualize seus pacotes React para a versão React mais recente no canal Canary, usando a tag `canary` no npm. Usando o cli npm:

    ```console
    npm update react@canary react-dom@canary
    ```

    Ou yarn:

    ```console
    yarn upgrade react@canary react-dom@canary
    ```
*   Execute seu conjunto de testes em relação aos pacotes atualizados.
*   Se tudo passar, ótimo! Você pode esperar que seu projeto funcione com a próxima versão secundária do React.
*   Se algo quebrar inesperadamente, por favor, nos avise [abrindo um issue](https://github.com/facebook/react/issues).

Um projeto que usa este fluxo de trabalho é o Next.js. Você pode consultar sua [configuração do CircleCI](https://github.com/zeit/next.js/blob/c0a1c0f93966fe33edd93fb53e5fafb0dcd80a9e/.circleci/config.yml) como um exemplo.

### Canal Experimental {/*experimental-channel*/}

Como o Canary, o canal Experimental é um canal de pré-lançamento que rastreia a branch principal do repositório do React. Ao contrário do Canary, os lançamentos experimentais incluem recursos e APIs adicionais que não estão prontos para um lançamento mais amplo.

Normalmente, uma atualização para Canary é acompanhada por uma atualização correspondente para Experimental. Eles são baseados na mesma revisão de origem, mas são construídos usando um conjunto diferente de sinalizadores de recursos.

Os lançamentos experimentais podem ser significativamente diferentes dos lançamentos para Canary e Latest. **Não use lançamentos experimentais em aplicativos voltados para o usuário.** Você deve esperar mudanças frequentes e de última hora entre os lançamentos no canal Experimental.

Os lançamentos em Experimental são publicados com a tag `experimental` no npm. As versões são geradas a partir de um hash do conteúdo da compilação e da data do commit, por exemplo, `0.0.0-experimental-68053d940-20210623`.

#### O que entra em um lançamento experimental? {/*what-goes-into-an-experimental-release*/}

Recursos experimentais são aqueles que não estão prontos para serem lançados ao público em geral e podem mudar drasticamente antes de serem finalizados. Alguns experimentos podem nunca ser finalizados - a razão pela qual temos experimentos é testar a viabilidade das alterações propostas.

Por exemplo, se o canal Experimental tivesse existido quando anunciamos Hooks, teríamos lançado Hooks no canal Experimental semanas antes que estivessem disponíveis no Latest.

Você pode achar valioso executar testes de integração em Experimental. Isso é com você. No entanto, esteja ciente de que Experimental é ainda menos estável do que Canary. **Não garantimos nenhuma estabilidade entre os lançamentos experimentais.**

#### Como posso aprender mais sobre os recursos experimentais? {/*how-can-i-learn-more-about-experimental-features*/}

Recursos experimentais podem ou não ser documentados. Normalmente, os experimentos não são documentados até que estejam próximos do envio em Canary ou Latest.

Se um recurso não estiver documentado, eles podem ser acompanhados por um [RFC](https://github.com/reactjs/rfcs).

Publicaremos no [blog do React](/blog) quando estivermos prontos para anunciar novos experimentos, mas isso não significa que divulgaremos todos os experimentos.

Você sempre pode consultar o [histórico](https://github.com/facebook/react/commits/main) do nosso repositório público do GitHub para obter uma lista abrangente de alterações.
```