---
author: ["Edward Pasenidis"]
title: "B01lersCTF - Trouble at the SPA (Web)"
date: "2025-04-21"
description: "Fix your routes!"
tags: ["b01lersctf", "writeup", "ctf"]
categories: ["Writeups"]
ShowToc: false
---

## Overview

This whitebox challenge is running on a Vite/React server as a SPA (as the title of challenge suggests).

## Reviewing the code

After taking a look at the code, we notice that the flag is located inside a JSX/TSX component called `Flag()` but it's not directly exposed on the GitHub Pages instance, this is a common misconfiguration mistake. It happens because web servers like Apache and Nginx don't handle routing the way React Router does (you can read more [here](https://stackoverflow.com/questions/27928372/react-router-urls-dont-work-when-refreshing-or-writing-manually/57066837#57066837)).

```js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter, Routes, Route } from 'react-router';

// Pages
import App from './App.tsx';
import Flag from './Flag.tsx';

import './index.css';


createRoot(document.getElementById('root')!).render(
    <StrictMode>
        <BrowserRouter>
            <Routes>
                <Route index element={<App />} />
                <Route path="/flag" element={<Flag />} />
            </Routes>
        </BrowserRouter>
    </StrictMode>
);
```

We can see that the `Flag` component has been imported and used inside a `<Route>`, which means that Webpack included it in the bundle when the developer was deploying it to GitHub Pages.

`<BrowserRouter>` tells us that the Router is listening to the browser's history. So we can manually navigate to `/flag` like this:

```js
window.history.pushState({}, '', '/flag');
window.dispatchEvent(new PopStateEvent('popstate'));
```

This will, update the URL to `/flag`, and then trigger React Router to re-render the virtual DOM by dispatching an event.

You can read more about BrowserRouter [here](https://reactrouter.com/6.30.0/router-components/browser-router#basename)


## Epilogue
Thanks for reading! Hope you had fun with this challenge