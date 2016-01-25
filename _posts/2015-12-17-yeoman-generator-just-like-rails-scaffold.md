---
layout:     post
title:      "How to create a simple file and folder generator like the Rails scaffold on Yeoman based on Atomic Design for React"
excerpt: "Make your React project with Yeoman feels like Rails"
date:       2015-12-17 12:00:00
last_modified_at:  2015-12-17 12:00:00
author:     "cassioscabral"
header-img: "img/blog/header/post-bg-01.jpg"
thumbnail: /img/blog/thumbs/thumb01.png
tags: [rails, react, yeoman, generator, scaffold, rateissues, component, building react components, automation, build yeoman generator, factory design pattern on yeoman, cassioscabral, cassio, cabral, cassio s cabral, cassio soares cabral]
categories: rateissues
comments: true
share: true
image:
  feature: railsYeomanFeatureImage.png
  topPosition: 0px
bgContrast: dark
bgGradientOpacity: darker
syntaxHighlighter: yes
---

## How to create a simple file and folder generator like the Rails scaffold on Yeoman


### Why?

I like the workflow from Rails where I can use something like `rails generate controller MyController action1 action2` and get all the files that I need in seconds automatically.

Building a project following a folder structure convention it might be cumbersome, slow and error prone when doing manually. So... automation will help you avoid that.

My generator, that I explained in this post, is called atomic-reactor and you can see it on [Github](https://github.com/cassioscabral/generator-atomic-reactor)

### Is there any other Yeoman generators for that ?

Probably, but since I never built a Yeoman generator before, I chose to write my own for fun

### My motivations

I'm building an open-source application, called RateIssues, and decided to use a convention for my app folders. It has more details on the [github page](https://github.com/cassioscabral/rateissuesfront)(remember to :star: the project if you enjoy :heart:, it means a lot) but in short is:

  - React + Relay + GraphQL + Rails app(as backend)
  - My front is separated from my backend so I can can focus in each part separately
  - I want to build my React components following the [Atomic desing principle](http://bradfrost.com/blog/post/atomic-web-design/) but with a few tweaks and changes
  - So I opt for a folder and file structure that I believe it will work for the project

To make it easy following my convention for folder and files I went for creating my own generator.


### Folder structure desired on RateIssues for React components


Following the Atomic design structure it will have only 3 folders and one main file for the App.

  - atoms/
  - molecules/
  - organisms/
  - App.js

*Since I'm building a SPA, I chose that App.js will be my main component. Ignoring the templates and page parts from the original Atomic design philosophy*

Inside each folder it should have one folder per component

- my_component/ (in **snake_case**)

Inside the component folder it will have all the files related with that component

  - stylesheets
    - my_component.scss (*as the main stylesheet for this component*)
  - tests
  - the component itself (main JSX file)
  - anything related with that component


### Building the generator

#### How do I start ?

Go to yeoman [generators/generators Github page](https://github.com/yeoman/generator-generator) for generating your generator using a generator. (You will understand the "Yo Dawg" meme on their page)

It will setup an initial generator for you

Name **must** start with 'generator-' so it can be accessible/found on the main Yeoman search page

### building the subgenerators

 - atom
 - molecule
 - organism

#### Why?
 I had two options while building this generator

 Create one App generator that would receive the `type` and `component` as argument and for every run you should run the command like this `yo atomic-reactor atom component`

 The other option was building sub-generators that will result in commands like this `yo atomic-reactor:atom component`

 I chose the latter because it makes easy to deal with each type(atom, molecule or organism) independently. Trying to follow the [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle).

#### How ?

To get that command you must generate a sub-generator inside your generator with `yo generator:subgenerator <name>`

With that I was able to build each sub-generator. But I noticed the repeated pattern on them. Basically they share the same functionality but changing only the path where I will generate my folders.

With that in mind I went looking for a way to reuse the code and I went with the [factory design patter](https://en.wikipedia.org/wiki/Factory_method_pattern) where I would get the Yeoman required object to do the process when the command is called(like writing, prompting, etc). So I build a `base.js` file and export my object using `exports` from node, you can see how to use [on this SitePoint post](http://www.sitepoint.com/understanding-module-exports-exports-node-js/).

I required the file and all that I needed to do was call `baseGeneratorFactory('atom')` on `module.exports = yeoman.generators.Base.extend(baseGeneratorFactory('atom'));` you can see more on [Github](https://github.com/cassioscabral/generator-atomic-reactor)


### Copying templates

  Before anything you must look how to use the filesystem on [Yeoman docs](http://yeoman.io/authoring/file-system.html)

  In short what I used was:

  ```
  this.fs.copyTpl(
        this.templatePath(type + '.jsx'),
        this.destinationPath(appPath + componentType + '/' + snakeComponent + '/' + this.component + '.jsx'),
        {component: this.component}
      );
  ```

  Since this will run inside my sub-generator, let's say atom here. It will look for the the file `atom.jsx` in the templatePath which will be `generatos/atom/templates` and copy to my `destinationPath`. This object `{component: this.component}` will be passed to my template and will use the value here to make my custom component looks like `var MyComponent = ...` see [here](https://github.com/cassioscabral/generator-atomic-reactor/blob/master/generators/atom/templates/atom.jsx) how it's used.

#### Creating files

To create files I used this command `this.fs.write(stylesheetsPath + '/' + my_component.scss, '');` Which writes this empty string on that file on that path, it will be created if didn't exist before. Since I'm not defining any template here, I chose to do like that. But you could use a template for it.

#### Creating folders

When setting up my app the first run of the generator `yo atomic-reactor` will create the default folders in case they don't exist already.

I used the `makedirp` module to do this.

```
mkdirp.sync('src/app/atoms/');
mkdirp.sync('src/app/molecules/');
mkdirp.sync('src/app/organisms/');
```


#### Creating a command to generate something
With all that I was able to run the commands:

```
yo atomic-reactor:atom my_component
yo atomic-reactor:molecule my_component
yo atomic-reactor:organism my_component
```
Generating:

```
▶ yo atomic-reactor:organism my_component
create src/app/organisms/my_component/MyComponent.jsx
create src/app/organisms/my_component/stylesheets/my_component.scss
create src/app/organisms/my_component/tests/my_component.js
```

#### How to pass values through arguments on the command line

In each sub-generator I used the Yeoman suggestion for [arguments](http://yeoman.io/authoring/user-interactions.html) and did this on the constructor:

```
this.argument('component', { type: String, required: true });
this.component = _.capitalize(_.camelCase(this.component));
```

#### Notes

Yeoman will prompt the user if some conflict is found when creating, replacing, updating a file so the user can decide what to do.

To test your generator on some random application just go to your generator root folder and type `npm link` and then add to your random project `npm link generator-my-generator` to reference the local generator. You can read more about link and how to run packages locally [here](http://stackoverflow.com/questions/20888576/how-to-develop-npm-module-locally)

Yeoman offers several options for interact with the user. Using the prompting option for example.

Feel free to contact me for any doubts. I will be glad to help anyway I can.
