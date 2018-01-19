---
title: "How to achieve 100% test coverage in React"
author: "Vivek Kalyan"
date: "30/09/2017"
---

During my internship at Gametize, I spent a considerable portion of my effort ensuring that the application being created was well tested. The two building blocks for testing I used were [Jest](https://facebook.github.io/jest/) and [Enzyme](http://airbnb.io/enzyme/). Jest is a testing framework for JavaScript applications, while Enzyme is a utility to easily mount React components.

There are many good guides for Jest and Enzyme (their documentations are a great starting point) I will instead share some non-obvious tips I discovered after using them for 4 months.

Note: this assumes you used `create-react-app` to create your project and thus already have basic testing functionality such as watch available. If not, that should be the first thing to setup so that you can instant feedback on changes.

## Simulate Events

Enzyme allows us to to programmatically simulate browser events. For example, we can test that out input fields handles keyboard inputs correctly (eg. submit form on pressing Enter). We can mimic these events in our tests.

```javascript
wrapper.simulate("keyPress", { key: "Enter" });
```

Also works for mouse events such as `click`, `mouseDown` or any [browser events](https://developer.mozilla.org/en-US/docs/Web/Events).

## Selectors

HTML elements and CSS classes are a great way to find the components we want in the DOM. We can do even more powerful selections by using combinations of them with contextual selectors.

* `wrapper.find("p.foo")` returns all `<p>` nodes with class `foo`

* `wrapper.find("div span")` returns all `<span>` nodes inside `<div>` elements

* `wrapper.find("div > span")` returns all `<span>` nodes where the parent is a `<div>` element

* `wrapper.find("div + .foo")` returns all nodes with class `foo` that are directly after a `<div>` element

* `wrapper.find("div ~ .foo")` returns all `div` nodes that are preceded by an element with class `foo`

The above contextual selectors work not only with classes and elements but also with id and attributes. Enzyme's [official documentation on selectors](http://airbnb.io/enzyme/docs/api/selector.html) has the full list of supported selectors.

## Jest Spy

We can use `jest.spyOn` to track calls to methods in components. But what if we have a javascript function that we want to test that it is being called?

```javascript
// foobar.js

function foo()

export default foo;
```

Then, in our test:

```javascript
import * as foobar from 'path/to/foobar'

const spy = jest.spyOn(foobar, "default");
expect(spy).toHaveBeenCalled();
```

## Javascript global variables

Sometimes components need access global variables (eg. `window.localStorage`) of browsers.

For example, when the application is logging out, certain variables in the local storage of the browser needs to be changed. We simply create a mock function that allows us to test if the function is called. (The function is also tested separately for correctness)

```javascript
it("logs out on click", () => {
  function mockStorage() {
    return {
      setItem: jest.fn()
    };
  }
  window.localStorage = mockStorage();
  const wrapper = shallow(<Logout {...props} />);
  wrapper.simulate("click", { preventDefault: () => {} });
  expect(window.localStorage.setItem).toHaveBeenCalledTimes(1);
});
```

For changing url in tests (simulating navigation), we need a function to create an object that has all the required attributes filled in for us based on the url we provide. Adapted from this [comment](https://github.com/facebook/jest/issues/890#issuecomment-298594389) to an issue on Github.

```javascript
const setURL = url => {
  const parser = document.createElement("a");
  parser.href = url;
  [
    "href",
    "protocol",
    "host",
    "hostname",
    "origin",
    "port",
    "pathname",
    "search",
    "hash"
  ].forEach(prop => {
    Object.defineProperty(window.location, prop, {
      value: parser[prop],
      writable: true
    });
  });
};
```

## APIs

Splitting components to presentational and container components as outlined [here](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), results in a very clean separation of concerns. Testing container components however is not a trivial task, especially if they are making API calls.

For example, our login container makes a POST request with login details to `/login.json`, which returns `200` on success and `500` on failure. We want to test that the container handles the respective responses correctly.

We are using `superagent` to make requests and `superagent-mock` to mock responses. To create a mock api we define how the response should be structured based on the request. More details about the structure for mocking API can be found [here](https://github.com/M6Web/superagent-mock).

```javascript
// mockApi/login
const login = code => [
  {
    pattern: ENDPOINT + "(.*)?",
    fixtures: function(match, params, headers, context) {
      const n = match[1].indexOf("?");
      match[1] = match[1].substring(0, n !== -1 ? n : match[1].length);
      if (match[1] === "login.json") {
        return {
          code: code,
        };
      }
    },
    post: function(match, customData) {
      return {
        body: customData
      };
    }
  }
];

export default login;
```

In our test file, we instruct the DOM to use the mock API before triggering the container to make the API request (through `handleSubmit()`)

```javascript
it("handles submit function (API FAILURE)", () => {
  const mockApi = superagentMock(request, login(500));
  const wrapper = shallow(<LoginContainer {...props} />);
  wrapper.instance().handleSubmit({ preventDefault: () => {} });
  mockApi.unset();
  // check container handles failure correctly
});
```

## That elusive 5%

Sometimes some functions/branches in JS are practically impossible to test. And it is important not to waste time writing trivial tests. But, an coverage that shows 95% is much less satisfying than one that shows 100%. We can use `/*istanbul ignore next*/` to ignore the next block in your code. Istanbul is the coverage tool used by Jest, and it can be [granularly controlled](https://github.com/gotwarlost/istanbul/blob/master/ignoring-code-for-coverage.md) over what it ignores.


## Running tests before commits

One of the most frustrating things that can happen is making a minor change, committing it and realising that you broke the tests. Initially, I used git's [pre-commit](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) hooks to run tests before commits. However, the pre-commit lies in the `.git/` folder which is always ignored by version control. This means that the hook will not be present for any co-worker cloning the repository.

Fortunately, there is a node package [husky](https://github.com/typicode/husky), which allows for the hooks to be reproduced easily. After installing `huksy` and `cross-env`, all we need to do is to add this line to scripts in `package.json`

```javascript
// package.json
{
  "scripts": {
    "precommit": "cross-env CI=true react-scripts test --env=jsdom --findRelatedTests",
  },
}
```

Note: to use this in conjunction with [Prettier](https://github.com/prettier/prettier) (an opinionated code formatter to standardise styles before commits)

```
// package.json
{
  "scripts": {
    "precommit": "lint-staged",
    "test:staged": "cross-env CI=true react-scripts test --env=jsdom --findRelatedTests"
  },
  "lint-staged": {
    "src/**/*.js": [
      "prettier --write",
      "git add",
      "test:staged"
    ]
  }
}
```

### Closing remarks

Tests can seem to be a very daunting task, especially with the seemingly contorted steps one has to go through to test some of the components. I never really enjoyed writing tests (I just want to build cool stuff!!) but I was immensely grateful when I had to refactor a large portion of the project 3 months into my internship. What could have possibly been an week long exercise was done in less than a day due to the confidence that the tests provided.

PS: I never actually achieved 100% test coverage.