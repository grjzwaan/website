---
title: "Scaffolding in Eleventy"
description: "Automate everything: scaffolding in Eleventy and my own package Steiger."
date: 2020-07-06T16:27:46.409+02:00
draft: false
tags: [Static website]
layout: post
---
I've rebuild my website with [Elventy](https://www.11ty.dev/) and [Tailwind](https://tailwindcss.com/). One feature that I really liked about Hugo was to _scaffold_ new posts, so I've build my own [Steiger](https://github.com/grjzwaan/steiger).

All of my blogs have been static sites that are generated in a build process and then pushed to some hosting. Each post is simply a markdown page in the folder denoting it's date, so `posts/2020/07/06/scaffolding-in-eleventy.md` for this particular post. Each page also has metadata for the title, description, etc...

```markdown
---
title: "Scaffolding in Eleventy"
description: "Scaffolding in Eleventy"
date: 2020-07-06T16:27:46.409+02:00
draft: false
tags: [Elventy, static website]
---
```

With these things I'm really lazy. I do not want to copy/paste, create folders, etc. Just execute `scaffold posts/2020/07/06/scaffolding-in-eleventy.md` and I want to have a premade template where I fill in the content.

Unfortunately it was not included in Eleventy, so I rolled my own using [Chalk](https://github.com/chalk/chalk) for colored terminal output,  [Commander](https://github.com/tj/commander.js/) for the CLI, [Luxon](https://moment.github.io/luxon/) for dates, times (because the vanilla dates in Javascript breaks your brain), [Nunjucks](https://mozilla.github.io/nunjucks/) for templating.

The package has reached version `0.0.2` and it autogenerates the file you want, checks if it exists and gives the current datetime to the template. Over time as I use it, I'll probably add more features.
