
## 🔍 Visualizar o JSON de uma Página

Cada página tem um arquivo de configuração JSON localizado em:

```
/.deco/blocks/{nome-do-arquivo}.json
```

Para descobrir o nome do arquivo da página atual, use no console do navegador:

```js
window.LIVE.page.id
```

Esse ID corresponde ao nome do JSON que está sendo usado na página.

---

## ⚡ Lazy Loading via Código (`deco` + `htmx`)

Você pode carregar partes de um componente de forma assíncrona usando `useSection` e `htmx`.

### Passo a passo:

1. Use `useSection()` para gerar a URL do `hx-get`.
2. Use a prop `renderHiddenPart` para indicar o que deve ser carregado.

### Exemplo – Lazy loading no componente `Header`:

```ts
interface Props {
  renderHiddenPart: "left-menu" | "searchbar";
}

function Header({ renderHiddenPart }: Props) {
  if (renderHiddenPart === "left-menu") return <LeftMenuComponent />;
  if (renderHiddenPart === "searchbar") return <Searchbar />;

  return (
    <header>
      <button
        hx-get={useSection({ props: { renderHiddenPart: "left-menu" } })}
        hx-swap="#left-menu-placeholder"
      />
      <div id="left-menu-placeholder" class="hidden" />

      <button
        hx-get={useSection({ props: { renderHiddenPart: "searchbar" } })}
        hx-swap="#searchbar-placeholder"
      />
      <div id="searchbar-placeholder" />
    </header>
  );
}
```

---

## 🧩 Lazy Loading de uma Section inteira via CMS

Ideal para sections que estão **abaixo da dobra (below the fold)**.

### Antes (sem lazy):

```json
{
  "sections": [
    { "__resolveType": "...", // component props }
  ]
}
```

### Depois (com lazy):

```json
{
  "sections": [
    {
      "__resolveType": "website/sections/Rendering/Lazy.tsx",
      "section": {
        "__resolveType": "...",
        // component props
      }
    }
  ]
}
```

> NOTE: É Possível criar exportar um component LoadingFallback na section, que será rendrizado enquanto carrega o componente de section (default function).

---
