---
layout: post
title:  "Creating a self learning art web app using the Harvard Art API and ES6"
date:   2019-12-15 15:12:58 +0100
categories: jekyll update
---
Today I will show you how to create a web application that teaches itself how to recommend the best art from the Harvard Art Museum to its users. We will use Adobe XD, HTML/CSS, vanilla Javascript and webpack. With an API connection we will retrieve the art.

What you’ll learn in this tutorial

* High fidelity prototyping with Adobe XD
* Responsive HTML5/CSS3 layout
* CSS BEM naming
* CSS Flexbox
* CSS Grid
* CSS button animation
* Webpack config
* Vanilla JS ES6
* Setting up a backend server with Express 
* Setting up a private API to connect backend with frontend
* Setting up a 3rd party API with Harvard art API
* Deploying front- and backend to Heroku 

## 1. Creating the design

For the design, I like to keep things simple and clean. If you are new to design, try to check out dribbble.com and search for ‘art’ or gallery and get inspiration from here. I’m using Adobe XD, which you can download for free from https://www.adobe.com/products/xd.html

If you prefer an online solution, you can use https://www.figma.com/ - which is also free and works similar.

For the app we basically only need two pages: 1) new user page where he/she selects here art preferences, and 2) an overview page with all art recommendations. It’s important to do some reaearch beforehand


### 1.1 Creating the mockup

To combine the 2 main functions from the app, we can place them on one page. So we will have the controls/settings on the left panel, and in the center we will see our art. Make sure you do not use any special fonts/shadows/colors in this stage. Try to make the functionalities clear and have a good balance of elements.

![My helpful screenshot](/assets/1.png)

### 1.2 High fidelity mockup

Here’s comes the special part. The details will make or break your design, so it is not uncommon most time will be spend on the details.

Colors
Fonts
Shadows
Icons

![My helpful screenshot](/assets/2.png)


## 2. Setting up the project

We will create this project using Visual Studio code as a text editor, you can use any you like but I prefer Visual Code because it gives you great feedback and has many extension possibilities.

To test the project we need to use a test web server, we will use Node.js for this. Make sure you have node installed on your computer, you can download it for free from https://nodejs.org/en/download/

Same goes for Visual Studio Code, which you can download for free from https://code.visualstudio.com/download

Then we’ll also use Git and Github, which both are free.  You can download GIT from https://git-scm.com/downloads

Then create an account if you don’t have one already on github.com and create a new repository, this is basically a folder where all you project data will be stored online and locally on your pc. We will call the repository ‘smartart’. Then go to your Github folder on your pc on also create a folder here called ‘smartart’.

We will use the command prompt for managing our git project. Open up the command prompt and browse to your Github folder, in my case C:\Users\’username’\Github. Then go to the smartart folder by using cd smartart (Windows users).

We will initialize this repository by using git init in the command line and then npm init
Then we’ll install the webpack package on our node server by using the following commands
npm install --save-dev webpack webpack-cli webpack-dev-server html-webpack-plugin 

Later when we have added our first files we will upload them to the remote (online) github repository.
Now open up visual code and open your just created smartart folder by using the shortcut (ctr+k) + (ctrl+o).
Open up the package.json file to check if your packages are installed correctly:

{% highlight js %}
"devDependencies": {
    "html-webpack-plugin": "^3.2.0",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10",
    "webpack-dev-server": "^3.9.0"
}
{% endhighlight %} 

Then remove the line in the scripts section and add the following:

{% highlight js %}
"dev": "webpack --mode development",
"build": "webpack --mode production",
"start": "webpack-dev-server --mode development --open"
{% endhighlight %} 

Then create a file called webpack.config.js and add the following: 

{% highlight js %}
const path = require('path');

module.exports = {
    entry: './src/js/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'js/bundle.js'
    },
    devServer: {
        contentBase: './dist'
    },
};
plugins: [
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: './src/index.html'
        })
],
{% endhighlight %} 

Then add the following folders and files

* dist/js/
* src/js/index.js
* src/index.html
* src/css/main.scss

In index.html type doc and press enter to load a standard HTML file.

Then before the ending body tag add <script src="./js/bundle.js"></script>

Let’s add some text on this page, for example <h1>Hello world</h1>

Now open up your src/js/index.js file and add the following

{% highlight js %}
const h1 = document.querySelector('h1');
h1.style.color = 'red';
{% endhighlight %} 

Now use the command ctrl + ~  to open up the terminal in visual studio code.

Type in npm start to open up your new project! Your text should turn red if everything went okay.

We will use sass in our project so we need to add a package in our webpack project that converts scss to css.

Run the command  npm install style-loader css-loader --save

Then in index.js delete everything and add: import '../css/main.scss';

Then fill in the following in main.scss to test if it is working:

{% highlight scss %}
$color1: red;

h1 {
    color: $color1;
}
{% endhighlight %} 

Run npm start again and your h1 should be red!



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
