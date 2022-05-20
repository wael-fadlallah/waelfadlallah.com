---
title: "Building Simple CI/CD Pipeline with GitHub Actions and App Engine"
date: 2022-03-30T22:05:57+04:00
draft: false
---

## Overview

One of the most important skills for becoming an experienced software developer is the automation of the repeatable tasks and I believe the most important one of them is the deployment process itself which includes merging, testing and deploying the code in the article, I want to show to efficiently build a CI/CD pipeline that does all of that

I found GitHub actions to be extremely powerful and easy to use and the fact that GitHub is widely used by developers made it ideal to choose it as part of this pipeline

- **Prerequisites**:
  - Gcloud CLI <https://cloud.google.com/sdk/docs/install>
  - Google cloud project with billing enabled
  - Google APIs to enable:
    - **App Engine**
    - **App Engine Admin API**
    - **Cloud Build API**
  - PNPM (optional, you can use npm or yarn)

## About the example app

For demonstration purposes I have built a simple Todo list app with React and Typescript, I have used Vite as a development server and build tool because its simple, fast, and most importantly easily used with Vitest for testing

App structure

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

I have already installed and configured Vitest and enzyme to save us time and to keep this post focused on the CI/CD pipeline, the test is straightforward.

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
We are setting the environment to `standard` and the runtime to `nodejs16` you can learn more about app engine environments by click [here](https://cloud.google.com/appengine/docs/the-appengine-environments),And the `handlers` are URL patterns and descriptions of how they should be handled, for exapmle am making `assets` a static dirctory and mapping very request for a JS or CSS file to it, and finally I'm mapping any other request to `index.html` file

Now if you didn't install gcloud yet now is a good time, click [here](https://cloud.google.com/sdk/docs/install) for instructions.

First login to your account

```shell
gcloud auth login
```

Select the account and allow Google SDK to use your account you will notice the project name in the terminal, if it's not the project that you want to use you can change it with this command.

```shell
gcloud config set project PROJECT_ID
```

Now that we are authenticated we can deploy the app with:

```shell
gcloud app deploy
```

After the deploying finish, you will see the deployed app URL, copy it and past it in the browser or hold `control` and click on it

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

      - uses: google-github-actions/auth@v0
        with:
          credentials_json: "${{ secrets.GCP_SA_KEY }}"

      - uses: google-github-actions/deploy-appengine@main
        with:
          working_directory: dist
          version: 20220313t185328
{{< /code>}}

We have two jobs the first one installs PNPM and run `npx vitest`, and the second one is where the magic is happening so let's explain it deeper.

Before deploying we need to make sure the test is finished, that is why we added `needs: test` to the job, then we install PNPM and build the project.

If you notice that the `app.yaml` is at the root of the project that is why after building the project we need to copy `app.yaml` to `dist` which is the build directory.

Finally, inside the `dist` directory, we simply use `google-github-actions/auth@v0` action to authenticate and `google-github-actions/deploy-appengine@main` to deploy the project

## Conclusion

CI/CD supercharge software development flow and is very important to prevent problems by abstracting the deployment and stop deploying bad code by adding tests to the pipeline, Github actions provide a clean and easy way to build them directly into the project.
