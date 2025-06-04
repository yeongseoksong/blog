---
{"dg-publish":true,"permalink":"//frontend/react/vite-wsl-hotreload/"}
---

```js
import { fileURLToPath, URL } from "url";

import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  // === 해당 부분 추가 ===
  server: {
    watch: {
      usePolling: true,
    },
  },
 // === ===
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
});
```