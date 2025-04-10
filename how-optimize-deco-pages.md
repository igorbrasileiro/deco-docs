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

## 🧩 Async Rendering de uma Section inteira via CMS

O Async Render é uma maneira de fazer o carregamento tardio de uma section. A section de Lazy (website/sections/Rendering/Lazy.tsx) é responsável por implementar o mecanismo
de não renderizar a section no servidor e fazer um request para o servidor pedindo a section. Caso a section exporte um componente chamado `LoadingFallback`, ele será renderizado
pelo Lazy no servidor, serve para colocar spinners ou pequenas partes de section.

### Quando utilizar?

Ideal para sections que estão **abaixo da dobra (below the fold)**.

> NOTA: Se possuir acesso a ferramenta (tool) para navegar por páginas, NAVEGE e identifique quais sections estão abaixo da dobra (below the fold).

### Passo a Passo

Edite o conteudo da página do CMS, compondo a section Lazy com a section que já existe na página.

#### Antes (sem async render):

```json
// page json with CMS Content
{
  "sections": [
    { "__resolveType": "...", // component props }
  ]
}
```

#### Depois (com async render):

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

## ⚡ Async Render de componente via Código (`deco` + `htmx`)

Você pode pedir para o servidor renderizar partes de um componente `useSection` (from jsr package "@deco/deco/hooks". Github link https://github.com/deco-cx/deco/blob/main/hooks/mod.ts) e `htmx`.

### Quando utilizar?

Utilize quando tiver partes da página que é exibido somente dado uma iteração do usuário na página, como click, hover, resultado de uma outra ação. Exemplo: Um modal, cujo acionador é de um botão. Outro exemplo: O menu lateral que abre quando o usuário clica.

### Passo a passo para implementar:

1. Use `useSection()` para gerar a URL do `hx-get`.
2. Use a prop `renderHiddenPart` para indicar o que deve ser carregado.

### Exemplo – Renderizando partes do componente `Header` através de iteração do usuário:

```ts
interface Props {
 // ... outras propriedades
  /** @ignore deixar invisível do CMS */
  renderHiddenPart: "left-menu" | "searchbar";
}

function Header({ renderHiddenPart }: Props) {
  if (renderHiddenPart === "left-menu") return <LeftMenuComponent />;
  if (renderHiddenPart === "searchbar") return <Searchbar />;

  return (
    <header>
      <button
       // Não precisa passar as outras propriedades por que o servidor consegue recupera-las. Somente passar propriedades que serão sobreescritas
        hx-get={useSection({ props: { renderHiddenPart: "left-menu" } })}
        hx-swap="#left-menu-placeholder"
      />
      <div id="left-menu-placeholder" class="hidden" />

      <button
       // Não precisa passar as outras propriedades por que o servidor consegue recupera-las. Somente passar propriedades que serão sobreescritas
        hx-get={useSection({ props: { renderHiddenPart: "searchbar" } })}
        hx-swap="#searchbar-placeholder"
      />
      <div id="searchbar-placeholder" />
    </header>
  );
}
```

## Checando se o Async Render está bem cacheado

Toda section renderizada através do Async Render ou useSection faz o um request no browser para `/deco/render?<query parameters>`. Para uma boa UX a resposta do servidor precisa conter o header de cache-control e com tempo maior do que `0`, para habilitar o cache da CDN e navegador, fazendo com que a resposta da renderização da section seja rápida e tenha uma boa UX.

O request header de cache-control: `cache-control: max-age=60 (or any value differente from zero) ...other values`.

## Bloquear querystrings do async render

- Liste as query strings que estão afetando o async render;
- Adicione a lista em um array no arquivo /blockedQs.ts e exporte o array;
- Importe as querystrings do arquivo /blockedQs.ts, no arquivo /apps/site.ts, e passe como argumento da função unstable_blockUseSectionHrefQueryStrings que é importada do pacote “@deco/deco/hooks”
