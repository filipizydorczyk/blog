Most of my career, I was working with React, and I got very used to `jsx` format. However, it's typically built for building single-page applications, and sometimes I want to create static websites. Or I want to create smaller parts of HTML that I can use with different frameworks. You could work with HTML-formatted strings, but you are missing out on many features that JSX already has for its syntax, like autocomplete, coloring or error checking.

Because of that, I tried to find a way to generate static HTML instead of JavaScript code. Since React supports server-side rendering, you could use that. Write react code and render it to a string with `ReactDOMServer.renderToString`. In fact, that's what I am doing with my website (but using **Vue** instead). But this solution requires you to have whole **React** installed, which you are not gonna use. Kind of a waste of space.

Another solution is to use some bundlers instead. You can use whatever you want if this bundler provides you with the possibility to compile React files and customize them. To me, the most straight-forward was `esbuild`. Really, all I had to do was provide a short configuration.

```js
import * as esbuild from "esbuild";

esbuild.build({
    bundle: true,
    platform: "node",
    entryPoints: ["app.tsx"],
    outfile: "app.js",
    jsxFactory: "CustomJSX.createElement",
});
```

With this configuration `esbuild` is going to replace `React.createElement` with `CustomJSX.createElement`. You might have never seen it, but when you compile `.jsx` file all jsx elements are replaced with this function, which allows us to write our very own implementation. That's all. Not a single React dependency is needed.

But we still need to write this function and import it. This should not take a long time. Basic implementation could be

```ts
const CustomJSX = {
    createElement: (tag, properties, ...children): string => {
        return `<${tag}>${children}</${tag}>`;
    },
};
```

Complete implementation would not take much longer, and if you need some specific features for your project, you can also implement them here!

Then we just need to import it into every jsx file. It's not really different from what we do in react. When we create a React component we need to import

```js
import React from "react";
```

every single time. For our case, we just need a different import, like

```js
import { CustomJSX } from "./jsx";
```

It's up to you to define your API. But that's really it. You might even skip the import step if your bundler allows you to create some autoimport configuration. The only issue is that if you want to use typescript, you will have to create `.d.ts` file to prevent some typescript syntax errors. But that is a whole different story.
