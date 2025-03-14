---
title: "Built-in React DOM Hooks"
---

<Intro>

O pacote `react-dom` contém Hooks que só são suportados para aplicações web (que rodam no ambiente DOM do navegador). Esses Hooks não são suportados em ambientes que não sejam navegadores, como aplicativos iOS, Android ou Windows. Se você está procurando por Hooks que são suportados em navegadores web *e em outros ambientes*, veja [a página de React Hooks](/reference/react). Esta página lista todos os Hooks no pacote `react-dom`.

</Intro>

---

## Form Hooks {/*form-hooks*/}

*Forms* permitem que você crie controles interativos para enviar informações. Para gerenciar formulários em seus componentes, use um desses Hooks:

* [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) permite que você faça atualizações na UI baseado no status de um formulário.

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