---
layout: post
title:  "Creating an art recommending web app using the Harvard Art API - part 2: MVC & State"
date:   2019-12-16 15:12:58 +0100
categories: vanillajs
---
## 4. Setting up the events

Next up we will make the navigation working and the buttons selectable.

### 4.1 Buttons

Let’s select all our buttons in index.js:

{% highlight js %}
const buttons = document.querySelectorAll('.box__item');
{% endhighlight %}

Then add an event listener to track clicks for each of the buttons by looping over them and adding a function btnClick() to each button click. Note that the function does not contain the () because it is not directly invoked, only when the click is called.

buttons.forEach(button => button.addEventListener('click', btnClick));

To toggle the active class on each button, we add the following code:

{% highlight js %}
const btnClick = (event) => {
    event.target.classList.toggle("active");
}
{% endhighlight %}

Because the btnClick function is a declared function, it is not hoisted as first in the javascript execution context. This basically means we need to write it before we add our eventlistener, otherwise they can not find the function to execute.

### 4.2 Painting slider

We currently have five example paintings which need to slide whenever we click the arrows. First we wrap our slides in a new div called art__wrapper which we will give the following nested slides, instead of the art section:

{% highlight css %}
.art__wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
}
{% endhighlight %}

Now we can control the painting the user is viewing by moving the wrapper left or right with margins.

Let’s select our arrows and add event listeners to them:

{% highlight js %}
const arrowLeft = document.querySelector('.circle__left');
const arrowRight = document.querySelector('.circle__right');

const slide = (target) => {
    console.log(target);
}

arrowLeft.addEventListener('click', slide);
arrowRight.addEventListener('click', slide);
{% endhighlight %}
 
Now we need to know in our function if the right or the left slide has been pressed. The user can also click the arrow icon which does not contain a left or right indication. We can solve this by grabbed the parentNode of the icon:

{% highlight js %}
const slide = (event) => {
    let direction;
    if (event.target.classList.contains("circle__left") || event.target.parentNode.classList.contains("circle__left")) {
        direction = 'left';
    } else {
        direction = 'right';
    }
    console.log(direction);
}
{% endhighlight %}

Add a querySelector on the art wrapper. Then we need to get the current margin left and then add some to it to move the painting. We can do this by the currentstyle property or the getComputedStyle (if not microsoft). Then we parse this string to a number.

{% highlight js %}
if (event.target.classList.contains("circle__left") || event.target.parentNode.classList.contains("circle__left")) {
    // LEFT
    const style = artWrapper.currentStyle || window.getComputedStyle(artWrapper);
    let currentMargin = parseInt(style.marginLeft.replace('px', ''));
    artWrapper.style.marginLeft = currentMargin + 200;
} else {
    // RIGHT
}
{% endhighlight %}


We do not want our users the be able to scroll forever so we need to limit the amount they can scroll. We can do this by checking the amount of paintings and their total width including margins. First add a query selector for all the paintings. Our total slide functionality now looks like this:

{% highlight js %}
const arrowLeft = document.querySelector('.circle__left');
const arrowRight = document.querySelector('.circle__right');
const artWrapper = document.querySelector('.art__wrapper');
const paintings = document.querySelectorAll('.painting');

const slide = (event) => {
    let direction, currentMargin, maxWidth;

    maxWidth = (paintings.length) * 300;

    const style = artWrapper.currentStyle || window.getComputedStyle(artWrapper);
    currentMargin = parseInt(style.marginLeft.replace('px', ''));

    if (event.target.classList.contains("circle__left") || event.target.parentNode.classList.contains("circle__left")) {
        // LEFT
        let currentMargin = parseInt(style.marginLeft.replace('px', ''));
        if (currentMargin < maxWidth) artWrapper.style.marginLeft = currentMargin + 300;

    } else {
        // RIGHT
        let currentMargin = parseInt(style.marginLeft.replace('px', ''));
        if (currentMargin > (maxWidth * -1)) artWrapper.style.marginLeft = currentMargin - 300;
    }
}

arrowLeft.addEventListener('click', slide);
arrowRight.addEventListener('click', slide);
{% endhighlight %}

And that’s it for the event listeners! In the next section we will change our code to the MVC model and set the state.

## 5. Adding MVC and state


### 5.1 Setting up a MVC model

Although setting up the model, view and controller system is a lot of work for just this small app, it is good to practise and get familiar with MVC. The model manages the data of the application, the view manages what actually get displayed on screen and the controller connect the two. The model never touches the view. The view never touches the model. The controller connects them. Create two news folders within your /js folder called models and views. We do not have a model yet (which stores and manages data) so we will start with the view. Create two new files inside the views folder called elements.js and painting.js. Elements will contain all our query selectors.
Add the following query selectors in elements.js:

{% highlight js %}
export const elements = {
    settings: document.querySelector('.settings'),
    buttons: document.querySelectorAll('.box__item'),
    arrowLeft: document.querySelector('.circle__left'),
    arrowRight: document.querySelector('.circle__right'),
    artWrapper: document.querySelector('.art__wrapper'),
    paintings: document.querySelectorAll('.painting'),
    generate: document.querySelector('.box__generate'),
    classification: document.querySelector('.classification'),
    period: document.querySelector('.period'),
};
{% endhighlight %}

Now we can import these files in index.js by adding the following in the top of the page:

{% highlight js %}
import { elements } from './views/elements';
import * as paintings from './views/paintingView';
{% endhighlight %}

Place the code of the painting slider inside views/paintingView.js file.

So it looks like this:

{% highlight js %}
import { elements } from './elements';

// SLIDE FUNCTIONALITY 

export const slide = (event) => {
    let direction, currentMargin, maxWidth;

    maxWidth = (elements.paintings.length) * 300;

    const style = elements.artWrapper.currentStyle || window.getComputedStyle(elements.artWrapper);
    currentMargin = parseInt(style.marginLeft.replace('px', ''));

    if (event.target.classList.contains("circle__left") || event.target.parentNode.classList.contains("circle__left")) {
        // LEFT
        let currentMargin = parseInt(style.marginLeft.replace('px', ''));
        if (currentMargin < maxWidth) elements.artWrapper.style.marginLeft = currentMargin + 300;

    } else {
        // RIGHT
        let currentMargin = parseInt(style.marginLeft.replace('px', ''));
        if (currentMargin > (maxWidth * -1)) elements.artWrapper.style.marginLeft = currentMargin - 300;
    }
};
{% endhighlight %}

### 5.2 Creating state 

Let’s start working on the settings section. The preferences of the user should be stored and saved somewhere while the user is using the application. We can do this in a new object which we call the state. Let’s add an empty object in index.js called state.

{% highlight js %}
const state = {};
{% endhighlight %}

Add a query selector in elements for our generate button. Then in index.js add:

{% highlight js %}
// SAVE NEW SETTINGS
const controlSettings = () => {

    // Retrieve settings from settingsView
    const newSettings = settingsView.getSettings();

    // Update state with new settings
    state.settings.userSettings = newSettings;

}

elements.generate.addEventListener('click', controlSettings);
{% endhighlight %}

Now create a new file called settingsView.js where we will render the setting items and also retrieve the new settings when the generate button is called:

{% highlight js %}
import { elements } from './elements';

export const renderSettings = (data, type) => {
    const markup = `
        <div data-type="${type}" class="box__item">${data}</div>
    `;
    type === 'classification' ? 
    elements.classification.insertAdjacentHTML('afterend', markup)
    : elements.period.insertAdjacentHTML('afterend', markup)
}

export const getSettings = () => {
    const userSettings = {
        classification: [],
        period: []
    }
    const active = document.querySelectorAll('.box__item.active');
    active.forEach(item => {
        const value = item.innerHTML;
        const type = item.dataset.type;
        if (type === 'classification') {
            userSettings.classification.push(value);
        } else if (type === 'period') {
            userSettings.period.push(value);
        }
    })
    return userSettings;
}
{% endhighlight %}

Then we will create the file that stores our settings in /models/Settings.js:

{% highlight js %}
export class Settings {
    constructor() {
        this.userSettings = {
            classification: [],
            period: []
        } 
    }
}
{% endhighlight %}

And store our default data in /models/Data.js:

{% highlight js %}
export const data = {
    classification: ['history', 'portrait', 'landscape', 'still life', 'genre'],
    period: ['modern', 'imperial', 'roman', 'crusdar']
}
{% endhighlight %}

In index.js we will now initialize our app by calling the settings items and creating a new settings instance object.

{% highlight js %}
import '../css/main.scss';
import Settings from './models/Settings';
import { data } from './models/Data';
import { elements } from './views/elements';
import * as paintings from './views/paintingView';
import * as settingsView from './views/settingsView';

const state = {};

// INIT APPLICATION
const init = () => {
    if (!state.settings) state.settings = new Settings();

    // Render data on screen
    data.classification.forEach((el, i) => {
        settingsView.renderSettings(data.classification[i], 'classification');
    })

    data.period.forEach((el, i) => {
        settingsView.renderSettings(data.period[i], 'period');
    })
}

init();
{% endhighlight %}

The toggle functionality on the buttons is now not working anymore because they are rendered after this code has been executed. So we need to call an event listener on it’s parent and then listen if any of the children is called, we call this event bubbling:

{% highlight js %}
// TOGGLE BUTTONS - CHECK CHANGES IN SETTINGS
elements.settings.addEventListener('click', (e) => {
    if (!e.target.classList.contains('box__generate')) {
        const target = e.target.closest('.box__item');
        target.classList.toggle("active");
    }
})
{% endhighlight %}