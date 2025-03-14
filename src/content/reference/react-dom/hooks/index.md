---
title: "Hooks DOM do React embutidos"
---

<Intro>

O pacote `react-dom` contém Hooks que só são suportados para aplicações web (que são executadas no ambiente DOM do navegador). Estes Hooks não são suportados em ambientes que não sejam navegadores, como aplicações iOS, Android ou Windows. Se você estiver procurando por Hooks que são suportados em navegadores web *e outros ambientes*, consulte [a página React Hooks](/reference/react). Esta página lista todos os Hooks no pacote `react-dom`.

</Intro>

---

## Hooks de formulário {/*form-hooks*/}

*Formulários* permitem criar controles interativos para enviar informações. Para gerenciar formulários em seus componentes, use um destes Hooks:

* [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) permite que você faça atualizações na UI com base no status de um formulário.

```js
function Form({ action }) {
  async function increment(n) {
    return n + 1;
  }
  const [count, incrementFormAction] = useActionState(increment, 0);
  return (
    <form action={action}>
      <button formAction={incrementFormAction}>Count: {count}</button>
      <Button />
    </form>
  );
}

function Button() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending} type="submit">
      Submit
    </button>
  );
}
```