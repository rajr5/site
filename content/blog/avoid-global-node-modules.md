+++
date = "2017-07-08"
draft = false
title = "Avoid global node modules"

+++

I suggest avoiding project related global dependencies and installing everything you need for a project as main or dev dependencies. Sometimes you just want to hack something up and you do a `npm i -g ...` and start coding without thinking about the consequences. For example my team used a globally installed `sequelize-cli` which later cause problems due to different versions being installed accross the team. It can cause problems for your personal projects too, if you have one project using `sequelize` 3.x and one using `sequelize` 4.x and you're in a bad spot, now you have to think about managing the issue which adds unneeded and avoidable overhead.

In my opinion the main reasons to avoid global dependencies are:

- non-explicit project dependencies
- harder developer colaboration
- issues if multiple projects use different versions of globally installed packages

## devDependencies, a smarter alternative

Instead use locally installed node modules whenever you can, if they are tooling or build related packages install them as a dev dependencies via the -D flag, e.g. `npm i -D jest`. This'll add `jest` as a development dependency (which means you don't really need it installed to run the project, only to run the tests), now your *package.json* should have `jest` installed in the devDependencies section:

```json
{
    "name": "test-project",
    "dependencies": {
        ...
    },
    "devDependencies": {
        "jest": "^20.0.4",
        ...
    },
    "scripts": {
        ...
    },
}
```
The scripts will now be installed when you run `npm i`, no need to ask around what the projects global dependencies are, e.g. _"How do I run the project migrations"_, if the package for migrations is installed locally you'll see the related package listed under dev dependencies.

## Using locally installed packages

If you are using a globally installed package, you can access it anywhere via it's name directly:

`sequelize migration:create --name create_user_table`

If the *sequelize-cli* has been added as a dev dependency, it's a bit more tedious, now you'll have to specify a path either directly to the node module, or to the symlink inside the `.bin` file, which is basically the same thing:

```bash
# calling the module directly
./node_modules/sequelize-cli/bin/sequelize migration:create --name create_user_table

# calling the module symlink
./node_modules/.bin/sequelize migration:create --name create_user_table
```

### Localy installed packages

All the installed packages you might need to interact with are added to the **node_modules/.bin** directory, if you run:
`ls -al node_modules/.bin` inside a node project you'll see a number of the installed packages, all the items you see are symlinks which are linked to scripts in their corresponding node_module folder. You can see that when you list the contents of `<project root>/node_modules/.bin`:

```bash
→ ls -al node_modules/.bin
total 456
drwxr-xr-x   59 dbadrov  staff   2006 Jul 19 12:50 .
drwxr-xr-x  839 dbadrov  staff  28526 Jul 19 12:50 ..
lrwxr-xr-x    1 dbadrov  staff     19 Jul 18 10:00 _mocha -> ../mocha/bin/_mocha
lrwxr-xr-x    1 dbadrov  staff     19 Jul 18 10:00 gulp -> ../gulp/bin/gulp.js
lrwxr-xr-x    1 dbadrov  staff     28 Jul 18 10:00 handlebars -> ../handlebars/bin/handlebars
lrwxr-xr-x    1 dbadrov  staff     30 Jul 18 10:00 sequelize -> ../sequelize-cli/bin/sequelize
...
```


## The new npx tool

A new tool called `npx` is introduced in `npm@5.2.0`. It's called the npm package runner and it can call locally installed scripts without the need to specify a path, it'll check if the package is installed and use it if it's there, otherwise it'll install it, execute the command and remove the package. Using `npx` you can now write:

`npx sequelize migration:create --name create_user_table`

I didn't get to play much with it, but it seems like a super useful tool. For more info check the [introductory article](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b).

## npm scripts

To make the available options of a project more explicit and intuitive you can add all of the used commands as npm scripts, which are there to avoid repetetive tasks. E.g. if you use **sequelize** in your project and **sequelize-cli** for migrations.


Npm scripts can directly call the installed tools without specifying the path, so inside package.json you can add a script for creating migrations like:
```json
{
    "name": "npm-local-scripts",
    "version": "0.0.1",
    "scripts": {
        "create-migration": "sequelize migration:create"
    }
}
```

To call it and properly pass the parameters you need to add the `--` flag like this:
`npm run create-migration -- --name create_user_table`

The downside to using this is that you have to rembember the `--` param, but now you don't need to remember the exact syntax for the script commands, e.g. _"was it sequelize migration:create or sequelize create:migration"_. Another advantage of having the script inside package.json is that you can list and tab-complete all the possible commands with `npm run ` and then pressing tab:

```bash
→ npm run
create-fixtures       deploy                integration           sequelize-seed
create-migration      elastic               integration-test      start
create-test-database  import-test-database  sequelize             test
```

## Enable autocompletion of npm script commands

To ease up the usage of npm scripts you can pretty easily enable the tab-completion of all npm commands, including scripts.
To do it, open your shell and depending on what you use:
```sh
# if you're using zshell
npm completion >> ~/.zshrc # adds the autocompletion code to your rc file
source ~/.zshrc # enable autocompletion by executing the definitions inside the rc file

# if you're using bash
npm completion >> ~/.bashrc
source ~/.bashrc
```

Now you can enter `npm run` and press tab, and it'll list all your scripts, start typing the name of one of the scripts and it'll try to autocomplete it just like file/directory names in your unix shell.

## Some exceptions

This is just my view of a best practice, and there are always some exceptions, you shouldn't install `create-react-app` and similar tools as dependencies/devDependencies, that doesn't make much sense.
For packages like `nodemon` (backend watcher and live reload) or `live-server` (super simple file server with live reload) I think there's no need to have them installed as dev dependencies and these are safe to be used as global packages, your teammates could have some 
