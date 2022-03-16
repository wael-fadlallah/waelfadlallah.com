---
title: "Building Efficient Ci/Cd Pipeline with GitHub actions and app engine"
date: 2022-03-14T22:05:57+04:00
draft: false
---

## Overview

One of the most important skills to master for becoming an experienced software developer is the automation of the repeatable tasks and I believe the most important one of them is the development process is self which includes merging, testing, and deploying the code and in the article, I want to show to efficiently build a CI/CD pipeline that does all of that

I found that GitHub actions to be extremely powerful and easy to use and the fact that GitHub is widely used by developers made it ideal to chose it as part of this pipeline

- **Prerequisites**:
  - Google cloud project with billing enabled
  - Google APIs to enable:
    - **App Engine**
    - **App Engine Admin API**
    - **Cloud Build API**
  - PNPM (optional, you can use any npm or yarn)

## About the example app

For demonstration purposes I have built a simple Todo list app with React and Typescript, I have used Vite as a development server and build tool because its simple, fast, and most importantly easily used with Vitest for testing

    > App structure

- Getting our hands dirty with the tests

{{< code language="javascript" title="Really cool snippet" id="1" expand="Show" collapse="Hide" isCollapsed="true" >}}
import { it, describe, expect, beforeEach, vi } from "vitest";
import App from "./App";
import TodoPanel from "./components/TodoPanel";
import { shallow } from "enzyme";
import { render, screen, userEvent, fireEvent } from "./utils/test-utils";
import "@testing-library/jest-dom/extend-expect";

describe("App Test", () => {
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

- Manual Deployment - setting up the app engine configurations
- Setting up GitHub actions
- Conclusion
