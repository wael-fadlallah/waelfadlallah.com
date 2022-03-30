---
title: "Building Efficient CI/CD Pipeline with GitHub actions and app engine"
date: 2022-03-14T22:05:57+04:00
draft: false
---

- [Overview](#overview)
- [About the example app](#about-the-example-app)
- [Getting our hands dirty with the tests](#getting-our-hands-dirty-with-the-tests)
- [Manual Deployment](#manual-deployment)
- [Setting up GitHub actions](#setting-up-github-actions)

## Overview

One of the most important skills to master for becoming an experienced software developer is the automation of the repeatable tasks and I believe the most important one of them is the development process is self which includes merging, testing, and deploying the code and in the article, I want to show to efficiently build a CI/CD pipeline that does all of that

I found that GitHub actions to be extremely powerful and easy to use and the fact that GitHub is widely used by developers made it ideal to chose it as part of this pipeline

- **Prerequisites**:
  - Gcloud CLI <https://cloud.google.com/sdk/docs/install>
  - Google cloud project with billing enabled
  - Google APIs to enable:
    - **App Engine**
    - **App Engine Admin API**
    - **Cloud Build API**
  - PNPM (optional, you can use any npm or yarn)

## About the example app

For demonstration purposes I have built a simple Todo list app with React and Typescript, I have used Vite as a development server and build tool because its simple, fast, and most importantly easily used with Vitest for testing

```text
 📂 Todo-App
 ┣ 📂node_modules
 ┣ 📂src
 ┃ ┣ 📂components
 ┃ ┃ ┗ 📜TodoPanel.tsx
 ┃ ┣ 📂utils
 ┃ ┃ ┗ 📜test-utils.tsx
 ┃ ┣ 📜App.tsx
 ┃ ┣ 📜App.spec.ts
 ┃ ┣ 📜index.css
 ┃ ┣ 📜Types.ts
 ┣ 📜App.yaml
 ┣ 📜index.html
 ┣ 📜postcss.config.js
 ┣ 📜tailwind.config.js
 ┣ 📜tsconfig.json
 ┣ 📜vite.config.ts
 ┣ 📜.gitignore
 ┗ 📜package.json
```

## Getting our hands dirty with the tests

I have already install and configure Vitest and enzyme to save us time and to keep this post focuses about the CI/CD pipeline, the test is straight forward under the code I will explian what I did.

{{< code language="javascript" title="App.spec.tsx" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
import { it, describe, expect, beforeEach, vi } from "vitest";
import App from "./App";
import TodoPanel from "./components/TodoPanel";
import { shallow } from "enzyme";
import { render, screen, userEvent, fireEvent } from "./utils/test-utils";
import "@testing-library/jest-dom/extend-expect";

describe("Test App", () => {
  let wrapper: any;

  beforeEach(() => {
    wrapper = shallow(<App />);
  });

  it("Check if the component is mounted", () => {
    expect(App).toBeTruthy();
  });

  it("Check if the TodoPanel has been rendered", () => {
    expect(wrapper.find(TodoPanel).length).toBe(1);
  });

  it("Check if the button has been rendered", () => {
    expect(wrapper.find("button").length).toBe(1);
  });

  it("Check if todo item is added", async () => {
    render(<App />);
    const input = screen.getByTestId("todo-input") as HTMLInputElement;
    fireEvent.change(input, {
      target: { value: "Testing" },
    });
    userEvent.click(screen.getByRole("button"));
    expect(await screen.getByText(/Testing/i)).toBeInTheDocument();
  });
});
{{< /code >}}

Now we should be able to run vitest and the all test cases shloud pass

```shell
npx vitest
```

## Manual Deployment

If everything goes well now its the time to deploy the project, first lets start by setting up the app engine configurations, create a file in the root of the project and name it `app.yaml` and put the following

{{< code language="yaml" title="app.yaml" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
env: standard
runtime: nodejs16

handlers:
  - url: /assets
    static_dir: assets

  - url: /(.*\.(js|css))$
    static_files: assets/\1
    upload: assets/.*\.(json|ico|js|css)$
  - url: .*
    static_files: index.html
    upload: index.html

{{< /code>}}

now if you didn't install gcloud yet now is a good click [here](https://cloud.google.com/sdk/docs/install)

First login to your account

```shell
gcloud auth login
```

After you you select the account and allow google sdk to use your account you will notice in the terminal your project name, if its not the project that you want to use you can change it with this command 

```shell
gcloud config set project PROJECT_ID
```

Now that we are authenticated we can deploy the app with

```shell
gcloud app deploy
```

After the deploing finish you will see the deployed app url, copy it and past it in brower or hold `control` and click on it

## Setting up GitHub actions

{{< code language="yaml" title="main.yaml" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [master]
  pull_request:
    branches: [master]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  test:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
      - uses: pnpm/action-setup@v2.1.0
        with:
          version: 6.0.2
          run_install: |
            - recursive: true
              args: [--frozen-lockfile, --strict-peer-dependencies]
      - name: Run vitest
        run: pnpm run test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - uses: pnpm/action-setup@v2.1.0
        with:
          version: 6.0.2
          run_install: |
            - recursive: true
              args: [--frozen-lockfile, --strict-peer-dependencies]
      - name: Build the project
        run: pnpm run build

      - name: copy app.yaml file to the build directory
        run: cp app.yaml dist

      - name: cd to the build directory and deploy the project
        run: cd dist

#       - id: "auth"
      - uses: "google-github-actions/auth@v0"
        with:
          credentials_json: "${{ secrets.GCP_SA_KEY }}"

#       - id: "deploy"
      - uses: google-github-actions/deploy-appengine@main
        with:
          working_directory: dist
          version: 20220313t185328
{{< /code>}}

- Conclusion
