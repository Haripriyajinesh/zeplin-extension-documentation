## Tutorial

This short tutorial will guide you through your first Zeplin extension.

### Installing zem

[zem](https://github.com/zeplin/zem), Zeplin extension manager, is a command line tool that lets you quickly create and test  extensions.

If you have npm 5.2+ installed (or node), you can run zem directly to create the extension:

Let's create it in a directory called `hello-layer`.

```sh
npx zem create hello-layer
```

If you have an older version of npm, you can install zem globally and run it right after:

```sh
npm install -g zem
zem create hello-layer
```

### Project structure

Under the `hello-layer` directory, zem should generate the initial project structure:

```
hello-layer
├── README.md
├── package.json
└── src
    └── index.js
```

Open the project in your favorite text editor and let's go over the generated documents.

#### `package.json`

Everything you, and Zeplin, need to know about an extension, along with its dependencies are defined in this JSON document. It functions as a regular npm `package.json` document, also containing Zeplin-specific properties.

Here's what zem generates by default:

```
{
    "name": "hello-layer",
    "description": "Sample description.",
    "version": "0.1.0",
    "scripts: {
        …
    },
    "devDependencies": {
        "zem": …
    },
    "zeplin": {
        "displayName": "hello-layer",
        "platforms": ["web"]
    }
}
```

Name, description and version properties are pretty self explanatory, they work just like they do in npm. Dependencies  include zem by default, letting you run scripts on the extension, which we'll get to in a bit. You can also add other npm packages as dependencies, as long as they work in the browser environment.

Notice that there's an additional property called `zeplin`, where Zeplin-specific properties live:

- `displayName` is the name of the extension displayed in the apps, not to be confused with the root `name` property, which is npm specific.
- `platforms` define which platforms this extension should be available to.

Let's update `displayName` to “Hello Layer” and `platforms` to cover all the platform types, for the purpose of this tutorial:

```
    …
    "zeplin": {
        "displayName": "Hello Layer",
        "platforms": ["web", "ios", "android", "osx"]
    }
    …
```

Normally, a Swift extension for example, should only define `ios` and `osx` as the platforms.

#### `index.js`

`index.js` is the default entry point for the extension, a JavaScript document, an ECMAScript module that exports functions corresponding to different actions in Zeplin.

By default zem adds all the functions that Zeplin supports to the module. For this tutorial, we're only interested in generating snippets from layers so we can delete all the other functions and their export statements. Every time a layer is selected, Zeplin should call this `layer` function and display the result.

Let's return a simple string from this function. `index.js` should look like this at this point:

```js
function layer(context, layer) {
    return `Hello ${layer.name}.`;
};

export default {
    layer
};
```

Time to give this extension a try.

### Testing

zem also lets you run an extension in the command line. You can execute functions with sample data and observe the results, without running the extension inside Zeplin.

Make sure that you're at the project directory and let's run the `exec` script for the `layer` function:

```sh
npm run exec -- layer
```

zem executes the function on a bunch of sample layers and outputs the results. The output for the first layer should look something like this:
```
Hello Layer with border.
```

### Displaying code snippets

Instead of returning a string directly from our `layer` function, we can return an object to display a code snippet.

Let's update it to display a JSON instead:

```js
function layer(context, layer) {
    const containerType = "styleguide" in context ? "styleguide" : "project";
    const object = {
        "layerName": layer.name,
        [`${containerType}Name`]: context[containerType].name
    };

    const JSONString = JSON.stringify(object, null, 2);

    return {
        code: JSONString,
        language: "json"
    };
};
```

We now create an object first and turn it into a JSON string using `JSON.stringify`. Instead of returning the string directly though, we return an object to let Zeplin (no pun intended) know that we want this string to be highlighted as a JSON.

Here's what we get, if we run the `exec` script again:

```json
{
  "layerName": "Top Bar",
  "projectName": "Sky Travel"
}
```

All looking good. Time to add this extension to Zeplin!

### Adding a local extension

To debug and work with your extensions, you can add them to your Zeplin projects and shared styleguides locally from their Extensions window.

On Mac, Windows or Web apps, holding down the <kbd>⌥ Option</kbd> or <kbd>Alt</kbd> key will enable the “Add Local Extension” option on the title bar:

![Add local extension](img/addLocalExtension.png)

zem provides a script to serve the extension locally:

```sh
npm start
```

After running the script, enter `http://localhost:7070/manifest.json` as the URL and click Add.

That's pretty much it. Go ahead, click a layer!

![Code snippet](img/codeSnippet.png)

#### Logging
You can use `console.log` within your extensions to print logs. These logs should be visible either in the browser console if you're debugging using the Web app, or the Console app if you're using the macOS app.

Tip for macOS app users: In order to filter logs in the Console app, type in `process:Zeplin`, hit ENTER key, and then type `message:[Log] {PLUGIN_DISPLAY_NAME}` (e.g. `message:[Log] React Native` for the React Native Extension), and hit ENTER key again.

### Available functions

Displaying code snippets in the project's local styleguide or shared styleguides is pretty straightforward as well. Similar to the `layer` function, you can implement `colors` and `textStyles` functions.

You can also implement `screen` or `component` functions to display snippets from screens and components.

All the functions that an extension can define are listed in detail in the [`Extension`](model/extension.md) documentation.

### What's next?

So far, we've only accessed the name of the selected layer and the project or the styleguide but extensions can access pretty much all the data you see in Zeplin. In fact, all the built-in code snippets in Zeplin are implemented as extensions, using the same infrastructure.

Make sure to explore the documentation for [package.json](package.md), [extension functions](model/extension.md) and [models](model) to see what's in store! 📚

If you have any questions, feel free to ping us at [extensions@zeplin.io](mailto:extensions@zeplin.io).
