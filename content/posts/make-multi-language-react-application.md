---
title: "Make Multi Language React Application"
date: 2022-04-04T23:23:50+04:00
draft: false
---

- [Overview](#overview)
- [About the example app](#about-the-example-app)
- [Install and configure the dependencies](#install-and-configure-the-dependencies)
- [Adding the translations](#adding-the-translations)
- [Configure Local Storage](#configure-local-storage)
- [Managing assets](#managing-assets)
- [Conclusion](#conclusion)

## Overview

Being a software engineer in the middle east most of my projects is multi language and making multi language web application can somtimes become challenging, especially when working with a right to left languages like arabic and its not just about the translations and adjusting the direction we need to architecture the whole project in meanful way to make things easier

## About the example app

For the purposes of this article I have built a small one page project with react and tailwind you can grap the source code from [here](https://github.com/wael-fadlallah/blog-example-multilingual.git), switch to `start` branch to follow along if you want

## Install and configure the dependencies

We will use [I18next](https://react.i18next.com/getting-started) which is internationalization-framework written in and for JavaScript

```shell
yarn add i18next react-i18next
```

## Adding the translations

Beside our `main.tsx` lets create `i18n.ts` which will hold our localizing config and add the following

{{< code language="yaml" title="app.yaml" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
import i18n from "i18next";
import en from "./locale/en";
import ar from "./locale/ar";
import { initReactI18next } from "react-i18next";

const resources = {
...en,
...ar,
};

i18n.use(initReactI18next).init({
  resources,
  lng: "en", // default language
  interpolation: {
  escapeValue: false, // react already safes from xss
  },
});

export default i18n;

{{< /code >}}

## Configure Local Storage

## Managing assets

## Conclusion
