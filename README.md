# React without CRA: Removing the Training Wheels
I've been using React for about 7 years now.  I've built several production applications with it and I enjoy working with React.
I've always leaned on Create React App to set up the application for me.  It's a great tool and saves time.  I think though that
at some point you really need to move past the "Done For You" scripts.  This is my exercise in doing that.  Of course you can also
skip all the typing and just clone this repo, run `npm i`, then delete the `.git` folder and use this as your starter code.  That's 
my plan as well.

## Why would you want to do this?
I personally just want to learn to create the simplest possible React app without using any automation.  There are some advantages
to skipping CRA:
* CRA doesn't allow you to alter the `webpack` scripts without breaking the CRA functionality.  webpack is a powerful tool and there 
is a lot you can do to tweak the way your app is built by editing the build scripts.
* No bloat?  Maybe?  Theoretically?  Its NodeJS.  Its all bloated.  Maybe this is less bloated?

## How is it done?

I wanted to come up with a way to memorize this stuff and be able to do this from memory using nothing more than a terminal with a 
terminal editor like NeoVim.  Why? Its a great way to impress the ladies at parties and random bars with good wifi.  I don't know.  I'm 
happily married, but I think it would work well if I weren't.  Maybe it would make for a baller move during a job interview?  This seems
like the kind of useful thing nobody would ever ask you on an interview, instead preferring to try to make you read the interviewer's mind
to glean out some silly memory hack they read about 10 minutes before the intervew?  Whatever, back to memorizing the whole process...

Honestly, I can't do that.  There is a lot of configuration that is just too long to memorize.  Maybe I can trim this down later, but
even still, I'm not sure this is a party trick you can pull off from memory unless you are really dedicated, and I can't guarantee it 
will over pay off the way my Database Touch And Go exercise does.

This isn't going to be a deep tutorial.  I'm going to assume you know your way around your OS's terminal.  I am using Windows 11 at the 
moment, so mine is PowerShell.  That said, I'm really a Linux guy at heart, so all the commands are going to be Linux commands which 
kind of work in PowerShell -- well enough anway.  I'm also going to assume you know how to install NodeJS.  I happen to be using version 20 LTS.

Let's get started.  Begin by making a folder:

```bash
mkdir nocra
cd nocra
```
Initialize your node project.

```bash
npm init -y
```

The `-y` switch just answers 'yes' to all the usual questions.  Of course you can fill in the entries if you'd prefer, but I usually do this later.
So far, these are steps you could memorize if you wanted to.  The next few are installing a bunch of packages you'll need that would normally be 
handled by CRA.  First, I'll do the dev dependencies.  These primarily revolve around the `babel` transpiler and the `webpack` build tool.  JavaScript
is normally considered an interpreted language and therefore shouldn't need a build step at all.  In reality, JavaScript is compiled, but it is compiled
just-in-time as its interpreted for the first time.  That said, we still shouldn't need a build step like we would in a C program where the C code is
converted into binary.  With React, the build step doesn't compile, but rather transpiles - the code gets converted from a secondary language, namely JSX
into normal JavaScript.  A lot of what makes React great is our ability to use JSX as a templating language.

The tool that performs the transpile from JSX to JavaScript is called `babel`, taking its name from the Tower of Babel in Genesis chapter 11.  Babel is the 
transpiler, but `webpack` is the build technology used to control the transpile process.  The following collection of packages install all the dependencies 
needed for this.  Since the transpile is performed and results in a fully built artifact, these packages are not needed in the production product.  As such,
they are dev dependencies.

```bash
npm install --save-dev @babel/core babel-loader @babel/cli @babel/preset-env @babel/preset-react webpack webpack-cli webpack-dev-server html-webpack-plugin
```

Next, let's install React itself:

```bash
npm install react react-dom
```

Everything is now installed that you will need!  Next we need to create some folders and some files for the content of our application.

### Files and folders

Let's create our project folders.  At a minimum we will need two.

```bash
mkdir src
mkdir public
cd public
```
We created our two required folders and we've cd'ed into public.  We need an `index.html` file to hold everything.

```bash
nvim index.html
```

If you're not using neovim, you can just substitute your normal file creation process here.  Inside the index file, add this boilerplate HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>React without CRA</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```
Save this file and exit.  Next cd into the src folder we created earlier, and create `App.js`.

```bash
cd ../src
nvim App.js
```
Add this code:
```JavaScript
import React from "react";

const App = () => {
  return (
    <h1> Hello World! I am using React without CRA! </h1>
  )
};

export default App;
```
Save the file.  Don't get too excited, there's still a lot left to do.  Exit the editor if you need to.  CD back up to the root folder.
Create a file called `index.js`.

```bash
cd ../
nvim index.js
```

Type in this code:

```JavaScript
import React from 'react';
import {createRoot} from 'react-dom/client';
import App from './src/App.js';

const container = document.getElementById('root');
const root = createRoot(container);

root.render(<App />);
```
All this is left is to configure the build!

### Configuring the build
In the project root, create a file called .babelrc and add this to it:

```json
{
    "presets": ["@babel/preset-env","@babel/preset-react"]
}
```

Save that.  Make another file called `webpack.config.js`.  I'll have to confess that I pulled this out of an app I already have. 
I don't remember where it came from.  This one's a bit longer:

```JavaScript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: './index.js',
  mode: 'development',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js',
  },
  target: 'web',
  devServer: {
    port: '5000',
    static: {
      directory: path.join(__dirname, 'public')
},
    open: true,
    hot: true,
    liveReload: true,
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/, 
        exclude: /node_modules/, 
        use: 'babel-loader', 
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, 'public', 'index.html')
    })
  ]
};
```
Save it.  One last thing:  you need to alter your `package.json` file.  Open it in your editor 
and find the `scripts` section.  Alter it to look like this:

```json
  "scripts": {
    "start": "webpack-dev-server .",
    "build": "webpack .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

Save it.  You're done!

### Run it

Exit your editor if necessary.

```bash
npm start
```

Your browser should launch with your app as soon as the transpile step completes!

And there you have it.  You are not longer a slave to CRA!  You can alter your webpack
process to your heart's content!

