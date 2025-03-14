---
title: Regras do React
---

<Intro>
Assim como diferentes linguagens de programação têm suas próprias maneiras de expressar conceitos, o React tem seus próprios idiomas — ou regras — para expressar padrões de uma maneira que seja fácil de entender e produza aplicativos de alta qualidade.
</Intro>

<InlineToc />

---

<Note>
Para saber mais sobre como expressar UIs com React, recomendamos [Thinking in React](/learn/thinking-in-react).
</Note>

Esta seção descreve as regras que você precisa seguir para escrever código React idiomático. Escrever código React idiomático pode ajudar você a escrever aplicativos bem organizados, seguros e compostos. Essas propriedades tornam seu aplicativo mais resiliente a mudanças e facilitam o trabalho com outros desenvolvedores, bibliotecas e ferramentas.

Essas regras são conhecidas como **Regras do React**. Elas são regras – e não apenas diretrizes – no sentido de que, se forem violadas, seu aplicativo provavelmente terá erros. Seu código também se torna não idiomático e mais difícil de entender e raciocinar.

Recomendamos fortemente o uso de [Strict Mode](/reference/react/StrictMode) junto com o [plugin ESLint](https://www.npmjs.com/package/eslint-plugin-react-hooks) do React para ajudar seu codebase a seguir as Regras do React. Seguindo as Regras do React, você poderá encontrar e resolver esses erros e manter seu aplicativo sustentável.

---

## Componentes e Hooks devem ser puros {/*components-and-hooks-must-be-pure*/}

[Pureza em Componentes e Hooks](/reference/rules/components-and-hooks-must-be-pure) é uma regra chave do React que torna seu aplicativo previsível, fácil de depurar e permite que o React otimize automaticamente seu código.

*   [Componentes devem ser idempotentes](/reference/rules/components-and-hooks-must-be-pure#components-and-hooks-must-be-idempotent) – Componentes React são considerados sempre retornarem a mesma saída em relação às suas entradas – props, state e context.
*   [Efeitos colaterais devem ser executados fora do renderizar](/reference/rules/components-and-hooks-must-be-pure#side-effects-must-run-outside-of-render) – Efeitos colaterais não devem ser executados no renderizar, pois o React pode renderizar componentes várias vezes para criar a melhor experiência possível para o usuário.
*   [Props e state são imutáveis](/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable) – As props e o state de um componente são snapshots imutáveis em relação a um único renderizar. Nunca os mute diretamente.
*   [Valores de retorno e argumentos para Hooks são imutáveis](/reference/rules/components-and-hooks-must-be-pure#return-values-and-arguments-to-hooks-are-immutable) – Depois que os valores são passados para um Hook, você não deve modificá-los. Como props em JSX, os valores se tornam imutáveis quando passados para um Hook.
*   [Valores são imutáveis depois de serem passados para JSX](/reference/rules/components-and-hooks-must-be-pure#values-are-immutable-after-being-passed-to-jsx) – Não mute os valores depois que eles forem usados em JSX. Mova a mutação antes que o JSX seja criado.

---

## React chama Componentes e Hooks {/*react-calls-components-and-hooks*/}

[O React é responsável por renderizar componentes e hooks quando necessário para otimizar a experiência do usuário.](/reference/rules/react-calls-components-and-hooks) Ele é declarativo: você diz ao React o que renderizar na lógica do seu componente, e o React descobrirá a melhor forma de exibi-lo para seu usuário.

*   [Nunca chame funções de componente diretamente](/reference/rules/react-calls-components-and-hooks#never-call-component-functions-directly) – Componentes devem ser usados apenas em JSX. Não os chame como funções regulares.
*   [Nunca passe hooks como valores regulares](/reference/rules/react-calls-components-and-hooks#never-pass-around-hooks-as-regular-values) – Hooks devem ser chamados apenas dentro de componentes. Nunca o passe como um valor regular.

---

## Regras de Hooks {/*rules-of-hooks*/}

Hooks são definidos usando funções JavaScript, mas eles representam um tipo especial de lógica de UI reutilizável com restrições sobre onde eles podem ser chamados. Você precisa seguir as [Regras de Hooks](/reference/rules/rules-of-hooks) ao usá-los.

*   [Chame os Hooks apenas no nível superior](/reference/rules/rules-of-hooks#only-call-hooks-at-the-top-level) – Não chame Hooks dentro de loops, condições ou funções aninhadas. Em vez disso, sempre use Hooks no nível superior de sua função React, antes de quaisquer retornos antecipados.
*   [Chame os Hooks apenas de funções React](/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions) – Não chame Hooks de funções JavaScript regulares.