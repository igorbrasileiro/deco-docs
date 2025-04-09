# 📘 Mini Guia – Como otimizar sites deco
--- 

## Identificar sections que tem tamanho de HTML maior do que 30kb

- Identifique quais os componentes preact que foram renderizados pela página. Observe o data-manifest-key das HTML Tag section

> NOTA: A section website/sections/Rendering/Lazy.tsx é um "wrapper", pegue o data-manifest-key da section filha imediata. Exemplo: <section data-manifest-key="website/sections/Rendering/Lazy.tsx"><section data-manifest-key="caminho/do/arquivo.tsx">...</section></section>

## 🔍 Visualizar o JSON de uma Página

Cada página tem um arquivo de configuração JSON localizado em:

```
/.deco/blocks/{id-da-pagina}.json
```

Para descobrir o nome do arquivo da página atual, use no console do navegador:

```js
window.LIVE.page.id
```

Esse ID corresponde ao nome do JSON que está sendo usado na página.

## ⚡ Lazy Loading via Código (`deco` + `htmx`)

Você pode carregar partes de um componente de forma assíncrona (async render, lazy loading) usando `useSection` (from package "@deco/deco/hooks") e `htmx`.

### Quando utilizar?

Utilize quando tiver partes da página que é exibido somente dado uma iteração do usuário na página, como click, hover, resultado de uma outra ação. Exemplo: Um modal, cujo acionador é de um botão. Outro exemplo: O menu lateral que abre quando o usuário clica.

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


## 🧩 Lazy Loading de uma Section inteira via CMS

> NOTA: Se possuir acesso a ferramenta (tool) para navegar por páginas, NAVEGE e identifique quais sections estão abaixo da dobra (below the fold).

Ideal para sections que estão **abaixo da dobra (below the fold)**.

Utilize a section website/sections/Rendering/Lazy.tsx que é responsável por implementar o async rendering/lazy loading de uma section. A section Lazy.tsx adiciona um element html como placeholder com a tag HTMX hx-get que pedira ao servidor a section async renreder/lazy loaded.

### Antes (sem lazy):

```json
// page json with CMS Content
{
  "sections": [
    { "__resolveType": "...", // component props }
  ]
}
```

### Depois (com lazy):

```json
{
// page json with CMS Content
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

> NOTA: É Possível criar exportar um component LoadingFallback na section, que será rendrizado enquanto carrega o componente de section (default function).

## Checking if Async Render Sections Are Being Cached

Each async rendered section via Rendering/Lazy.tsx section or `useSection`, does a request to `/deco/render` via HTMX hx-get.

Cached request answers with `cache-control: max-age=60 (or any value differente from zero) ...other values`.

## Bloquear querystrings do async render

- Liste as query strings que estão afetando o async render;
- Adicione a lista em um array no arquivo /blockedQs.ts e exporte o array;
- Importe as querystrings do arquivo /blockedQs.ts, no arquivo /apps/site.ts, e passe como argumento da função unstable_blockUseSectionHrefQueryStrings que é importada do pacote “@deco/deco/hooks”
