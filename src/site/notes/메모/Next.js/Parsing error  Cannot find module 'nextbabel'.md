---
{"dg-publish":true,"permalink":"//next-js/parsing-error-cannot-find-module-nextbabel/"}
---



[reference](https://stackoverflow.com/posts/68193974/timeline)

Create file called `.babelrc` in your root directory and add this code:

```
{
  "presets": ["next/babel"],
  "plugins": []
}
```

And in `.eslintrc`, replace the existing code with:

```
{
  "extends": ["next/babel","next/core-web-vitals"]
}
```

   Disabled SWC as replacement for Babel because of custom Babel configuration ".babelrc