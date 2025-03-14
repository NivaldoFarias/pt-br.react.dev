---
title: "<option>"
---

<Intro>

O [componente `<option>` do navegador](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) integrado permite que você renderize uma opção dentro de uma caixa [`<select>`](/reference/react-dom/components/select).

```js
<select>
  <option value="someOption">Some option</option>
  <option value="otherOption">Other option</option>
</select>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<option>` {/*option*/}

O [componente `<option>` do navegador](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) integrado permite que você renderize uma opção dentro de uma caixa [`<select>`](/reference/react-dom/components/select).

```js
<select>
  <option value="someOption">Some option</option>
  <option value="otherOption">Other option</option>
</select>
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<option>` suporta todos os [props comuns de elementos.](/reference/react-dom/components/common#props)

Além disso, `<option>` suporta esses props:

* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#disabled): Um booleano. Se `true`, a opção não poderá ser selecionada e aparecerá esmaecida.
* [`label`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#label): Uma string. Especifica o significado da opção. Se não for especificado, o texto dentro da opção é usado.
* [`value`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#value): O valor a ser usado [ao enviar o `<select>` pai em um formulário](/reference/react-dom/components/select#reading-the-select-box-value-when-submitting-a-form) se esta opção for selecionada.

#### Ressalvas {/*caveats*/}

* React não suporta o atributo `selected` em `<option>`. Em vez disso, passe o `value` desta opção para o [`<select defaultValue>`](/reference/react-dom/components/select#providing-an-initially-selected-option) pai para uma caixa de seleção não controlada, ou [`<select value>`](/reference/react-dom/components/select#controlling-a-select-box-with-a-state-variable) para um select controlado.

---

## Uso {/*usage*/}

### Exibindo uma caixa de seleção com opções {/*displaying-a-select-box-with-options*/}

Renderize um `<select>` com uma lista de componentes `<option>` dentro para exibir uma caixa de seleção. Dê a cada `<option>` um `value` representando os dados a serem enviados com o formulário.

[Leia mais sobre como exibir um `<select>` com uma lista de componentes `<option>`.](/reference/react-dom/components/select)

<Sandpack>

```js
export default function FruitPicker() {
  return (
    <label>
      Pick a fruit:
      <select name="selectedFruit">
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="orange">Orange</option>
      </select>
    </label>
  );
}
```

```css
select { margin: 5px; }
```

</Sandpack>
```