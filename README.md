# Hugo PaperMod i18n Starter (ES/EN)

## Pasos
1) Instalar Hugo **extended** y Git.
2) Clonar este proyecto o descomprimir el ZIP dentro de una carpeta vacía.
3) Agregar el tema PaperMod como submódulo:
```bash
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

4) Editar `hugo.toml` -> `baseURL = "https://TU_USUARIO.github.io/"` (o `https://TU_USUARIO.github.io/NOMBRE_REPO/` si es Project Page).
5) Probar local:
```bash
hugo server -D
```
- ES: http://localhost:1313/es/
- EN: http://localhost:1313/en/

6) Deploy con GitHub Pages (Actions):
- Habilitar Pages con **Source: GitHub Actions**
- Hacer push a `main`; el workflow `.github/workflows/deploy.yml` publicará a Pages.

## Crear nuevos posts
```bash
# ES
hugo new content/es/posts/mi-post.md
# EN
hugo new content/en/posts/my-post.md
```
Usá `archetypes/posts.md` de ejemplo para estructura estándar.
