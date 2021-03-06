Testing a React/Redux application can be tricky. The balance of trying to not test the framework without missing out on key behavior is hard. Over the last few months my team and I tried a few ways to test our application. Tested out the balance of too much and too little.

We went through cycles of thinking of what to test. First on how to test the Redux cycle to ensure that our React components were behaving properly and finally testing the behavior of our app as a whole.

Here's what I found:

As a first approach our team looked at the application as what it was: a React/Redux app. Thus, we broke down the Redux and React lifecycle and behaviors and thought about each section and what we wanted to ensure as our app went through its life cycle.

The general React/Redux flow we identified:

Container gives state to the ---> Small component that displays things to the user ---> and when something happens (user interaction or something in the react lifecycle) ---> the container dispatches an action ---> the reducer handles the action and creates new state ---> Container gives state to the ---> Small component.... etc etc

This was in general what our app was doing. So we broke that down to which part of our app did what:

  * Test that the container gave the correct props to the component, given the state was in a certain way
      ```
          describe("(Container) AwesomeContainer", () => {
            it("should give CoolComponent props", () => {
                const store = createStore(...);
                const awesomeContainer = shallowRender(AwesomeContainer, store);

                const coolComponent = awesomeContainer.find(CoolComponent);

                expect(coolComponent).to.have.props("thing", valueA);
            });
          });
      ```

  * Test the component rendered/handled the props in the way we expected
  ```
      describe("(Component) CoolComponent", () => {
        it("should give CoolComponent props", () => {
            const store = createStore(...);
            const coolComponent = shallowRender(CoolComponent, store);

            const textBox = awesomeContainer.find(TextBox);

            expect(textBox).to.exist;
        });
      });
  ```

  * Test that the container reacted to user interactions correctly (given this button was clicked, then a new page was navigated to; given a form was submitted, an action was dispatched)
  * Test that the container's lifecycle methods [link to lifecycle methods in react] did the behavior expected
  * Test the reducer initialized state correctly, gave the correct new state based on actions, and the selectors we created returned the state as we expected.

  This was what we wanted to test in general.
