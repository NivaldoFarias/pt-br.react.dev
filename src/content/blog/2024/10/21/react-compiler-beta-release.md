---
title: "Lançamento Beta do React Compiler"
author: Lauren Tan
date: 2024/10/21
description: Na React Conf 2024, anunciamos o lançamento experimental do React Compiler, uma ferramenta de tempo de build que otimiza seu aplicativo React por meio de memoização automática. Nesta publicação, queremos compartilhar o que está por vir para código aberto e nosso progresso no compilador.

---

21 de outubro de 2024 por [Lauren Tan](https://twitter.com/potetotes).

---

<Intro>

A equipe do React está animada para compartilhar novas atualizações:

</Intro>

1. Estamos publicando o React Compiler Beta hoje, para que os primeiros usuários e mantenedores de bibliotecas possam experimentá-lo e fornecer feedback.
2. Estamos oficialmente suportando o React Compiler para aplicativos no React 17+, por meio de um pacote opcional `react-compiler-runtime`.
3. Estamos abrindo a participação pública do [React Compiler Working Group](https://github.com/reactwg/react-compiler) para preparar a comunidade para a adoção gradual do compilador.

---

Na [React Conf 2024](/blog/2024/05/22/react-conf-2024-recap), anunciamos o lançamento experimental do React Compiler, uma ferramenta em tempo de build que otimiza seu aplicativo React por meio de memoização automática. [Você pode encontrar uma introdução ao React Compiler aqui](/learn/react-compiler).

Desde o primeiro lançamento, corrigimos vários erros relatados pela comunidade React, recebemos várias correções de erros de alta qualidade e contribuições[^1] para o compilador, tornamos o compilador mais resiliente à ampla diversidade de padrões JavaScript e continuamos a implantar o compilador de forma mais ampla na Meta.

Nesta publicação, queremos compartilhar o que está por vir para o React Compiler.

## Experimente o React Compiler Beta hoje {/*try-react-compiler-beta-today*/}

Na [React India 2024](https://www.youtube.com/watch?v=qd5yk2gxbtg), compartilhamos uma atualização sobre o React Compiler. Hoje, temos o prazer de anunciar um novo lançamento Beta do React Compiler e do plugin ESLint. Novas versões beta são publicadas no npm usando a tag `@beta`.

Para instalar o React Compiler Beta:

<TerminalBlock>
npm install -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Ou, se você estiver usando Yarn:

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Você pode assistir à palestra de [Sathya Gunasekaran](https://twitter.com/_gsathya) na React India aqui:

<YouTubeIframe src="https://www.youtube.com/embed/qd5yk2gxbtg" />

## Recomendamos que todos usem o linter do React Compiler hoje {/*we-recommend-everyone-use-the-react-compiler-linter-today*/}

O plugin ESLint do React Compiler ajuda os desenvolvedores a identificar e corrigir proativamente violações das [Regras do React](/reference/rules). **Nós recomendamos fortemente que todos usem o linter hoje**. O linter não exige que você tenha o compilador instalado, portanto você pode usá-lo independentemente, mesmo que não esteja pronto para experimentar o compilador.

Para instalar apenas o linter:

<TerminalBlock>
npm install -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Ou, se você estiver usando Yarn:

<TerminalBlock>
yarn add -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Após a instalação, você pode habilitar o linter [adicionando-o à sua configuração ESLint](/learn/react-compiler#installing-eslint-plugin-react-compiler). Usar o linter ajuda a identificar quebras nas Regras do React, tornando mais fácil adotar o compilador quando ele for totalmente lançado.

## Compatibilidade com versões anteriores {/*backwards-compatibility*/}

O React Compiler produz código que depende das APIs de tempo de execução adicionadas no React 19, mas desde então adicionamos suporte para o compilador também funcionar com React 17 e 18. Se você ainda não estiver no React 19, no lançamento Beta agora você pode testar o React Compiler especificando um `target` mínimo em sua configuração do compilador e adicionando `react-compiler-runtime` como uma dependência. [Você pode encontrar documentos sobre isso aqui](/learn/react-compiler#using-react-compiler-with-react-17-or-18).

## Usando o React Compiler em bibliotecas {/*using-react-compiler-in-libraries*/}

Nosso lançamento inicial foi focado na identificação de grandes problemas com o uso do compilador em aplicativos. Recebemos ótimos comentários e melhoramos substancialmente o compilador desde então. Agora estamos prontos para feedback amplo da comunidade e para que os autores de bibliotecas experimentem o compilador para melhorar o desempenho e a experiência do desenvolvedor na manutenção de sua biblioteca.

O React Compiler também pode ser usado para compilar bibliotecas. Como o React Compiler precisa ser executado no código-fonte original antes de qualquer transformação de código, não é possível que o pipeline de build de um aplicativo compile as bibliotecas que ele usa. Portanto, nossa recomendação é que os mantenedores da biblioteca compilem e testem suas bibliotecas de forma independente com o compilador e enviem o código compilado para o npm.

Como seu código é pré-compilado, os usuários de sua biblioteca não precisarão ter o compilador ativado para se beneficiarem da memoização automática aplicada à sua biblioteca. Se sua biblioteca tiver como alvo aplicativos que ainda não estão no React 19, especifique um `target` mínimo e adicione o `react-compiler-runtime` como uma dependência direta. O pacote de tempo de execução usará a implementação correta das APIs dependendo da versão do aplicativo e fará o polyfill das APIs ausentes, se necessário.

[Você pode encontrar mais documentação sobre isso aqui.](/learn/react-compiler#using-the-compiler-on-libraries)

## Abrindo o React Compiler Working Group para todos {/*opening-up-react-compiler-working-group-to-everyone*/}

Anunciamos anteriormente o [React Compiler Working Group](https://github.com/reactwg/react-compiler), somente por convite, na React Conf para fornecer feedback, fazer perguntas e colaborar no lançamento experimental do compilador.

A partir de hoje, junto com o lançamento Beta do React Compiler, estamos abrindo a participação no Working Group para todos. A meta do React Compiler Working Group é preparar o ecossistema para uma adoção suave e gradual do React Compiler por aplicativos e bibliotecas existentes. Continue a registrar relatórios de erros no [repositório React](https://github.com/facebook/react), mas deixe feedback, faça perguntas ou compartilhe ideias no [fórum de discussão do Grupor de Trabalho](https://github.com/reactwg/react-compiler/discussions).

A equipe principal também usará o repositório de discussões para compartilhar nossos resultados de pesquisa. À medida que o lançamento estável se aproxima, qualquer informação importante também será postada neste fórum.

## React Compiler na Meta {/*react-compiler-at-meta*/}

Na [React Conf](/blog/2024/05/22/react-conf-2024-recap), compartilhamos que nossa implantação do compilador na Quest Store e no Instagram foi bem-sucedida. Desde então, implantamos o React Compiler em vários aplicativos da web importantes da Meta, incluindo [Facebook](https://www.facebook.com) e [Threads](https://www.threads.net). Isso significa que, se você usou algum desses aplicativos recentemente, pode ter tido sua experiência alimentada pelo compilador. Conseguimos integrar esses aplicativos no compilador com poucas alterações de código necessárias, em um monorepositório com mais de 100.000 componentes React.

Vimos melhorias notáveis de desempenho em todos esses aplicativos. Conforme implementamos, continuamos a ver resultados da ordem [das vitórias que compartilhamos anteriormente na ReactConf](https://youtu.be/lyEKhv8-3n0?t=3223). Esses aplicativos já foram muito ajustados e otimizados por engenheiros da Meta e especialistas em React ao longo dos anos, então mesmo melhorias da ordem de alguns por cento são uma grande vitória para nós.

Também esperávamos ganhos de produtividade do desenvolvedor com o React Compiler. Para medir isso, colaboramos com nossos parceiros de ciência de dados na Meta[^2] para conduzir uma análise estatística completa do impacto da memoização manual na produtividade. Antes de implantar o compilador na Meta, descobrimos que apenas cerca de 8% das solicitações de pull do React usavam memoização manual e que essas solicitações de pull levavam de 31 a 46% mais tempo para serem criadas[^3]. Isso confirmou nossa intuição de que a memoização manual introduz sobrecarga cognitiva, e prevemos que o React Compiler levará a uma criação e revisão de código mais eficientes. Notavelmente, o React Compiler também garante que *todo* o código seja memoizado por padrão, e não apenas os (em nosso caso) 8% em que os desenvolvedores aplicam explicitamente a memoização.

## Roteiro para Estável {/*roadmap-to-stable*/}

*Este não é um roteiro final e está sujeito a alterações.*

Pretendemos enviar um Release Candidate do compilador em um futuro próximo após a versão Beta, quando a maioria dos aplicativos e bibliotecas que seguem as Regras do React tiverem comprovadamente um bom funcionamento com o compilador. Após um período de feedback final da comunidade, planejamos um lançamento estável para o compilador. O lançamento estável marcará o início de uma nova base para o React, e todos os aplicativos e bibliotecas são fortemente recomendados a usar o compilador e o plugin ESLint.

* ✅ Experimental: Lançado na React Conf 2024, principalmente para feedback dos primeiros usuários.
* ✅ Beta Pública: Disponível hoje, para feedback da comunidade em geral.
* 🚧 Release Candidate (RC): O React Compiler funciona para a maioria dos aplicativos e bibliotecas que seguem regras sem problemas.
* 🚧 Disponibilidade Geral: Após o período de feedback final da comunidade.

Esses lançamentos também incluem o plugin ESLint do compilador, que mostra diagnósticos analisados estaticamente pelo compilador. Planejamos combinar o plugin eslint-plugin-react-hooks existente com o plugin ESLint do compilador, para que apenas um plugin precise ser instalado.

Após o lançamento estável, planejamos adicionar mais otimizações e melhorias do compilador. Isso inclui melhorias contínuas na memoização automática e novas otimizações, com pouca ou nenhuma alteração no código do produto. A atualização para cada novo lançamento do compilador visa ser direta, e cada atualização continuará a melhorar o desempenho e adicionar um melhor tratamento de diversos padrões JavaScript e React.

Ao longo deste processo, também planejamos prototipar uma extensão IDE para React. Ainda está muito no início da pesquisa, então esperamos poder compartilhar mais de nossos resultados com você em uma futura postagem do blog React Labs.

---

Obrigado a [Sathya Gunasekaran](https://twitter.com/_gsathya), [Joe Savona](https://twitter.com/en_JS), [Ricky Hanlon](https://twitter.com/rickhanlonii), [Alex Taylor](https://github.com/alexmckenley), [Jason Bonta](https://twitter.com/someextent) e [Eli White](https://twitter.com/Eli_White) por revisar e editar esta postagem.

---

[^1]: Obrigado a [@nikeee](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Anikeee), [@henryqdineen](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Ahenryqdineen), [@TrickyPi](https://github.com/facebook/react/pulls?q=is%3Apr+author%3ATrickyPi) e vários outros por suas contribuições ao compilador.

[^2]: Obrigado a [Vaishali Garg](https://www.linkedin.com/in/vaishaligarg09) por liderar este estudo sobre o React Compiler na Meta e por revisar esta postagem.

[^3]: Após controlar o tempo de autor, o comprimento/complexidade da diferença e outros fatores de confusão potenciais.
```