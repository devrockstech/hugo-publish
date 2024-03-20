# Hugo based static website from markdown 

To setup follow below steps 
1. Create Hugo site on local 
```
hugo new site <PROJECT>
cd <PROJECT>
```
2. Initialise git repo & add `hugo-book` as submodule
```
git init
git submodule add https://github.com/alex-shpak/hugo-book themes/hugo-book
```
3. Add or Update theme in `hugo.toml`
```
theme = 'hugo-book'
```
4. Start the hugo server to run on local
```
hugo server --minify --theme hugo-book
```
5. Create directories
```
<PROJECT>
  - content
    - docs
      - _index.md
      - <sub directory>
        - _index.md
        - <markdown files>
```
6. Add `_index.md` wherever you need a directory-level page is required
7. Add optional macros at the top of markdown files.
Example
```
---
title: Introduction
type: docs
bookFlatSection: true
weight: 10
---
```
|Macro|Description|
|-----|-----------|
|title| title of the page |
|weight| weight determines the order of page in navigation menu |

Refer more configurations at [Page configurations](https://github.com/alex-shpak/hugo-book/tree/master?tab=readme-ov-file#page-configuration)
