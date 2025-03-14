---
title: "<option>"
---

<Intro>

O [componente `<option>` do navegador](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) incorporado permite que você renderize uma option dentro de uma caixa [`<select>`](/reference/react-dom/components/select).

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

O [componente `<option>` do navegador](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) incorporado permite que você renderize uma option dentro de uma caixa [`<select>`](/reference/react-dom/components/select).

```js
<select>
  <option value="someOption">Some option</option>
  <option value="otherOption">Other option</option>
</select>
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<option>` suporta todas as [props de elementos comuns.](/reference/react-dom/components/common#props)

Adicionalmente, `<option>` suporta estas props:

* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#disabled): Um booleano. Se `true`, a option não será selecionável e irá aparecer esmaecida.
* [`label`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#label): Uma string. Especifica o significado da option. Se não especificado, o texto dentro da option é usado.
* [`value`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#value): O valor a ser usado [ao enviar o `<select>` pai em um formulário](/reference/react-dom/components/select#reading-the-select-box-value-when-submitting-a-form) se esta option estiver selecionada.

#### Ressalvas {/*caveats*/}

* React não suporta o atributo `selected` na `<option>`. Em vez disso, passe o `value` desta option para o pai [`<select defaultValue>`](/reference/react-dom/components/select#providing-an-initially-selected-option) para uma caixa de seleção não controlada, ou [`<select value>`](/reference/react-dom/components/select#controlling-a-select-box-with-a-state-variable) para um select controlado.

---

## Uso {/*usage*/}

### Exibindo uma caixa de seleção com options {/*displaying-a-select-box-with-options*/}

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