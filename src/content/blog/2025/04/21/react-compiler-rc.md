---
title: "React Compiler RC"
author: Lauren Tan and Mofei Zhang
date: 2025/04/21
description: Estamos lançando o primeiro Release Candidate (RC) do compilador hoje.

---

21 de abril de 2025 por [Lauren Tan](https://x.com/potetotes) e [Mofei Zhang](https://x.com/zmofei).

---

<Intro>

A equipe do React está animada para compartilhar novas atualizações:

</Intro>

1. Estamos publicando o React Compiler RC hoje, em preparação para o lançamento estável do compilador.
2. Estamos mesclando `eslint-plugin-react-compiler` em `eslint-plugin-react-hooks`.
3. Adicionamos suporte para swc e estamos trabalhando com oxc para oferecer suporte a builds sem Babel.

---

[React Compiler](https://react.dev/learn/react-compiler) é uma ferramenta de tempo de build que otimiza seu aplicativo React por meio de memorização automática. No ano passado, publicamos a [primeira versão beta](https://react.dev/blog/2024/10/21/react-compiler-beta-release) do React Compiler e recebemos muitos comentários e contribuições excelentes. Estamos entusiasmados com as vitórias que vimos das pessoas que adotaram o compilador (veja os estudos de caso de [Sanity Studio](https://github.com/reactwg/react-compiler/discussions/33) e [Wakelet](https://github.com/reactwg/react-compiler/discussions/52)) e estamos trabalhando para um lançamento estável.

Estamos lançando o primeiro Release Candidate (RC) do compilador hoje. O RC tem como objetivo ser uma versão estável e quase final do compilador, e seguro para experimentar em produção.

## Use React Compiler RC hoje {/*use-react-compiler-rc-today*/}
Para instalar o RC:

npm
<TerminalBlock>
{`npm install --save-dev --save-exact babel-plugin-react-compiler@rc`}
</TerminalBlock>

pnpm
<TerminalBlock>
{`pnpm add --save-dev --save-exact babel-plugin-react-compiler@rc`}
</TerminalBlock>

yarn
<TerminalBlock>
{`yarn add --dev --exact babel-plugin-react-compiler@rc`}
</TerminalBlock>

Como parte do RC, tornamos o React Compiler mais fácil de adicionar aos seus projetos e adicionamos otimizações à forma como o compilador gera a memorização. O React Complier agora oferece suporte a cadeias opcionais e índices de array como dependências. Estamos explorando como inferir ainda mais dependências, como verificações de igualdade e interpolação de strings. Essas melhorias resultam, em última análise, em menos re-renders e UIs mais responsivas.

Também ouvimos da comunidade que a validação de ref-in-render às vezes tem falsos positivos. Como uma filosofia geral, queremos que você possa confiar totalmente nas mensagens de erro e dicas do compilador, estamos desativando-o por padrão por enquanto. Continuaremos trabalhando para melhorar essa validação e a reativaremos em um lançamento de acompanhamento.

Você pode encontrar mais detalhes sobre como usar o Compiler em [nossos documentos](https://react.dev/learn/react-compiler).

## Feedback {/*feedback*/}
Durante o período do RC, incentivamos todos os usuários do React a experimentar o compilador e fornecer feedback no repositório do React. Por favor, [abra um problema](https://github.com/facebook/react/issues) se você encontrar algum bug ou comportamento inesperado. Se você tiver uma pergunta ou sugestão geral, publique-as no [React Compiler Working Group](https://github.com/reactwg/react-compiler/discussions).

## Compatibilidade com versões anteriores {/*backwards-compatibility*/}
Conforme observado no anúncio Beta, o React Compiler é compatível com React 17 e superior. Se você ainda não estiver no React 19, poderá usar o React Compiler especificando um destino mínimo na configuração do compilador e adicionando `react-compiler-runtime` como uma dependência. Você pode encontrar documentos sobre isso [aqui](https://react.dev/learn/react-compiler#using-react-compiler-with-react-17-or-18).

## Migrando de eslint-plugin-react-compiler para eslint-plugin-react-hooks {/*migrating-from-eslint-plugin-react-compiler-to-eslint-plugin-react-hooks*/}
Se você já instalou o eslint-plugin-react-compiler, agora pode removê-lo e usar `eslint-plugin-react-hooks@rc`. Muito obrigado a [@michaelfaith](https://bsky.app/profile/michael.faith) por contribuir para essa melhoria!

Para instalar:

npm
<TerminalBlock>
{`npm install --save-dev eslint-plugin-react-hooks@rc`}
</TerminalBlock>

pnpm
<TerminalBlock>
{`pnpm add --save-dev eslint-plugin-react-hooks@rc`}
</TerminalBlock>

yarn
<TerminalBlock>
{`yarn add --dev eslint-plugin-react-hooks@rc`}
</TerminalBlock>

```js
// eslint.config.js
import * as reactHooks from 'eslint-plugin-react-hooks';

export default [
  // Flat Config (eslint 9+)
  reactHooks.configs.recommended,

  // Legacy Config
  reactHooks.configs['recommended-latest']
];
```

Para habilitar a regra do React Compiler, adicione `'react-hooks/react-compiler': 'error'` à sua configuração do ESLint.

O linter não exige que o compilador seja instalado, portanto, não há risco em atualizar o eslint-plugin-react-hooks. Recomendamos que todos atualizem hoje.

## Suporte swc (experimental) {/*swc-support-experimental*/}
O React Compiler pode ser instalado em [várias ferramentas de build](/learn/react-compiler#installation), como Babel, Vite e Rsbuild.

Além dessas ferramentas, temos colaborado com Kang Dongyoon ([@kdy1dev](https://x.com/kdy1dev)) da equipe [swc](https://swc.rs/) para adicionar suporte adicional ao React Compiler como um plugin swc. Embora esse trabalho não esteja concluído, o desempenho da build do Next.js agora deve ser consideravelmente mais rápido quando o [React Compiler estiver habilitado em seu aplicativo Next.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/reactCompiler).

Recomendamos o uso do Next.js [15.3.1](https://github.com/vercel/next.js/releases/tag/v15.3.1) ou superior para obter o melhor desempenho de build.

Os usuários do Vite podem continuar a usar [vite-plugin-react](https://github.com/vitejs/vite-plugin-react) para habilitar o compilador, adicionando-o como um [plugin Babel](https://react.dev/learn/react-compiler#usage-with-vite). Também estamos trabalhando com a equipe [oxc](https://oxc.rs/) para [adicionar suporte ao compilador](https://github.com/oxc-project/oxc/issues/10048). Assim que [rolldown](https://github.com/rolldown/rolldown) for lançado e suportado oficialmente no Vite e o suporte oxc for adicionado ao React Compiler, atualizaremos os documentos com informações sobre como migrar.

## Atualizando o React Compiler {/*upgrading-react-compiler*/}
O React Compiler funciona melhor quando a auto-memorização aplicada é estritamente para desempenho. Versões futuras do compilador podem alterar como a memorização é aplicada, por exemplo, ela pode se tornar mais granular e precisa.

No entanto, como o código do produto pode, às vezes, quebrar as [regras do React](https://react.dev/reference/rules) de maneiras que nem sempre são detectáveis estaticamente em JavaScript, a alteração da memorização pode, ocasionalmente, ter resultados inesperados. Por exemplo, um valor memorizado anteriormente pode ser usado como uma dependência para um useEffect em algum lugar na árvore de componentes. Alterar como ou se esse valor é memorizado pode causar disparos excessivos ou insuficientes desse useEffect. Embora incentivemos o [useEffect apenas para sincronização](https://react.dev/learn/synchronizing-with-effects), sua base de código pode ter useEffects que cobrem outros casos de uso, como efeitos que precisam ser executados apenas em resposta a valores específicos em mudança.

Em outras palavras, a alteração da memorização pode, em raras circunstâncias, causar um comportamento inesperado. Por esse motivo, recomendamos seguir as Regras do React e empregar testes contínuos de ponta a ponta do seu aplicativo para que você possa atualizar o compilador com confiança e identificar quaisquer violações das regras do React que possam causar problemas.

Se você não tiver uma boa cobertura de teste, recomendamos fixar o compilador em uma versão exata (por exemplo, `19.1.0`) em vez de um intervalo SemVer (por exemplo, `^19.1.0`). Você pode fazer isso passando as flags `--save-exact` (npm/pnpm) ou `--exact` (yarn) ao atualizar o compilador. Em seguida, você deve fazer quaisquer atualizações do compilador manualmente, tomando cuidado para verificar se seu aplicativo ainda funciona conforme o esperado.

## Roteiro para Estável {/*roadmap-to-stable*/}
*Este não é um roteiro final e está sujeito a alterações.*

Após um período de feedback final da comunidade sobre o RC, planejamos um Lançamento Estável para o compilador.

* ✅ Experimental: Lançado na React Conf 2024, principalmente para feedback de desenvolvedores de aplicativos.
* ✅ Beta Pública: Disponível hoje, para feedback de autores de bibliotecas.
* ✅ Release Candidate (RC): O React Compiler funciona para a maioria dos aplicativos e bibliotecas que seguem as regras sem problemas.
* Disponibilidade Geral: Após o período de feedback final da comunidade.

Após o Estável, planejamos adicionar mais otimizações e melhorias do compilador. Isso inclui melhorias contínuas na memorização automática e novas otimizações, com pouca ou nenhuma alteração no código do produto. Cada atualização continuará a melhorar o desempenho e adicionar um melhor tratamento de diversos padrões JavaScript e React.

---

Agradecimentos a [Joe Savona](https://x.com/en_JS), [Jason Bonta](https://x.com/someextent), [Jimmy Lai](https://x.com/feedthejim) e [Kang Dongyoon](https://x.com/kdy1dev) (@kdy1dev) por revisar e editar esta postagem.