# 📘 Mini Guia – Dicas úteis para trabalhar com o CMS Deco

---
## Tecnologias utilizados por sites deco

1. Preact
2. HTMX
3. Deco Framework
4. Tailwind CSS
5. deno
6. All pages are rendered by server with server side rendering.

## About deco framework and CMS

- The deco framework are based on blocks, blocks are functions with meaning and some particularity
- There some blocks like:
    - sections: which renders HTML with preact and can have inline loaders/actions;
    - loaders: which is a data loader and can connect to outside APIs;
    - actions: are like loaders but accessed via POST
    - pages: are like sections but has path and sections[] props
- The CMS content are written in JSON files inside ./deco/blocks/ folder. Also, each json should have the __resolveType property which is the path of the block will be executed with the props from the JSON by the deco framework. The name of the file is the id of block.
- You can compose blocks, like: You have a section/ProductShelf.tsx which receives a products: property that comes from a loader which return this type. The CMS json that represents  this is: { __resolveType: “site/sections/ProductShelf.tsx”, products: { __resolveType: “site/loaders/MyProduct.ts”, /* loader properties */ }, /* … section prorperties*/ } . 
- The interface of each block is compiled and converted into a formulary at deco CMS to be edited by the business user that changes the content of site.
