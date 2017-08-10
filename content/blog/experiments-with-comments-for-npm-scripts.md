+++
date = "2017-06-19T12:53:11+02:00"
draft = false
title = "Experiments with comments for npm scripts"

+++

The amount of npm scripts inside package.json can grow immensely for larger projects, it can become hard to track what each script does, especially if there are multiple people working on the same project and adding the mentioned scripts. It would be of huge help if we could comment what each script does so it's easy to check what it does and how it's used, but due to package.json being a JSON file format, it doesn't support any comments. Despite non-native support for comments there are some tricks we can use to work around that limitation.

## Checking available scripts

To check what scripts we can run in terminal we can type `npm run` (if you're using **yarn** you can do `yarn run`), this will return a list of npm scripts available to us. Let's assume we have a **package.json** like this in our project directory:
```json
{
    "name": "npm-script-tips",
    "version": "0.0.1",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "build": "echo \"Building static assets\""
    }
}
```

To check the available scripts we can run `npm run` in our shell, and we get the list of all available scripts:
```sh
Lifecycle scripts included in npm-script-tips:
  test
    echo "Error: no test specified" && exit 1

available via `npm run-script`:
  build
    echo "Building static assets"
```

## Solution 1: Attempting comments with // prefixes

To add a comment you can add fake a npm script entry like the following:
```json
{
    "name": "npm-script-tips",
    "version": "0.0.2",
    "scripts": {
        "//test": "This will run the app's tests",
        "test": "echo \"Error: no test specified\" && exit 1",
        "//build": "This will build the static assets",
        "build": "echo \"Building static assets\""
    }
}
```

The output after `npm run` is now a bit better:
```sh
Lifecycle scripts included in npm-script-tips:
  test
    echo "Error: no test specified" && exit 1

available via `npm run-script`:
  //test
    This will run the app's tests
  //build
    This will build the static assets
  build
    echo "Building static assets"
```

**PROS**

- should work on all platforms
- the comment is next to the file inside package.json
- the comment is visible on `npm run`

**CONS**

- on `npm run` the comments and scripts aren't grouped next to each other

## Solution 2: A hacky attempt using \#

Watching [egghead.io's](https://egghead.io) tutorials on npm scripts a great solution has been proposed by [Elijah Manor](http://elijahmanor.com).

Most of egghead tutorials are for subscribers only but they definitely pay out to have and their videos usually take only a few minutes and if you can afford them you'll find great value in them.

Elijah suggests using the following trick:
```json
{
    "name": "npm-script-tips",
    "version": "0.0.3",
    "scripts": {
        "test": "# This will run the tests \n\t echo \"Error: no test specified\" && exit 1",
        "build": "# This will build the static assets \n\t echo \"Building static assets\""
    }
}
```
What I believe this translates to and why it works is that when we run `npm run` it feeds the following command into the shell:
\# This will run the tests
    echo "Building static assets"
The first line is treated like a comment and won't be evaluated, the second line is properly executed.

The output looks great with this trick:
```sh
Lifecycle scripts included in npm-script-tips:
  test
    # This will run the tests
     echo "Error: no test specified" && exit 1

available via `npm run-script`:
  build
    # This will build the static assets
     echo "Building static assets"
```

**PROS**

- looks great on `npm run`, the code is properly displayed and nicely formatted
- works with both **npm** and **yarn**

**CONS**

- it doesn't work on Windows, it's only supported on *nix type systems
- the comments can hide the script command

Because **npm** supports only JSON for it's package file format we're stuck with these limitations which is pretty unfortunate.

## Solution 3: Using extern tooling for solutions
We can use scripts like the [npm-ls-scripts](https://github.com/jaketrent/npm-ls-scripts). If we install the tool globally `npm install npm-ls-scripts -g`, we can use it via `ls-scripts`. Now to enable comments we have to add a config object inside package.json which includes a scripts object with all the scripts we want to comment, it should look like this:
```json
{
    "name": "npm-script-tips",
    "version": "0.0.1",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "build": "echo \"Building static assets\""
    },
    "config": {
        "scripts": {
            "build": "Will build static assets"
        }
    }
}
```

It will output:
```sh
NPM - ls scripts
---
build - Will build static assets
test  - echo "Error: no test specified" && exit 1
---
```
Which looks great.

**PROS**

- Comments look great with this tool
- It's easy to use

**CONS**

- you have to deal with another tool which may not be acceptable for everyone
- although in the same file the comments and scripts are in separate places

Additional info can be found on the [author's page](https://jaketrent.com/post/list-npm-scripts/) and on [github](https://github.com/jaketrent/npm-ls-scripts).

## Solution 4: Use the README.md

I believe the README.md file should be present on every project to describe what it does and how to set it up. Depending on your team's dedication to maintain project hygiene you can maintain a list and describe all the scripts inside the README file.

**PROS**

- all can be nicely formatted there
- you have the power of MARKDOWN to add links, usage examples and longer explanations for complex scripts

**CONS**

- the comments and the scripts are now in different places and may run out of sync


## Conclusion

There's no perfect solution here like with most things programming related, it's up to you to weight the pros and cons and determine what works best for you and your team.