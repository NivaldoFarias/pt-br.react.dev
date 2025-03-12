---
title: Política de Versionamento
---

<Intro>

Todas as compilações estáveis do React passam por um alto nível de teste e seguem o versionamento semântico (semver). O React também oferece canais de lançamento instáveis para incentivar o feedback inicial sobre recursos experimentais. Esta página descreve o que você pode esperar dos lançamentos do React.

</Intro>

Esta política de versionamento descreve nossa abordagem para os números de versão de pacotes como `react` e `react-dom`. Para uma lista de lançamentos anteriores, consulte a página [Versões](/versions).

## Lançamentos estáveis {/*stable-releases*/}

Os lançamentos estáveis do React (também conhecidos como canal de lançamento "Mais Recente") seguem os princípios de [versionamento semântico (semver)](https://semver.org/).

Isso significa que com um número de versão **x.y.z**:

* Ao lançar **correções de bugs críticas**, fazemos um **lançamento de patch** alterando o número **z** (ex: 15.6.2 para 15.6.3).
* Ao lançar **novos recursos** ou **correções não críticas**, fazemos um **lançamento secundário** alterando o número **y** (ex: 15.6.2 para 15.7.0).
* Ao lançar **mudanças radicais**, fazemos um **lançamento principal** alterando o número **x** (ex: 15.6.2 para 16.0.0).

Os lançamentos principais também podem conter novos recursos, e qualquer lançamento pode incluir correções de bugs.

Os lançamentos secundários são o tipo de lançamento mais comum.

Sabemos que nossos usuários continuam a usar versões antigas do React em produção. Se soubermos de uma vulnerabilidade de segurança no React, lançamos uma correção retroativa para todas as versões principais afetadas pela vulnerabilidade.

### Mudanças radicais {/*breaking-changes*/}

As mudanças radicais são inconvenientes para todos, então tentamos minimizar o número de lançamentos principais - por exemplo, o React 15 foi lançado em abril de 2016 e o React 16 foi lançado em setembro de 2017, e o React 17 foi lançado em outubro de 2020.

Em vez disso, lançamos novos recursos em versões secundárias. Isso significa que os lançamentos secundários são frequentemente mais interessantes e atraentes do que os principais, apesar de seu nome modesto.

### Compromisso com a estabilidade {/*commitment-to-stability*/}

À medida que modificamos o React ao longo do tempo, tentamos minimizar o esforço necessário para aproveitar os novos recursos. Sempre que possível, manteremos uma API antiga funcionando, mesmo que isso signifique colocá-la em um pacote separado. Por exemplo, [mixins foram desencorajados por anos](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html), mas eles são suportados até hoje [ via create-react-class](https://legacy.reactjs.org/docs/react-without-es6.html#mixins) e muitos códigos bases continuam a usá-los em código estável e herdado.

Mais de um milhão de desenvolvedores usam o React, mantendo coletivamente milhões de componentes. O código base do Facebook sozinho tem mais de 50.000 componentes React. Isso significa que precisamos tornar o mais fácil possível a atualização para novas versões do React; se fizermos grandes alterações sem um caminho de migração, as pessoas ficarão presas em versões antigas. Testamos esses caminhos de atualização no próprio Facebook - se nossa equipe de menos de 10 pessoas pode atualizar mais de 50.000 componentes sozinhas, esperamos que a atualização seja gerenciável para qualquer pessoa que use o React. Em muitos casos, escrevemos [scripts automatizados](https://github.com/reactjs/react-codemod) para atualizar a sintaxe do componente, que então incluímos no lançamento de código aberto para que todos usem.

### Atualizações graduais por meio de avisos {/*gradual-upgrades-via-warnings*/}

As compilações de desenvolvimento do React incluem muitos avisos úteis. Sempre que possível, adicionamos avisos em preparação para futuras mudanças radicais. Dessa forma, se seu aplicativo não tiver nenhum aviso no lançamento mais recente, ele será compatível com o próximo lançamento principal. Isso permite que você atualize seus aplicativos um componente por vez.

Os avisos de desenvolvimento não afetarão o comportamento de tempo de execução do seu aplicativo. Dessa forma, você pode ter certeza de que seu aplicativo se comportará da mesma forma entre as compilações de desenvolvimento e produção - as únicas diferenças são que a compilação de produção não registrará os avisos e que ela é mais eficiente. (Se você notar o contrário, registre um problema.)

### O que conta como uma mudança radical? {/*what-counts-as-a-breaking-change*/}

Em geral, *nós* não aumentamos o número da versão principal para alterações em:

*   **Avisos de desenvolvimento.** Como estes não afetam o comportamento de produção, podemos adicionar novos avisos ou modificar avisos existentes entre as versões principais. Na verdade, é isso que nos permite avisar de forma confiável sobre as próximas mudanças radicais.
*   **APIs começando com `unstable_`.** Estes são fornecidos como recursos experimentais cujas APIs ainda não temos confiança. Ao liberá-los com um prefixo `unstable_`, podemos iterar mais rapidamente e chegar a uma API estável mais cedo.
*   **Versões Alfa e Canary do React.** Fornecemos versões alfa do React como uma forma de testar novos recursos antecipadamente, mas precisamos da flexibilidade para fazer alterações com base no que aprendemos no período alfa. Se você usa essas versões, observe que as APIs podem mudar antes do lançamento estável.
*   **APIs não documentadas e estruturas de dados internas.** Se você acessar nomes de propriedades internas como `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` ou `__reactInternalInstance$uk43rzhitjg`, não há garantia. Você está por sua conta.

Esta política foi elaborada para ser pragmática: certamente, não queremos lhe causar dores de cabeça. Se aumentássemos a versão principal para todas essas alterações, acabaríamos lançando mais versões principais e, em última análise, causando mais dor de versionamento para a comunidade. Também significaria que não podemos progredir em melhorar o React tão rápido quanto gostaríamos.

Dito isso, se esperarmos que uma alteração nesta lista cause grandes problemas na comunidade, ainda faremos o possível para fornecer um caminho de migração gradual.

### Se um lançamento secundário não incluir novos recursos, por que não é um patch? {/*if-a-minor-release-includes-no-new-features-why-isnt-it-a-patch*/}

É possível que um lançamento secundário não inclua novos recursos. [Isso é permitido pelo semver](https://semver.org/#spec-item-7), que afirma **"[uma versão secundária] PODE ser incrementada se novas funcionalidades ou melhorias substanciais forem introduzidas no código privado. PODE incluir alterações de nível de patch."**

No entanto, levanta a questão de por que esses lançamentos não são versionados como patches.

A resposta é que qualquer alteração no React (ou outro software) acarreta algum risco de quebra de maneiras inesperadas. Imagine um cenário em que um lançamento de patch que corrige um erro introduza acidentalmente um erro diferente. Isso não apenas seria perturbador para os desenvolvedores, mas também prejudicaria sua confiança em futuros lançamentos de patch. É especialmente lamentável se a correção original for para um erro que raramente é encontrado na prática.

Temos um bom histórico de manter os lançamentos do React livres de erros, mas os lançamentos de patch têm uma barra ainda maior para confiabilidade porque a maioria dos desenvolvedores assume que eles podem ser adotados sem consequências adversas.

Por essas razões, reservamos os lançamentos de patch apenas para os bugs e vulnerabilidades de segurança mais críticos.

Se um lançamento incluir alterações não essenciais — como refatorações internas, alterações em detalhes de implementação, melhorias de desempenho ou pequenas correções de erros — aumentaremos a versão secundária, mesmo quando não houver novos recursos.

## Todos os canais de lançamento {/*all-release-channels*/}

O React conta com uma comunidade de código aberto próspera para registrar