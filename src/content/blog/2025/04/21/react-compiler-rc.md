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
3. Adicionamos suporte para swc e estamos trabalhando com oxc para suportar builds sem Babel.

---

[React Compiler](https://react.dev/learn/react-compiler) é uma ferramenta de build-time que otimiza seu aplicativo React através de memoização automática. No ano passado, publicamos o [primeiro beta](https://react.dev/blog/2024/10/21/react-compiler-beta-release) do React Compiler e recebemos muitos feedbacks e contribuições excelentes. Estamos animados com os sucessos que vimos de pessoas adotando o compilador (veja estudos de caso de [Sanity Studio](https://github.com/reactwg/react-compiler/discussions/33) e [Wakelet](https://github.com/reactwg/react-compiler/discussions/52)) e estamos trabalhando em direção a um lançamento estável.

Estamos lançando o primeiro Release Candidate (RC) do compilador hoje. O RC é destinado a ser uma versão estável e quase final do compilador, e seguro para experimentar em produção.

## Use o React Compiler RC hoje {/*use-react-compiler-rc-today*/}
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

Como parte do RC, temos tornado o React Compiler mais fácil de adicionar aos seus projetos e adicionado otimizações à forma como o compilador gera a memoização. O React Compiler agora suporta optional chains e índices de array como dependências. Estamos explorando como inferir ainda mais dependências, como verificações de igualdade e interpolação de strings. Essas melhorias resultam, em última análise, em menos re-renderizações e UIs mais responsivas.

Também ouvimos da comunidade que a validação de ref-in-render às vezes tem falsos positivos. Como uma filosofia geral, queremos que você possa confiar totalmente nas mensagens de erro e dicas do compilador, estamos desativando-a por padrão por enquanto. Continuaremos trabalhando para melhorar essa validação e a reativaremos em uma versão futura.

Você pode encontrar mais detalhes sobre como usar o Compilador em [nossa documentação](https://react.dev/learn/react-compiler).

## Feedback {/*feedback*/}
Durante o período do RC, encorajamos todos os usuários do React a experimentar o compilador e fornecer feedback no repositório do React. Por favor, [abra uma issue](https://github.com/facebook/react/issues) se encontrar algum bug ou comportamento inesperado. Se você tiver uma pergunta geral ou sugestão, por favor, poste-as no [React Compiler Working Group](https://github.com/reactwg/react-compiler/discussions).

## Compatibilidade com Versões Anteriores {/*backwards-compatibility*/}
Conforme observado no anúncio do Beta, o React Compiler é compatível com React 17 e superior. Se você ainda não está no React 19, pode usar o React Compiler especificando um target mínimo em sua configuração do compilador e adicionando `react-compiler-runtime` como uma dependência. Você pode encontrar a documentação sobre isso [aqui](https://react.dev/learn/react-compiler#using-react-compiler-with-react-17-or-18).

## Migrando de eslint-plugin-react-compiler para eslint-plugin-react-hooks {/*migrating-from-eslint-plugin-react-compiler-to-eslint-plugin-react-hooks*/}
Se você já instalou o `eslint-plugin-react-compiler`, agora pode removê-lo e usar `eslint-plugin-react-hooks@rc`. Muitos agradecimentos a [@michaelfaith](https://bsky.app/profile/michael.faith) por contribuir para esta melhoria!

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

O linter não requer que o compilador seja instalado, portanto, não há risco em atualizar o `eslint-plugin-react-hooks`. Recomendamos que todos atualizem hoje.

## Suporte a swc (experimental) {/*swc-support-experimental*/}
O React Compiler pode ser instalado em [várias ferramentas de build](/learn/react-compiler#installation), como Babel, Vite e Rsbuild.

Além dessas ferramentas, temos colaborado com Kang Dongyoon ([@kdy1dev](https://x.com/kdy1dev)) da equipe [swc](https://swc.rs/) na adição de suporte adicional para o React Compiler como um plugin swc. Embora este trabalho não esteja concluído, o desempenho do build do Next.js agora será consideravelmente mais rápido quando o [React Compiler estiver habilitado em seu aplicativo Next.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/reactCompiler).

Recomendamos o uso do Next.js [15.3.1](https://github.com/vercel/next.js/releases/tag/v15.3.1) ou superior para obter o melhor desempenho de build.

Usuários do Vite podem continuar usando [vite-plugin-react](https://github.com/vitejs/vite-plugin-react) para habilitar o compilador, adicionando-o como um [plugin Babel](https://react.dev/learn/react-compiler#usage-with-vite). Também estamos trabalhando com a equipe [oxc](https://oxc.rs/) para [adicionar suporte ao compilador](https://github.com/oxc-project/oxc/issues/10048). Assim que o [rolldown](https://github.com/rolldown/rolldown) for oficialmente lançado e o suporte do oxc for adicionado para o React Compiler, atualizaremos a documentação com informações sobre como migrar.

## Atualizando o React Compiler {/*upgrading-react-compiler*/}
O React Compiler funciona melhor quando a auto-memoização aplicada é estritamente para performance. Versões futuras do compilador podem mudar como a memoização é aplicada, por exemplo, ela pode se tornar mais granular e precisa.

No entanto, como o código do produto às vezes pode quebrar as [regras do React](https://react.dev/reference/rules) de maneiras que nem sempre são detectáveis estaticamente em JavaScript, mudar a memoização pode ocasionalmente ter resultados inesperados. Por exemplo, um valor previamente memoizado pode ser usado como uma dependência para um `useEffect` em algum lugar na árvore de componentes. Mudar como ou se esse valor é memoizado pode causar disparos excessivos ou insuficientes desse `useEffect`. Embora incentivemos o [useEffect apenas para sincronização](https://react.dev/learn/synchronizing-with-effects), sua base de código pode ter `useEffect`s que cobrem outros casos de uso, como efeitos que precisam ser executados apenas em resposta a mudanças em valores específicos.

Em outras palavras, mudar a memoização pode, em raras circunstâncias, causar comportamento inesperado. Por esse motivo, recomendamos seguir as Regras do React e empregar testes contínuos de ponta a ponta de seu aplicativo para que você possa atualizar o compilador com confiança e identificar quaisquer violações das regras do React que possam causar problemas.

Se você não tiver uma boa cobertura de testes, recomendamos fixar o compilador em uma versão exata (por exemplo, `19.1.0`) em vez de um intervalo SemVer (por exemplo, `^19.1.0`). Você pode fazer isso passando as flags `--save-exact` (npm/pnpm) ou `--exact` (yarn) ao atualizar o compilador. Você deve então fazer quaisquer atualizações do compilador manualmente, tomando cuidado para verificar se seu aplicativo ainda funciona como esperado.

## Roteiro para Estável {/*roadmap-to-stable*/}
*Este não é um roteiro final e está sujeito a alterações.*

Após um período de feedback final da comunidade sobre o RC, planejamos um Lançamento Estável para o compilador.

* ✅ Experimental: Lançado no React Conf 2024, principalmente para feedback de desenvolvedores de aplicativos.
* ✅ Public Beta: Disponível hoje, para feedback de autores de bibliotecas.
* ✅ Release Candidate (RC): O React Compiler funciona para a maioria dos aplicativos e bibliotecas que seguem as regras sem problemas.
* Disponibilidade Geral: Após o período de feedback final da comunidade.

Após o lançamento estável, planejamos adicionar mais otimizações e melhorias ao compilador. Isso inclui melhorias contínuas na memoização automática e novas otimizações, com mínima ou nenhuma alteração no código do produto. Cada atualização continuará a melhorar o desempenho e adicionar melhor tratamento a diversos padrões de JavaScript e React.

---

Obrigado a [Joe Savona](https://x.com/en_JS), [Jason Bonta](https://x.com/someextent), [Jimmy Lai](https://x.com/feedthejim) e [Kang Dongyoon](https://x.com/kdy1dev) (@kdy1dev) por revisar e editar esta postagem.