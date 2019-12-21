---
layout: post
title:  "Creating an art recommending web app using the Harvard Art API - part 3: API"
date:   2019-12-17 15:12:58 +0100
categories: javascript
---
## 6. Setting up the API

### 6.1 Async and adding loading spinner

For retrieving data from the API we need an asynchronous function, because we do not want the rest of our code to stop. Change the controlsettings function in index to the following:

{% highlight js %}
// SAVE NEW SETTINGS
const controlSettings = async () => {

    // Remove current paintings
    paintingView.clear();

    // Render loader icon
    paintingView.renderLoader();

    // Retrieve settings from settingsView
    const newSettings = settingsView.getSettings();

    // Update state with new settings
    state.settings.userSettings = newSettings;

    // New Search object and add to state
    state.search = new Search(newSettings);

    paintingView.renderPaintings('test');
}
{% endhighlight %}

Now we will add the methods in the paintingView file by adding the following code:

{% highlight js %}
// CLEAR PAINTINGS

export const clear = () => {
    elements.paintings.forEach(painting => {
        painting.style.opacity = 0;
    })
}

// RENDER LOADER

export const renderLoader = () => {
    const loader = '<div class="lds-dual-ring"></div>';
    elements.artWrapper.insertAdjacentHTML('afterbegin', loader);
}
{% endhighlight %}

Our elements.js now contains a couple more query selectors:

{% highlight js %}
export const elements = {
    settings: document.querySelector('.settings'),
    buttons: document.querySelectorAll('.box__item'),
    arrowLeft: document.querySelector('.circle__left'),
    arrowRight: document.querySelector('.circle__right'),
    artWrapper: document.querySelector('.art__wrapper'),
    paintings: document.querySelectorAll('.painting'),
    paintingImg: document.querySelectorAll('.painting img'),
    generate: document.querySelector('.box__generate'),
    classification: document.querySelector('.classification'),
    period: document.querySelector('.period'),
};
{% endhighlight %}

And add the following code for the loader spinner in main.scss:

{% highlight scss %}
// Loader spinner
.lds-dual-ring {
    display: inline-block;
    width: 80px;
    height: 80px;
    position: absolute;
    z-index: 1;
    color: $color1;
}
.lds-dual-ring:after {
    content: " ";
    display: block;
    width: 64px;
    height: 64px;
    margin: 8px;
    border-radius: 50%;
    border: 6px solid $color1;
    border-color: $color1 transparent $color1 transparent;
    animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
}
{% endhighlight %}
  

### 6.2 Retrieving new paintings from the Harvard Art API

We first need to get our API key from Harvard. You can get one here: https://www.harvardartmuseums.org/collections/api

Then we can go to the documentation and see what we have to do:
https://github.com/harvardartmuseums/api-docs

But let’s first set up our API call in our application. Add the following code in the controlSettings method:

{% highlight js %}
// Retrieve paintings
try {
    // 4) Search for paintings
    await state.search.getPaintings();

    // 5) Render results
    paintingView.renderPaintings(state.search.result);
    
} catch (err) {
    alert('Something wrong with the search...');
}
{% endhighlight %}

Then run the command npm install axios this will make it easier for us to do API calls. Then make sure your /models/Search.js looks like this:

{% highlight js %}
import axios from 'axios';
import { key } from '../config';

export default class Search {
    constructor(query) {
        this.query = query;
    }

    async getResults() {
        try {
            const res = await axios(`${proxy}http://food2fork.com/api/search?key=${key}&q=${this.query}`);
            this.result = res.data.recipes;
            // console.log(this.result);
        } catch (error) {
            alert(error);
        }
    }
}
{% endhighlight %}

In the main JS folder, create a file called config.js - here we will place our API key.

{% highlight js %}
export const key = ‘...’;
{% endhighlight %} 

We want to retrieve at least:
The image path
Name of the artist
Name of the painting

Let’s check how we can do that. With an object we have all the information we need:
https://github.com/harvardartmuseums/api-docs/blob/master/sections/object.md

We will try to run a query in Search.js using the following code

{% highlight js %}
async getPaintings() {
    try {
        const res = await axios(`https://api.harvardartmuseums.org/object?person=33430&apikey=${key}`);
        this.result = res.data;
        console.log(this.result);
    } catch (error) {
        alert(error);
    }
}
{% endhighlight %}

Press generate in the app and check your console.log, it works! We received an object will all kinds of data. Now let’s build the correct query.

### 6.3 Retrieving data based on users input

Now we need to actually have the real classifications and periods which Harvard Art uses. Let’s get them from the website so your data file looks like this.

{% highlight js %}
export const data = {
    classification: ['Paintings', 'Photographs', 'Drawings', 'Vessels', 'Prints'],
    period: ['Middle Kingdom', 'Bronze Age', 'Roman period', 'Iron Age']
}
{% endhighlight %}

Our complete Search.js now looks like:

{% highlight js %}
import axios from 'axios';
import { key } from '../config';

export default class Search {
    constructor(settings) {
        this.settings = settings;
    }

    buildQuery(settings) {
        let classification = [];
        settings.classification.forEach(el => classification.push('&classification=' + el));
        classification = classification.toString();

        let period = [];
        settings.period.forEach(el => period.push('&period=' + el));
        period = period.toString();

        let query = classification + period;
        query = query.replace(',', '');
        this.query = query;
    }

    async getPaintings() {
        try {
            this.buildQuery(this.settings);
            const res = await axios(`https://api.harvardartmuseums.org/object?apikey=${key}${this.query}`);
            console.log(res);
            this.result = res.data.records;
            console.log(this.result);
        } catch (error) {
            alert(error);
        }
    }
}
{% endhighlight %}

With our buildQuery function we are setting up our query based on the user settings.

Now let’s render the resulting paintings on the screen, update your renderPaintings function in paintingView with the following:

{% highlight js %}
export const renderPaintings = paintings => {

    // Remove loader
    const loader = document.querySelector(`.lds-dual-ring`);
    if (loader) loader.parentElement.removeChild(loader);

    console.log(paintings);

    // Replace paintings
    elements.paintingImg.forEach((img, i) => {
        img.src = paintings[i].primaryimageurl;
    })

    // Show paintings again
    elements.paintings.forEach(painting => {
        painting.style.opacity = 1;
    })
}
{% endhighlight %}

### 6.4 Combining different user preferences

We have a bug now,  we can not combine any classifications or periods with each other. Only single requests e.g. period=Iron Age is possible unfortunately with the API. We will solve this by limiting the user to 1 classification and 1 period. Then we will filter the data by period.
We can limit the classification and period by changing our button toggle function:

{% highlight js %}
elements.settings.addEventListener('click', (e) => {
    if (!e.target.classList.contains('box__generate')) {
        const activeClassification = document.querySelector('.box__item.active[data-type="classification"]');
        const activePeriod = document.querySelector('.box__item.active[data-type="period"]');
        const target = e.target.closest('.box__item');
        if (target.dataset.type == 'classification' && activeClassification) {
            settingsView.toggle(activeClassification);
        }
        if (target.dataset.type == 'period' && activePeriod) {
            settingsView.toggle(activePeriod);
        }
        settingsView.toggle(target);
    }
})
{% endhighlight %}

And adding the settingsView.toggle method:

{% highlight js %}
export const toggle = target => {
    target.classList.toggle("active");
}
{% endhighlight %}

Now the classification part is working! Let’s filter our data if the user has select a period.

Not so many object actually have a period, so lets change the period to century. You can make a folder wide replace in visual code by using SHIFT+CTRL+F and then search and replace for ‘period’ to ‘century’.

Now the data.js file looks like:

{% highlight js %}
export const data = {
    classification: ['Paintings', 'Jewelry', 'Drawings', 'Vessels', 'Prints'],
    century: ['16th century', '17th century', '18th century', '19th century', '20th century']
}
{% endhighlight %}

Then remove /models/Settings.js as we do not need the settings state anymore, the search state is enough. Also remove it in the index.js file.

Our complete Search.js file then looks like

{% highlight js %}
import axios from 'axios';
import { key } from '../config';

export default class Search {
    constructor(settings) {
        this.settings = settings;
    }

    filterByCentury(results) {   
        const century = this.settings.century.toString();
        const filtered = results.filter(result => result.century == century);
        return filtered;
    }

    async getPaintings() {
        try {
            this.classification = this.settings.classification;
            const res = await axios(`https://api.harvardartmuseums.org/object?apikey=${key}&classification=${this.classification}&size=100`);
            this.result = this.filterByCentury(res.data.records);
        } catch (error) {
            alert(error);
        }
    }
}
{% endhighlight %}

Now we can filter the classification that the user has chosen. The resulting artworks are the same every time, let’s make them random by adding a randomize method in Search.js

{% highlight js %}
randomize(data, limit) {
    let result = [];
    let numbers = [];
    for (let i = 0; i <= limit; i++) {
        const random = Math.floor(Math.random() * data.length);
        if (numbers.indexOf(random) === -1) {
            numbers.push(random);
            result.push(data[random]);
        }
    }
    console.log('result', result);
    return result;
}
{% endhighlight %}

We can filter the limit the data we get back from randomize by the limit variable. The other methods then look like:

{% highlight js %}
filterByCentury(results) {   
    const century = this.settings.century.toString();
    const filtered = results.filter(result => result.century == century);
    const result = this.randomize(filtered, 5);
    return result;
}

async getPaintings() {
    try {
        this.classification = this.settings.classification.toString();
        const res = await axios(`https://api.harvardartmuseums.org/object?apikey=${key}&classification=${this.classification}&size=100`);
        this.result = this.filterByCentury(res.data.records);
    } catch (error) {
        alert(error);
    }
}
{% endhighlight %}

Then we need to update out paintingView.js:

{% highlight js %}
 // RENDER PAINTINGS

export const renderPaintings = paintings => {

    console.log('paintings', paintings);

    // Show paintings again
    elements.paintings.forEach(painting => {
        painting.style.opacity = 1;
    })

    // Replace paintings

    paintings.forEach((painting, i) => {
        const imgPath = paintings[i].primaryimageurl;
        if(imgPath) elements.paintingImg[i].src = imgPath;
    })

    // Remove loader
    const loader = document.querySelectorAll(`.lds-dual-ring`);
    if (loader) {
        loader.forEach(loader => loader.parentElement.removeChild(loader));
    }
}
{% endhighlight %}

### 6.5 Loading default artworks

To load a default query we will add the following method to the init function:

{% highlight js %}
// Render default artworks
settingsView.renderDefault('Prints', '20th century');
controlSettings();
{% endhighlight %}

And the in settingsView we will make the selected items active by toggling their classes. We have to select them again because they are rendered later than elements.js select them.

{% highlight js %}
export const renderDefault = (classification, century) => {
    const buttons = document.querySelectorAll('.box__item');
    buttons.forEach(button => {
        if (button.innerHTML == classification || button.innerHTML == century) {
            button.classList.toggle('active');
        }
    })
}
{% endhighlight %}

Let’s improve our error handling. We can do this by throwing an error back when no images have been found. Also we will place a loading spinner remove function outside the renderPaintings function so we can call it from the controller.

{% highlight js %}
// RENDER PAINTINGS
export const renderPaintings = paintings => {

    if (paintings.length > 1) {

        // Show paintings again
        elements.paintings.forEach(painting => {
            painting.style.opacity = 1;
        })

        // Replace paintings
        paintings.forEach((painting, i) => {
            const imgPath = paintings[i].primaryimageurl;
            if(imgPath) elements.paintingImg[i].src = imgPath;
        })

    } else {
        throw "No images found";
    }
}

// Remove loader
export const removeLoader = () => {
    const loader = document.querySelectorAll(`.lds-dual-ring`);
    if (loader) {
        loader.forEach(loader => loader.parentElement.removeChild(loader));
    }
}
{% endhighlight %}