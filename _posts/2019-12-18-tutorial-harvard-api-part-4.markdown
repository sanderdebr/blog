---
layout: post
title:  "Creating an art recommending web app using the Harvard Art API - part 4: Art & Error handling"
date:   2019-12-18 15:12:58 +0100
categories: vanillajs
---
## 7. Art detail information and error handling

In this section we will create the bottom part where information about the artwork is displayed, which is currently hard coded information.

### 7.1 Setting up event listener

Let’s add three new query selector in our elements.js file

{% highlight js %}
year: document.querySelector('.year'),
artist: document.querySelector('.content__h2'),
title: document.querySelector('.content__h5')
{% endhighlight %}

Then create a new view file called detailView.js. Here we will add the renderDetails method which we will call in index.js after the user has clicked on an artwork.

In the renderPaintings method we will now set the year, title and description as data attributes on the parent div of the image by the following code:

{% highlight js %}
// Replace paintings
paintings.forEach((painting, i) => {
    const imgPath = paintings[i].primaryimageurl;
    const artist = paintings[i].title;
    const year = paintings[i].accessionyear;
    const desc = paintings[i].medium;
    if(imgPath) {
        elements.paintingImg[i].src = imgPath;
        elements.paintingImg[i].parentNode.setAttribute('data-year', year);
        elements.paintingImg[i].parentNode.setAttribute('data-desc', desc);
        elements.paintingImg[i].parentNode.setAttribute('data-artist', artist);
    }
})
{% endhighlight %}

In index.js we will then send the clicked painting information to the renderDetails function:

{% highlight js %}
// GET ART DETAILS
let newPaintings = document.querySelectorAll('.painting');
newPaintings.forEach(painting => {
   painting.addEventListener('click', () => {
        renderDetails(painting)
   });
});
{% endhighlight %}

In detailView.js we can then easily update our detail information:

{% highlight js %}
import { elements } from './elements';

export const renderDetails = painting => {
    elements.year.innerHTML = painting.dataset.year;
    elements.artist.innerHTML = painting.dataset.artist;
    elements.title.innerHTML = painting.dataset.desc;
}
{% endhighlight %}

Great! Our detail information is working.

### 7.2 Default artwork information with async await

Let’s adSetting up the default information is a bit tricky, because we only can get the data attributes of the artworks when they are loaded. So that is why we’ll be using an async await functionality. Let’s first return true in the controlSettings function when it has been completed. Make sure to include the removeLoader function before calling the return, otherwise the loader will stay on screen. Then in the init function we’ll add:

{% highlight js %}
// Render default information
async function renderDefault() {
    try {
        const response = await controlSettings();
        if (response) {
            let defaultPainting = document.querySelector('.painting:first-child');
            renderDetails(newPaintings[0]);
        }
    }
    catch (err) {
        console.log('renderdefault failed', err);
    }
}
    
renderDefault();  
{% endhighlight %}

The function above waits for the controlSettings function to finish, and then retrieves the artwork data from the rendered artworks.

### 7.3 Error handling 

First let’s remove the current detail information when a new search has been called. We’ll add removeDetails in the detailView.js page

{% highlight js %}
export const clear = () => {
    elements.year.innerHTML = '';
    elements.artist.innerHTML = 'Please select an artwork';
    elements.title.innerHTML = '';
}
{% endhighlight %}

And we’ll add it in the controlSettings function

{% highlight js %}
// Remove current paintings and detail information
paintingView.clear();
detailView.clear();
{% endhighlight %}

Also if no results are returned:

{% highlight js %}
export const noResults = () => {
    elements.year.innerHTML = '';
    elements.artist.innerHTML = 'No results found';
    elements.title.innerHTML = '';
}
{% endhighlight %}

Our renderPaintings then looks like this:

{% highlight js %}
export const renderPaintings = paintings => {

    if (paintings.length > 1) {

            // Show paintings again
            elements.paintings.forEach(painting => {
                painting.style.opacity = 1;
            })

            // Replace paintings
            paintings.forEach((painting, i) => {
                const imgPath = paintings[i].primaryimageurl;
                const artist = paintings[i].title;
                const year = paintings[i].accessionyear;
                const desc = paintings[i].medium;
                if(imgPath) {
                    elements.paintingImg[i].src = imgPath;
                    elements.paintingImg[i].parentNode.setAttribute('data-year', year);
                    elements.paintingImg[i].parentNode.setAttribute('data-desc', desc);
                    elements.paintingImg[i].parentNode.setAttribute('data-artist', artist);
                } 
            })
    } else {
        detailView.noResults();
        removeLoader();
        throw "No images found";
    }
}
{% endhighlight %}

Sometimes null or defined gets displayed on the default artwork. If we do not have that information we’ll ask the user to select an artwork first. This is an easy fix by changing our renderDetails function and checking if all the artwork data is actually available:

{% highlight js %}
export const renderDetails = painting => {
    if (painting.dataset.year && painting.dataset.artist && painting.dataset.desc) {
        elements.year.innerHTML = painting.dataset.year;
        elements.artist.innerHTML = painting.dataset.artist;
        elements.title.innerHTML = painting.dataset.desc;
    } else {
        clear();
    }
}
{% endhighlight %}

### 7.4 Fixing text breaking

Some artworks have a long title and description which breaks our layout. 

![My helpful screenshot](/assets/7.png)

Let’s set the font of .info .content .content__h2 to 2.3em so it still looks good on mobile.

![My helpful screenshot](/assets/8.png)

### 7.5 Adding styles to selected artwork

It’s currently unclear which artwork the user has selected, let’s change that. In our renderDetails method inside detailView.js we’ll call the showCurrent function on the current painting that just has been clicked:

{% highlight js %}
export const renderDetails = painting => {
    if (painting.dataset.year && painting.dataset.artist && painting.dataset.desc) {
        elements.year.innerHTML = painting.dataset.year;
        elements.artist.innerHTML = painting.dataset.artist;
        elements.title.innerHTML = painting.dataset.desc;
        paintingView.showCurrent(painting);
    } else {
        clear();
    }
}
{% endhighlight %}

Then we’ll add the showCurrent painting in paintingView by toggling a painting--active class and adding it on the current painting:

{% highlight js %}
// Show current painting
export const showCurrent = currentPainting => {
    Array.from(elements.paintings).forEach(painting => {
        painting.classList.remove('painting--active');
    });
    currentPainting.classList.add('painting--active');
}
{% endhighlight %}
 
I’ve added the following classes in the css file for the active painting:

{% highlight scss %}
.painting {
    margin: 0 5em;
    transition: all 250ms ease;
    border:solid 2px;
    border-bottom-color:#ffe;
    border-left-color:#eed;
    border-right-color:#eed;
    border-top-color:#ccb;
    transform: scale(.8);
    img {
        margin: 10px;
    }
}
.painting:hover {
    cursor: pointer;
    transform: scale(1.2);
}
.painting--active {
    transform: scale(1.2);
}
{% endhighlight %}

Then for the frame around the artworks, I found a nice little pure CSS code:

{% highlight scss %}
// CSS art frame
.frame {
    background-color:#ddc;
    border:solid 5vmin #eee;
    border-bottom-color:#fff;
    border-left-color:#eee;
    border-radius:2px;
    border-right-color:#eee;
    border-top-color:#ddd;
    box-shadow:0 0 5px 0 rgba(0,0,0,.15) inset, 0 5px 10px 5px rgba(0,0,0,.15);
    box-sizing:border-box;
    display:inline-block;
    max-height: 40vh;
    max-width: 30vw;
    position:relative;
    text-align:center;
    &:before {
      border-radius:2px;
      bottom:-2vmin;
      box-shadow:0 2px 5px 0 rgba(0,0,0,.15) inset;
      content:"";
      left:-2vmin;
      position:absolute;
      right:-2vmin;
      top:-2vmin;
    }
    &:after {
      border-radius:2px;
      bottom:-2.5vmin;
      box-shadow: 0 2px 5px 0 rgba(0,0,0,.15);
      content:"";
      left:-2.5vmin;
      position:absolute;
      right:-2.5vmin;
      top:-2.5vmin;
    }
  }
{% endhighlight %}