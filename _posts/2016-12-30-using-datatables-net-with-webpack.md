---
title: "Using DataTables.net with Webpack"
author: "Mitch Talmadge"
date: 2016-12-30T19:02:31.000-0700
last_modified_at: 2018-05-25T21:04:58.225-0600
tags: ["programming"]
description: "Getting DataTables.net to work with Webpack can be a pain! I figured it out so you don't have to."
---

Recently I found a need for installing DataTables.net in my Angular 2 application, which is packaged using Webpack. Unfortunately, I faced many errors while trying to install the library using NPM and importing DataTables.net into my `vendor.ts` file.

As it turned out, DataTables.net can be used with AMD and CommonJS importers, but Webpack allows loading of either one. AMD is the default loader in DataTables.net, so it was being loaded using the AMD method, when I was trying to load with CommonJS. The solution is to use the `imports-loader` plugin for Webpack to disable the AMD loading support for DataTables.net.

Add this to your `webpack.config.js` :
```js
module: {
    loaders: [
        ...
        {
            test: /datatables.net.*/,
            loader: 'imports?define=>false'
        }
        ...
}
```

[On the imports-loader github page](https://github.com/webpack/imports-loader#disable-amd), it says:

> There are many modules that check for a `define` function before using CommonJS. Since webpack is capable of both, they default to AMD in this case, which can be a problem if the implementation is quirky. 
>
> Then you can easily disable the AMD path by writing 
>
> `imports-loader?define=>false` 

Finally, to import the packages into your `vendor.ts` :
```typescript
require("datatables.net")(window, $);
require("datatables.net-bs")(window, $);
require("datatables.net-responsive")(window, $);
require("datatables.net-responsive-bs")(window, $);
require("datatables.net-select")(window, $);
```

Now DataTables.net loads up perfectly.

