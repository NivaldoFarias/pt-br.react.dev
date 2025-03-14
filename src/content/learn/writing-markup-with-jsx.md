---
title: Escrevendo Markup com JSX
---

<Intro>

*JSX* é uma extensão de sintaxe para JavaScript que permite escrever markup semelhante a HTML dentro de um arquivo JavaScript. Embora existam outras maneiras de escrever componentes, a maioria dos desenvolvedores React prefere a concisão do JSX, e a maioria das bases de código o usam.

</Intro>

<YouWillLearn>

* Por que o React mistura markup com lógica de renderização
* Como o JSX é diferente do HTML
* Como exibir informações com JSX

</YouWillLearn>

## JSX: Colocando markup em JavaScript {/*jsx-putting-markup-into-javascript*/}

A Web foi construída em HTML, CSS e JavaScript. Por muitos anos, os desenvolvedores web mantiveram o conteúdo em HTML, o design em CSS e a lógica em JavaScript — muitas vezes em arquivos separados! O conteúdo foi marcado dentro do HTML, enquanto a lógica da página vivia separadamente no JavaScript:

<DiagramGroup>

<Diagram name="writing_jsx_html" height={237} width={325} alt="Markup HTML com fundo roxo e um div com duas tags filha: p e form. ">

HTML

</Diagram>

<Diagram name="writing_jsx_js" height={237} width={325} alt="Três manipuladores JavaScript com fundo amarelo: onSubmit, onLogin e onClick.">

JavaScript

</Diagram>

</DiagramGroup>

Mas, à medida que a Web se tornou mais interativa, a lógica determinou cada vez mais o conteúdo. O JavaScript estava encarregado do HTML! É por isso que **no React, a lógica de renderização e o markup vivem juntos no mesmo lugar — componentes.**

<DiagramGroup>

<Diagram name="writing_jsx_sidebar" height={330} width={325} alt="Componente React com HTML e JavaScript dos exemplos anteriores misturados. O nome da função é Sidebar, que chama a função isLoggedIn, destacada em amarelo. Aninhados dentro da função destacada em roxo, está a tag p do exemplo anterior e uma tag Form referenciando o componente mostrado no diagrama seguinte.">

Componente React `Sidebar.js`

</Diagram>

<Diagram name="writing_jsx_form" height={330} width={325} alt="Componente React com HTML e JavaScript dos exemplos anteriores misturados. O nome da função é Form contendo dois manipuladores onClick e onSubmit destacados em amarelo. Seguindo os manipuladores está o HTML destacado em roxo. O HTML contém um elemento form com um elemento input aninhado, cada um com uma prop onClick.">

Componente React `Form.js`

</Diagram>

</DiagramGroup>

Manter a lógica de renderização e o markup de um botão juntos garante que eles permaneçam sincronizados em cada edição. Por outro lado, detalhes que não estão relacionados, como o markup do botão e o markup de uma barra lateral, são isolados um do outro, tornando mais seguro alterá-los separadamente.

Cada componente React é uma função JavaScript que pode conter algum markup que o React renderiza no navegador. Os componentes React usam uma extensão de sintaxe chamada JSX para representar esse markup. JSX se parece muito com HTML, mas é um pouco mais rigoroso e pode exibir informações dinâmicas. A melhor maneira de entender isso é converter algum markup HTML em markup JSX.

<Note>

JSX e React são duas coisas separadas. Eles são frequentemente usados juntos, mas você *pode* [usá-los independentemente](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-a-jsx-transform) um do outro. JSX é uma extensão de sintaxe, enquanto React é uma biblioteca JavaScript.

</Note>

## Convertendo HTML em JSX {/*converting-html-to-jsx*/}

Suponha que você tenha algum HTML (perfeitamente válido):

```html
<h1>Tarefas de Hedy Lamarr</h1>
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  class="photo"
>
<ul>
    <li>Inventar novos semáforos
    <li>Ensaio de uma cena de filme
    <li>Melhorar a tecnologia de espectro
</ul>
```

E você quer colocá-lo no seu componente:

```js
export default function TodoList() {
  return (
    // ???
  )
}
```

Se você copiar e colar como está, não funcionará:

<Sandpack>

```js
export default function TodoList() {
  return (
    // This doesn't quite work!
    <h1>Hedy Lamarr's Todos</h1>
    <img 
      src="https://i.imgur.com/yXOvdOSs.jpg" 
      alt="Hedy Lamarr" 
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve the spectrum technology
    </ul>
  );
}
```

```css
img { height: 90px }
```

</Sandpack>

Isso ocorre porque o JSX é mais rigoroso e tem mais algumas regras do que o HTML! Se você ler as mensagens de erro acima, elas o guiarão para corrigir o markup, ou você pode seguir o guia abaixo.

<Note>

Na maioria das vezes, as mensagens de erro na tela do React ajudarão você a encontrar onde está o problema. Leia-as se ficar preso!

</Note>

## As Regras do JSX {/*the-rules-of-jsx*/}

### 1. Retorne um único elemento raiz {/*1-return-a-single-root-element*/}

Para retornar vários elementos de um componente, **envolva-os com uma única tag pai.**

Por exemplo, você pode usar um `<div>`:

```js {1,11}
<div>
  <h1>Tarefas de Hedy Lamarr</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```

Se você não quiser adicionar um `<div>` extra ao seu markup, você pode escrever `<>` e `</>` em vez disso:

```js {1,11}
<>
  <h1>Tarefas de Hedy Lamarr</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

Esta tag vazia é chamada de *[Fragmento.](/reference/react/Fragment)* Os Fragmentos permitem que você agrupe as coisas sem deixar nenhum vestígio na árvore HTML do navegador.

<DeepDive>

#### Por que as múltiplas tags JSX precisam ser encapsuladas? {/*why-do-multiple-jsx-tags-need-to-be-wrapped*/}

JSX se parece com HTML, mas, por baixo dos panos, ele é transformado em objetos JavaScript simples. Você não pode retornar dois objetos de uma função sem encapsulá-los em um array. Isso explica por que você também não pode retornar duas tags JSX sem encapsulá-las em outra tag ou em um Fragmento.

</DeepDive>

### 2. Feche todas as tags {/*2-close-all-the-tags*/}

JSX requer que as tags sejam fechadas explicitamente: tags de fechamento automático como `<img>` devem se tornar `<img />`, e tags de fechamento como `<li>oranges` devem ser escritas como `<li>oranges</li>`.

É assim que a imagem de Hedy Lamarr e os itens da lista aparecem fechados:

```js {2-6,8-10}
<>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
   />
  <ul>
    <li>Inventar novos semáforos</li>
    <li>Ensaio de uma cena de filme</li>
    <li>Melhorar a tecnologia de espectro</li>
  </ul>
</>
```

### 3. camelCase <s>todas</s> a maior parte das coisas! {/*3-camelcase-salls-most-of-the-things*/}

JSX se transforma em JavaScript e os atributos escritos em JSX se tornam chaves de objetos JavaScript. Em seus próprios componentes, você frequentemente desejará ler esses atributos em variáveis. Mas o JavaScript tem limitações nos nomes das variáveis. Por exemplo, seus nomes não podem conter hifens nem ser palavras reservadas como `class`.

É por isso que, no React, muitos atributos HTML e SVG são escritos em camelCase. Por exemplo, em vez de `stroke-width`, você usa `strokeWidth`. Como `class` é uma palavra reservada, no React você escreve `className` em vez disso, com o nome da [propriedade DOM correspondente](https://developer.mozilla.org/en-US/docs/Web/API/Element/className):

```js {4}
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```

Você pode [encontrar todos esses atributos na lista de props dos componentes DOM.](/reference/react-dom/components/common) Se você errar um, não se preocupe — o React imprimirá uma mensagem com uma possível correção no [console do navegador.](https://developer.mozilla.org/docs/Tools/Browser_Console)

<Pitfall>

Por razões históricas, os atributos [`aria-*`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA) e [`data-*`](https://developer.mozilla.org/docs/Learn/HTML/Howto/Use_data_attributes) são escritos como no HTML com hífens.

</Pitfall>

### Dica profissional: Use um Conversor JSX {/*pro-tip-use-a-jsx-converter*/}

Converter todos esses atributos em markup existente pode ser tedioso! Recomendamos usar um [conversor](https://transform.tools/html-to-jsx) para traduzir seu HTML e SVG existentes para JSX. Os conversores são muito úteis na prática, mas ainda vale a pena entender o que está acontecendo para que você possa escrever JSX confortavelmente por conta própria.

Aqui está o seu resultado final:

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Tarefas de Hedy Lamarr</h1>
      <img 
        src="https://i.imgur.com/yXOvdOSs.jpg" 
        alt="Hedy Lamarr" 
        className="photo" 
      />
      <ul>
        <li>Inventar novos semáforos</li>
        <li>Ensaio de uma cena de filme</li>
        <li>Melhorar a tecnologia de espectro</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px }
```

</Sandpack>

<Recap>

Agora você sabe por que JSX existe e como usá-lo em componentes:

* Os componentes React agrupam a lógica de renderização junto com o markup porque eles estão relacionados.
* JSX é semelhante ao HTML, com algumas diferenças. Você pode usar um [conversor](https://transform.tools/html-to-jsx) se precisar.
* As mensagens de erro geralmente apontarão na direção certa para corrigir seu markup.

</Recap>

<Challenges>

#### Converter algum HTML em JSX {/*convert-some-html-to-jsx*/}

Este HTML foi colado em um componente, mas não é um JSX válido. Corrija-o:

<Sandpack>

```js
export default function Bio() {
  return (
    <div class="intro">
      <h1>Bem-vindo ao meu site!</h1>
    </div>
    <p class="summary">
      Você pode encontrar meus pensamentos aqui.
      <br><br>
      <b>E <i>fotos</b></i> de cientistas!
    </p>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

Seja para fazê-lo manualmente ou usando o conversor, a escolha é sua!

<Solution>

<Sandpack>

```js
export default function Bio() {
  return (
    <div>
      <div className="intro">
        <h1>Bem-vindo ao meu site!</h1>
      </div>
      <p className="summary">
        Você pode encontrar meus pensamentos aqui.
        <br /><br />
        <b>E <i>fotos</i></b> de cientistas!
      </p>
    </div>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

</Solution>

</Challenges>