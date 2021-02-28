
In this tutorial i will be showing you how to create an npm library that is composed of react component. This will definetly help you incase you want to reuse code in multiple projects or if you just want to create your own library.

#### Table of contents:
- [Getting Started](#getting-started)
- [Creating the library](#creating-the-library)
- [Initializing the library](#initializing-the-library)
- [Bundling the library](#bundling-the-library)
  - [Re-arranging the package directory](#re-arranging-the-package-directory)
  - [Installing bundlers](#installing-bundlers)
  - [Configuration](#configuration)
  - [Editing `package.json` scripts](#editing-packagejson-scripts)
  - [Build package](#build-package)
- [Editing `package.json`](#editing-packagejson)
  - [Fields](#fields)
    - [`name` and `vesrion`](#name-and-vesrion)
    - [decription](#decription)
    - [keywords](#keywords)
    - [homepage](#homepage)
    - [license](#license)
    - [people fields: author, contributors](#people-fields-author-contributors)
  - [Adding `peerDependecies`](#adding-peerdependecies)
  - [Final package.json](#final-packagejson)
- [Publishing](#publishing)
  - [Creating `.npmignore`](#creating-npmignore)
  - [Finding a name](#finding-a-name)
  - [Testing your package](#testing-your-package)
  - [Adding README.md](#adding-readmemd)
  - [Publishing](#publishing-1)
- [Conclusion](#conclusion)
  

If you are ready, lets get started.

## Getting Started
First we shall initialize a react project. So got to your terminal and enter the directory that you want to create your project and create a new react app with the `create-react-app` CLI.

```bash
npx create-react-app react-npm-library
## then enter the new directory
cd react-npm-library
## then start the dev server
yarn start
```
I recommend using `npx` sinc it installs the latest versions of `react-scripts` and does not install any thing globally.

Then enter the `src` directory and create a new directory where you will place your component library

```bash
cd src
mkdir react-library ## you can name it  any name
```

## Creating the library

if you have done the above steps now you are ready to create you libray. So now lets create a simple library that creates good buttons.

Inside the `react-library` directory we shall create a file for the component.
```bash
torch button.jsx
torch index.css
```
Then place this code in `button.jsx`

```jsx
import React, {useState, useEffect} from 'react';
import './button.css'

const AwesomeButton = (props) => {
    const [color, setColor] = useState(null)
    useEffect(() => {
        if (props.variant) {
            if (props.variant === 'primary') {
                setColor('#0077ff')
            } else if (props.variant === 'secondary') {
                setColor('#ff0062')
            } else if (props.variant === 'success') {
                setColor('#0f8000')
            } else {
                setColor('#949393')
            }
        }
    })
    const {children} = props
    
    return (
        <button 
            className='buttonComponent'
            style={{
                backgroundColor: color
            }}
        >
            {children.toUpperCase()}
        </button>
    )
}

export default AwesomeButton;
```

in index.css
```css
.buttonComponent {
    border-radius: 3px;
	padding: 0.3rem 0.5rem;
    transition: all .3s ease-out;
	box-shadow: #272727b0 1px 1px 1px, #272727b0 -1px -1px 1px;
}
.buttonComponent:hover {
    box-shadow: #272727b0 1px 1px 3px, #272727b0 -1px -1px 3px;
}
.buttonComponent:active {
    opacity: .8;
}
```

Now you can import it from `App.js` and test it. If it works well then lets move on.

## Initializing the library

Now if it works so you have to enter the `react-library` directory and create get it ready for publishing.
```bash
cd react-library
```

After then initialize an npm package
```bash
npm init -y
```
This will create a `package.json` file in the root directory. It should look like this

```json
{
  "name": "react-library",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
Now you are ready to move on to the next section

## Bundling the library

Now lets get ready to bundle the library. We shall do this in a few steps

### Re-arranging the package directory

So now lets arrange the `react-library` directory so it can be favourable for bundling.

Move to your terminal and type these commands inside the `react-library` directory
```bash
mkdir src
move button.jsx src
move index.css src
cd src
torch index.js
```
The above commands will move the `button.jsx` and `index.css` files to a new `src` directory and also create a new file called `index.js`
By now your project structure looks something like this.
```bash
|
 - src
   | - index.js
   | - index.css
   | - button.jsx
 - package.json
```

Inside the `index.js` file add the following code

```js
import AwesomeButton from  './button.js'

const returnLibrary = () => {
    return {
        AwesomeButton: AwesomeButton
        // you can add here other components that you want to export
    }
}
export default returnLibrary()
```

### Installing bundlers

Now lets install the bundlers. I recommend [rollup](https://rollupjs.org) since I thimk it is easy to use when bundling libraries compared to webpack but if you want to use another bundler then you can use it.
So in the root of the `react-library` directory install rollup.

> __Note__: You have to install them as `devDependencies` by adding the `--save-dev` flag. This is because they are used by you when in development mode else if you don't this will cause make your users to install thenm if you add them to the `dependency` list
```bash
npm install rollup --save-dev
```
Rollup will be used to compile our code. But since we definitely might want to compile into es5 syntax so we shall have to install [__babel__](https://babeljs.io/) and a plugin to use with rollup. You should not that `jsx` syntax is special and is not valid javascript so you should also install support for it.
So type the following commad int in the command line to install all __required__ packages.

```bash
npm install @babel/cli @babel/core @babel/preset-env @babel/preset-react @rollup/plugin-babel --save-dev
```

Since we are also bundling css then we shall have to install a styles bundler for rollup we shall use `rollup-plugin-styles`

```bash
npm install rollup-plugin-styles autoprefixer --save-dev
```
___
__Optional__

We can also add babel runtime helpers. this is important if you are bundling a library with babel. So type this in the command line
```bash
npm install @babel/runtime
npm install @babel/plugin-transform-runtime --save-dev
```

If you want sourcemaps then intall this plugin to.

```bash
npm install rollup-plugin-sourcemaps --save-dev
```
___

### Configuration

Now lets configure the rullop and babel for compiling our code.
In the root directory create these to files.
- `rollup.config.js`
- `.babelrc`


Inside `rollup.config.js` add the following code.
```js
import styles from "rollup-plugin-styles";
const autoprefixer = require('autoprefixer');
import { terser } from 'rollup-plugin-terser'
import babel from '@rollup/plugin-babel';

// the entry point for the library
const input = 'src/index.js'

// 
var MODE = [
  {
    fomart: 'cjs'
  },
  {
    fomart: 'esm'
  },
  {
    fomart: 'umd'
  }
]




var config = []


MODE.map((m) => {
    var conf = {
        input: input,
        output: {
            // then name of your package
            name: "react-awesome-buttons",
            file: `dist/index.${m.fomart}.js`,
            format: m.fomart,
            exports: "auto"
        },
        // this externelizes react to prevent rollup from compiling it
        external: ["react", /@babel\/runtime/],
        plugins: [
            // these are babel comfigurations
            babel({
                exclude: 'node_modules/**',
                plugins: ['@babel/transform-runtime'],
                babelHelpers: 'runtime'
            }),
            // this adds sourcemaps
            sourcemaps(),
            // this adds support for styles
            styles({
                postcss: {
                    plugins: [
                        autoprefixer()
                    ]
                }
            })
        ]
    }
    config.push(conf)
})

export default [
  ...config,
]
```
Also add this to `.babelrc`
```json
{
    "presets": [
        "@babel/preset-react",
        "@babel/preset-env"
    ]
}
```
### Editing `package.json` scripts

Now got to `package.json` and edit the scripts section and change it to this.
```json
// package.json
...
"scripts": {
    "build": "rollup -c"
}
...

```

### Build package

Now that everything is set run

```bash
npm run build
```
This will compile your package into the `dist` directory.


## Editing `package.json`

Now that our library has been built lets edit `package.json` to make our library ready for publishing.

If you have followed from the beginning i think your `packages.json` looks something like this.

```json
{
    "name": "react-library",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "directories": {
        "test": "test"
    },
    "scripts": {
        "build": "rollup -c"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "dependencies": {
		"@babel/runtime": "^7.12.5"
	},
    "devDependencies": {
		"@babel/cli": "^7.12.10",
		"@babel/core": "^7.12.10",
		"@babel/plugin-transform-runtime": "^7.12.10",
		"@babel/preset-env": "^7.12.11",
		"@babel/preset-react": "^7.12.10",
		"@rollup/plugin-babel": "^5.2.2",
		"rollup-plugin-sourcemaps": "^0.6.3",
		"rollup-plugin-styles": "^3.12.2",
	}
}
```

I will explain what a few important feilds represent and you can find out more on at the [npm documentation](https://docs.npmjs.com/cli/v6/configuring-npm/package-json "npm documentation").

### Fields
#### `name` and `vesrion`
If you plan to publish your package, the most important things in your package.json are the name and version fields as they will be required. The name and version together form an identifier that is assumed to be completely unique. Changes to the package should come along with changes to the version. If you don't plan to publish your package, the name and version fields are optional.
A name can be optionally prefixed by a scope, e.g. @myorg/mypackage.

#### decription
Put a description in it. It's a string. This helps people discover your package, as it's listed in npm search.
#### keywords
Put keywords in it. It's an array of strings. This helps people discover your package as it's listed in npm search.

#### homepage
The url to the project homepage.

#### license
You should specify a license for your package so that people know how they are permitted to use it, and any restrictions you're placing on it.

If you're using a common license such as BSD-2-Clause or MIT, add a current SPDX license identifier for the license you're using, like this:
```json
{"license":"BSD-3-Clause"}
```
#### people fields: author, contributors
The "author" is one person. "contributors" is an array of people. A "person" is an object with a "name" field and optionally "url" and "email", like this:

```json
{
    "name":"Barney Rubble",
    "email":"b@rubble.com",
    "url":"http://barnyrubble.tumblr.com/"
}
```
### Adding `peerDependecies`
Since we dont want to have react installed twice in the users projects but also need the user to have it we shall add it as a `peerDependecies` so add this to you package.json file
```json
"peerDependencies": {
		"react": "^17.0.1",
		"react-dom": "^17.0.1"
	}
```
### Final package.json

Now edit it to look like this

```json
{
	"name": "react-library",
	"version": "1.0.0",
	"description": "your description",
	"main": "dist/index.cjs.js",
	"scripts": {
		"build": "rollup -c"
	},
	"peerDependencies": {
		"react": "^17.0.1",
		"react-dom": "^17.0.1"
	},
	"dependencies": {
		"@babel/runtime": "^7.12.5"
	},
	"keywords": [
		"react",
		"keywords"
	],
	"author": "Your name",
	"license": "MIT",
	"devDependencies": {
		"@babel/cli": "^7.12.10",
		"@babel/core": "^7.12.10",
		"@babel/plugin-transform-runtime": "^7.12.10",
		"@babel/preset-env": "^7.12.11",
		"@babel/preset-react": "^7.12.10",
		"@rollup/plugin-babel": "^5.2.2",
		"rollup-plugin-sourcemaps": "^0.6.3",
		"rollup-plugin-styles": "^3.12.2",
	}
}

```

## Publishing

Now you are ready to publish.
First create an npm account if you don't have one.

### Creating `.npmignore`

I hope by now your projects structure looks like this:

```bash
|
| - dist
    | - index.esm.js
    | - index.cjs.js
    | - index.umd.js
| - src
    | - index.js
    | - index.css
    | - button.jsx
| - .babelrc
| - package.json
| - rollup.config.js
```
But since we dont want to publish our source code to npm we only want to publish the compiled code then we will create a `.npmignore` file that will prevent files we dont want to publish from being published

add this to `.npmignore` file.
```ignore
## the src folder
src
.babelrc
rollup.config.js
## node modules folder
node_modules
## incase you have a git repositiory initiated
.git
.gitignore
CVS
.svn
.hg
.lock-wscript
.wafpickle-N
.DS_Store
npm-debug.log
.npmrc

config.gypi
package-lock.json
```
### Finding a name

Sometimes you might try to publish a package and find that the name is either already taken or the name is almost identical to onother package so its better to first search and see if the package name is already taken. So type the following command in the command line.
```bash
npm search [package name]
```

if you find that nobody is using it them you can usethe name.

### Testing your package

To test your package your have to go to another projects on you comuter and type
```bash
npm link /path/to/your/package
```


### Adding README.md
You should also add a `Readme.md` file that will be displayed on npm having a description of your package. You might be familiar with it if you have ever created a repository on GitHub

### Publishing

If all works well then you can publish it by typing 
```bash
npm publish
```
## Conclusion
I hope this article has been helpful to you if you have any question just leave it in the comments section and all the source code can be found on [Github](https://github.com/jim-junior/react-npm-library-template "GitHub repository")