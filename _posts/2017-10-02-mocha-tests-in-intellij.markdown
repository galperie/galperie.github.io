---
layout: post
title: Setting Up Mocha Tests in Intellij
date: '2017-10-02 02:30:02 -0400'
categories: mocha testing intellij
---

This is a quick guide on how to set up Mocha tests in Intellij so that you can **utilize the testing shortcuts**, **run individual or all tests quickly**, and even **use Intellij debug mode on your tests** :beetle:

You can most likely apply this to other javascript testing frameworks, so feel free to mess around and experiment with my instructions.

{:.highlightDarkEmphasis}
If you don't have a React project with mocha enabled you can use :point_right: [the one I quickly set up](https://github.com/galperie/mocha-tests-react) :point_left: for this blog post. It's based on this cool [project](https://github.com/LenaBarinova/react-mocha-example).

# Setting up Intellij

Make sure you have Intellij set up for a react project. This includes downloading and enabling the Node.js plugin and making sure you have JSX syntax hightling enabled. If you have any issues, I have some links to the Intellij docs at the bottom of the page that might help.

# &#10060; Trying to run a test

When I try to run any test using the Mac shortcut `ctrl + shift + R`, I get this error:

{% include image name="small-error.png" caption="Image of Intellij with an Error" %}

That's not good. Here is how you can make it pass:

# Configuring Intellij to run Mocha correctly :runner:

1. At the top of the screen (usually on the top left) look for the Runner navigation bar
{% include image name="runner.png" caption="Image of Intellij with an Error" %}
2. Click the drop down and choose `Edit Configurations`
3. On the left choose `Defaults` and scroll down until you see `Mocha` :coffee:
4. Choose the correct options for Node Interpreter, Working Directory, User Interface, and Extra Mocha Options:

* **Node Interpreter**: *generally the default Intellij chooses works*. It'll be where on your computer you have installed Node.js
* **Working Directory**: when you are running particular tests this should be the root directory of your project. But since we are editing the default settings leave it blank and let Intellij auto fill it in when you run your tests
* **User Interface**: this is what kind of testing style you're using. My project uses BDD. You can find out which one you use [here](http://mochajs.org/#interfaces).
* :star: **Extra Mocha Options**: this is the KEY. For my project if you look into the `package.json` and look at how I run my tests I have added extra parameters to my mocha command:

    ```
    "scripts": {
      "test": "mocha test/**/*-test.js --compilers js:babel-core/register --recursive"
    }
    ```
  For extra options I add in what I have above:

{% include image name="configure.png" caption="Image of Intellij Configuration" %}

  :warning: **MAKE SURE YOU REMOVE ALL TESTS YOU ALREADY RAN FROM THE RUNNER DROP DOWN. IF YOU DON'T THEY WILL NOT GET THE DEFAULT CONFIGURATION**

### &#9989; Rerun test

{% include image name="small-success.png" caption="Image of Intellij with an Error" %}

Try debugging, running all tests, or whatever else you'd like :smile:

### Links and other useful resources I've come across:

* [Mocha doc about interfaces](http://mochajs.org/#interfaces)
* [Intellij docs on setting us react syntax hightlighting](https://blog.jetbrains.com/webstorm/2015/10/working-with-reactjs-in-webstorm-coding-assistance/)
* [Intellij docs on how to set up react in Intellij](https://www.jetbrains.com/help/idea/react.html)
* [barebones react projec that has mocha tests](https://github.com/LenaBarinova/react-mocha-example)
