---
title: Regras do React
---

<Intro>
Assim como diferentes linguagens de programação têm suas próprias maneiras de expressar conceitos, o React tem seus próprios idiomas — ou regras — para expressar padrões de maneira fácil de entender e produzir aplicações de alta qualidade.
</Intro>

<InlineToc />

---

<Note>
Para saber mais sobre como expressar UIs com React, recomendamos ler [Pensando em React](/learn/thinking-in-react).
</Note>

Esta seção descreve as regras que você precisa seguir para escrever código React idiomático. Escrever código React idiomático pode ajudá-lo a escrever aplicações bem organizadas, seguras e compostas. Essas propriedades tornam seu aplicativo mais resistente a mudanças e facilitam o trabalho com outros desenvolvedores, bibliotecas e ferramentas.

Essas regras são conhecidas como as **Regras do React**. Elas são regras - e não apenas diretrizes - no sentido de que, se forem quebradas, seu aplicativo provavelmente terá erros. Seu código também se torna não idiomático e mais difícil de entender e raciocinar.

Recomendamos fortemente o uso do [Strict Mode](/reference/react/StrictMode) junto com o [plugin ESLint](https://www.npmjs.com/package/eslint-plugin-react-hooks) do React para ajudar seu codebase a seguir as Regras do React. Ao seguir as Regras do React, você poderá encontrar e resolver esses erros e manter seu aplicativo fácil de manter.

---

## Componentes e Hooks devem ser puros {/*components-and-hooks-must-be-pure*/}

[Pureza em Componentes e Hooks](/reference/rules/components-and-hooks-must-be-pure) é uma regra chave do React que torna seu aplicativo previsível, fácil de depurar e permite que o React otimize automaticamente seu código.

*   [Componentes devem ser idempotentes](/reference/rules/components-and-hooks-must-be-pure#components-and-hooks-must-be-idempotent) – Componentes React são assumidos para sempre retornar a mesma saída com relação às suas entradas – props, state e context.
*   [Efeitos colaterais devem rodar fora do render](/reference/rules/components-and-hooks-must-be-pure#side-effects-must-run-outside-of-render) – Efeitos colaterais não devem ser executados em render, pois o React pode renderizar componentes várias vezes para criar a melhor experiência possível para o usuário.
*   [Props e state são imutáveis](/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable) – As props e o state de um componente são snapshots imutáveis em relação a um único render. Nunca os mute diretamente.
*   [Valores de retorno e argumentos para Hooks são imutáveis](/reference/rules/components-and-hooks-must-be-pure#return-values-and-arguments-to-hooks-are-immutable) – Uma vez que os valores são passados para um Hook, você não deve modificá-los. Como as props em JSX, os valores se tornam imutáveis quando passados para um Hook.
*   [Valores são imutáveis após serem passados para JSX](/reference/rules/components-and-hooks-must-be-pure#values-are-immutable-after-being-passed-to-jsx) – Não mute valores após eles terem sido usados em JSX. Mova a mutação antes que o JSX seja criado.

---

## React chama Componentes e Hooks {/*react-calls-components-and-hooks*/}

[O React é responsável por renderizar componentes e hooks quando necessário para otimizar a experiência do usuário.](/reference/rules/react-calls-components-and-hooks) Ele é declarativo: você diz ao React o que renderizar na lógica do seu componente, e o React descobrirá a melhor maneira de exibi-lo para o seu usuário.

*   [Nunca chame funções de componentes diretamente](/reference/rules/react-calls-components-and-hooks#never-call-component-functions-directly) – Componentes só devem ser usados em JSX. Não os chame como funções regulares.
*   [Nunca passe hooks como valores regulares](/reference/rules/react-calls-components-and-hooks#never-pass-around-hooks-as-regular-values) – Hooks só devem ser chamados dentro de componentes. Nunca o passe como um valor regular.

---

## Regras dos Hooks {/*rules-of-hooks*/}

Hooks são definidos usando funções JavaScript, mas eles representam um tipo especial de lógica de UI reutilizável com restrições sobre onde eles podem ser chamados. Você precisa seguir as [Regras dos Hooks](/reference/rules/rules-of-hooks) ao usá-los.

*   [Chame Hooks apenas na raiz](/reference/rules/rules-of-hooks#only-call-hooks-at-the-top-level) – Não chame Hooks dentro de loops, condições ou funções aninhadas. Em vez disso, sempre use Hooks na raiz de sua função React, antes de quaisquer retornos antecipados.
*   [Chame Hooks apenas de funções React](/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions) – Não chame Hooks de funções JavaScript regulares.
```