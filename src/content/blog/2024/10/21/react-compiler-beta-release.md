---
title: "Lançamento Beta do React Compiler"
author: Lauren Tan
date: 2024/10/21
description: Na React Conf 2024, anunciamos o lançamento experimental do React Compiler, uma ferramenta de tempo de build que otimiza seu aplicativo React por meio de memoização automática. Nesta postagem, queremos compartilhar o que vem a seguir para o código aberto e nosso progresso no compilador.

---

21 de outubro de 2024 por [Lauren Tan](https://twitter.com/potetotes).

---

<Intro>

A equipe do React está animada para compartilhar novas atualizações:

</Intro>

1. Estamos publicando React Compiler Beta hoje, para que os primeiros adotantes e mantenedores de bibliotecas possam experimentá-lo e fornecer *feedback*.
2. Estamos oficialmente dando suporte ao React Compiler para aplicativos no React 17+, por meio de um pacote opcional `react-compiler-runtime`.
3. Estamos abrindo a adesão pública do [React Compiler Working Group](https://github.com/reactwg/react-compiler) para preparar a comunidade para a adoção gradual do compilador.

---

Na [React Conf 2024](/blog/2024/05/22/react-conf-2024-recap), anunciamos o lançamento experimental do React Compiler, uma ferramenta de tempo de *build* que otimiza seu aplicativo React por meio de memoização automática. [Você pode encontrar uma introdução ao React Compiler aqui](/learn/react-compiler).

Desde o primeiro lançamento, corrigimos vários *bugs* relatados pela comunidade React, recebemos várias correções de *bugs* e contribuições de alta qualidade[^1] para o compilador, tornamos o compilador mais resiliente à ampla diversidade de padrões JavaScript e continuamos a lançar o compilador de forma mais ampla na Meta.

Nesta postagem, queremos compartilhar o que vem a seguir para o React Compiler.

## Experimente o React Compiler Beta hoje {/*try-react-compiler-beta-today*/}

Na [React India 2024](https://www.youtube.com/watch?v=qd5yk2gxbtg), compartilhamos uma atualização sobre o React Compiler. Hoje, estamos animados em anunciar um novo lançamento Beta do React Compiler e do plugin ESLint. Novas versões beta são publicadas no npm usando a *tag* `@beta`.

Para instalar o React Compiler Beta:

<TerminalBlock>
npm install -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Ou, se você estiver usando Yarn:

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Você pode assistir a palestra de [Sathya Gunasekaran](https://twitter.com/_gsathya) na React India aqui:

<YouTubeIframe src="https://www.youtube.com/embed/qd5yk2gxbtg" />

## Recomendamos que todos usem o linter do React Compiler hoje {/*we-recommend-everyone-use-the-react-compiler-linter-today*/}

O plugin ESLint do React Compiler ajuda os desenvolvedores a identificar e corrigir proativamente violações das [Regras do React](/reference/rules). **Recomendamos fortemente que todos usem o *linter* hoje**. O *linter* não exige que você tenha o compilador instalado, para que possa usá-lo de forma independente, mesmo que não esteja pronto para testar o compilador.

Para instalar o *linter* apenas:

<TerminalBlock>
npm install -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Ou, se você estiver usando Yarn:

<TerminalBlock>
yarn add -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Após a instalação, você pode habilitar o *linter* [adicionando-o à sua configuração do ESLint](/learn/react-compiler#installing-eslint-plugin-react-compiler). Usar o *linter* ajuda a identificar quebras das Regras do React, facilitando a adoção do compilador quando ele for totalmente lançado.

## Compatibilidade com versões anteriores {/*backwards-compatibility*/}

React Compiler produz código que depende de APIs de *runtime* adicionadas no React 19, mas, desde então, adicionamos suporte para o compilador também funcionar com React 17 e 18. Se você ainda não estiver no React 19, na versão Beta agora você pode testar o React Compiler especificando um `target` mínimo em sua configuração do compilador e adicionando `react-compiler-runtime` como uma dependência. [Você pode encontrar documentação sobre isso aqui](/learn/react-compiler#using-react-compiler-with-react-17-or-18).

## Usando React Compiler em bibliotecas {/*using-react-compiler-in-libraries*/}

Nosso lançamento inicial foi focado em identificar as principais questões sobre o uso do compilador em aplicativos. Recebemos ótimos *feedbacks* e melhoramos substancialmente o compilador desde então. Agora, estamos prontos para um amplo *feedback* da comunidade e para que os autores de bibliotecas testem o compilador para melhorar o desempenho e a experiência do desenvolvedor na manutenção de sua biblioteca.

React Compiler também pode ser usado para compilar bibliotecas. Como o React Compiler precisa ser executado no código-fonte original antes de qualquer transformação de código, não é possível que o *pipeline* de *build* de um aplicativo compile as bibliotecas que eles usam. Portanto, nossa recomendação é que os mantenedores de bibliotecas compilem e testem suas bibliotecas de forma independente com o compilador e enviem o código compilado para o npm.

Como seu código é pré-compilado, os usuários de sua biblioteca não precisarão ter o compilador habilitado para se beneficiarem da memoização automática aplicada à sua biblioteca. Se sua biblioteca tiver como alvo aplicativos que ainda não estão no React 19, especifique um `target` mínimo e adicione `react-compiler-runtime` como uma dependência direta. O pacote *runtime* usará a implementação correta de APIs dependendo da versão do aplicativo e *polyfill* as APIs ausentes, se necessário.

[Você pode encontrar mais documentação sobre isso aqui.](/learn/react-compiler#using-the-compiler-on-libraries)

## Abrindo o React Compiler Working Group para todos {/*opening-up-react-compiler-working-group-to-everyone*/}

Anunciamos anteriormente o [React Compiler Working Group](https://github.com/reactwg/react-compiler), somente para convidados, na React Conf para fornecer *feedback*, fazer perguntas e colaborar no lançamento experimental do compilador.

A partir de hoje, juntamente com o lançamento Beta do React Compiler, estamos abrindo a adesão ao Working Group para todos. O objetivo do React Compiler Working Group é preparar o ecossistema para uma adoção suave e gradual do React Compiler por aplicativos e bibliotecas existentes. Continue a registrar relatórios de *bugs* no [repositório React](https://github.com/facebook/react), mas deixe *feedbacks*, faça perguntas ou compartilhe ideias no [fórum de discussão do Working Group](https://github.com/reactwg/react-compiler/discussions).

A equipe principal também usará o repositório de discussões para compartilhar nossas descobertas de pesquisa. À medida que o lançamento estável se aproxima, qualquer informação importante também será postada neste fórum.

## React Compiler na Meta {/*react-compiler-at-meta*/}

Na [React Conf](/blog/2024/05/22/react-conf-2024-recap), compartilhamos que nossa implantação do compilador na Quest Store e no Instagram foi bem-sucedida. Desde então, implantamos o React Compiler em vários aplicativos da web importantes na Meta, incluindo [Facebook](https://www.facebook.com) e [Threads](https://www.threads.net). Isso significa que, se você usou algum desses aplicativos recentemente, sua experiência pode ter sido impulsionada pelo compilador. Conseguimos incorporar esses aplicativos no compilador com poucas alterações de código necessárias, em um monorepo com mais de 100.000 componentes React.

Vimos melhorias notáveis ​​de desempenho em todos esses aplicativos. À medida que implementamos, continuamos a ver resultados da ordem [das vitórias que compartilhamos anteriormente na ReactConf](https://youtu.be/lyEKhv8-3n0?t=3223). Esses aplicativos já foram manualmente ajustados e otimizados por engenheiros da Meta e especialistas em React ao longo dos anos, então, mesmo melhorias da ordem de alguns pontos percentuais são uma grande vitória para nós.

Também esperávamos vitórias na produtividade do desenvolvedor com o React Compiler. Para medir isso, colaboramos com nossos parceiros de ciência de dados na Meta[^2] para conduzir uma análise estatística completa do impacto da memoização manual na produtividade. Antes de lançar o compilador na Meta, descobrimos que apenas cerca de 8% das *pull requests* do React usavam memoização manual e que essas *pull requests* levavam de 31 a 46% mais tempo para serem criadas[^3]. Isso confirmou nossa intuição de que a memoização manual introduz sobrecarga cognitiva e antecipamos que o React Compiler levará a uma criação e revisão de código mais eficientes. Notavelmente, o React Compiler também garante que *todo* o código seja memorizado por padrão, não apenas os (em nosso caso) 8% em que os desenvolvedores aplicam explicitamente a memoização.

## Roteiro para Estável {/*roadmap-to-stable*/}

*Este não é um roteiro final e está sujeito a alterações.*

Pretendemos lançar uma *Release Candidate* do compilador no futuro próximo após o lançamento Beta, quando a maioria dos aplicativos e bibliotecas que seguem as Regras do React provarem funcionar bem com o compilador. Após um período de *feedback* final da comunidade, planejamos um lançamento estável para o compilador. O lançamento estável marcará o início de uma nova fundação para o React, e todos os aplicativos e bibliotecas serão fortemente recomendados a usar o compilador e o plugin ESLint.

* ✅ Experimental: Lançado na React Conf 2024, principalmente para *feedback* dos primeiros adotantes.
* ✅ Beta público: Disponível hoje, para *feedback* da comunidade mais ampla.
* 🚧 *Release Candidate* (RC): React Compiler funciona para a maioria dos aplicativos e bibliotecas que seguem as regras sem problemas.
* 🚧 Disponibilidade Geral: Após o período de *feedback* final da comunidade.

Esses lançamentos também incluem o plugin ESLint do compilador, que exibe diagnósticos analisados ​​estaticamente pelo compilador. Planejamos combinar o plugin eslint-plugin-react-hooks existente com o plugin ESLint do compilador, para que apenas um plugin precise ser instalado.

Pós-estável, planejamos adicionar mais otimizações e melhorias ao compilador. Isso inclui melhorias contínuas na memoização automática e novas otimizações, com pouca ou nenhuma alteração no código do produto. A atualização para cada novo lançamento do compilador visa ser direta, e cada atualização continuará a melhorar o desempenho e adicionar um melhor tratamento de padrões diversos de JavaScript e React.

Durante todo esse processo, também planejamos prototipar uma extensão IDE para React. Ainda está muito no início da pesquisa, por isso esperamos poder compartilhar mais de nossas descobertas com você em uma futura postagem do blog React Labs.

---

Agradecemos a [Sathya Gunasekaran](https://twitter.com/_gsathya), [Joe Savona](https://twitter.com/en_JS), [Ricky Hanlon](https://twitter.com/rickhanlonii), [Alex Taylor](https://github.com/alexmckenley), [Jason Bonta](https://twitter.com/someextent), e [Eli White](https://twitter.com/Eli_White) por revisarem e editarem esta postagem.

---

[^1]: Obrigado [@nikeee](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Anikeee), [@henryqdineen](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Ahenryqdineen), [@TrickyPi](https://github.com/facebook/react/pulls?q=is%3Apr+author%3ATrickyPi), e vários outros por suas contribuições para o compilador.

[^2]: Obrigado a [Vaishali Garg](https://www.linkedin.com/in/vaishaligarg09) por liderar este estudo sobre React Compiler na Meta e por revisar esta postagem.

[^3]: Após controlar a posse do autor, tamanho/complexidade da diferença e outros fatores de confusão potenciais.