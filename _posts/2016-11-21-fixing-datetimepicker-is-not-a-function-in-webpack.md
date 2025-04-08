---
title: "fixing `$(…).datetimepicker is not a function` in webpack"
author: "Mitch Talmadge"
date: 2016-11-21T17:03:09.000-0700
last_modified_at: 2018-05-25T21:06:28.806-0600
tags: ["programming","webpack","bootstrap","javascript","jquery"]
description: "I struggled to get a Bootstrap Date-Time Picker to work with Webpack. Luckily, the solution is super easy!"
---

I attempted to install [Eonasdan’s Bootstrap DateTimePicker](https://github.com/Eonasdan/bootstrap-datetimepicker) using webpack, but when I tried to use it, I got this error:

`TypeError: $(…).datetimepicker is not a function`

The problem is that datetimepicker is trying to use its own version of jquery, while the version of jquery used by _everything else_ on the web application was a different version… So the function didn’t exist for the version I was using. The fix is easy — I just added this to the `resolve` section of my `webpack.config.js`
```
alias: {
    // Force all modules to use the same jquery version.
    'jquery': path.join(__dirname, 'node_modules/jquery/src/jquery')
}
```

That’s it! (Thanks to the people on [this GitHub issue](https://github.com/Eonasdan/bootstrap-datetimepicker/issues/1319) for the fix)

