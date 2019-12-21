---
layout: post
title:  "Creating an art recommending web app using the Harvard Art API - part 5: Likes & LocalStorage"
date:   2019-12-19 15:12:58 +0100
categories: javascript
---

## 8. Adding likes to local storage

In this section we will add like abilities to each rendered painting, so we can give the user more relevant artworks in their following rendered. We will keep track of our like ratio to see if we are actually recommending better artworks. We will store this information in the local storage of the users browser, which will stay there even after they have closed the website.

### 8.1 Rendering likes per painting

First we will add the HTML and CSS for the likes element, we’ll add them just after the painting image tags:

{% highlight html %}
<div class="like"><ion-icon name="heart-empty"></ion-icon></div>
{% endhighlight %}

{% highlight scss %}
.like {
    position: absolute;
    margin: 0 auto;
    text-align: center;
    width: 100%;
    margin-top: 30px;
    transition: 175ms ease;
    &:hover {
        transform: scale(1.5);
    }
}

ion-icon {
    color: $color1;
    font-size: 33px;
}
{% endhighlight %}

Let’s add the eventListener in our controller file, where we also handle our heart icon:

{% highlight js %}
const controlLike = (e) => {
    let isLiked;

    if (!state.likes) state.likes = new Likes();

    if (e.target.name === 'heart-empty') {
        isLiked = true;
        e.target.name = 'heart';
    } else {
        isLiked = false;
        e.target.name = 'heart-empty';
    }
}
{% endhighlight %}

Then add a division data attribute and an object number data attribute in the paintingView so we can retrieve more information about the artwork. We will save the object number in our likes array.

Our controller now looks like this:

{% highlight js %}
// LIKE CONTROLLER

const controlLike = (e) => {
    let isLiked, objectnumber, division;

    if (!state.likes) state.likes = new Likes();

    objectnumber = e.target.parentNode.parentNode.dataset.objectnumber;
    division = e.target.parentNode.parentNode.dataset.division;

    // LIKE PAINTING
    if (e.target.name === 'heart-empty') {
        isLiked = true;
        e.target.name = 'heart';

        state.likes.addLike(objectnumber, division);
    
    // DISLIKE PAINTING
    } else {
        isLiked = false;
        e.target.name = 'heart-empty';

        state.likes.removeLike(objectnumber);
    }

    console.log(state.likes);

}
{% endhighlight %}

Then in our Likes class model we will add the methods to add or delete a like of an artwork based on the click event in the controller. We will add a new object containing the object number and division to our likes array which is stored in our likes state array.

{% highlight js %}
export default class Likes {
    constructor() {
        this.likes = [];
    }

    addLike(objectnumber, division) {
        const obj = {
            object: objectnumber,
            division: division
        }
        this.likes.push(obj);
        return obj;
    }

    removeLike(objectnumber) {
        const index = this.likes.findIndex(likes => likes.object === objectnumber);
        this.likes.splice(index, 1);
    }

}
{% endhighlight %}

And it is working!

### 8.2 Saving likes and storing them in localStorage

We will store the likes in the local storage of the users browser so that when the user closes and re-opens the application, the likes still appear. We will update our Likes model so it stores the state.likes array inside the localstorage:

{% highlight js %}
export default class Likes {
    constructor() {
        this.likes = [];
    }

    storeData() {
        localStorage.setItem('likes', JSON.stringify(this.likes));
    }

    readStorage() {
        const storage = JSON.parse(localStorage.getItem('likes'));
        // Restoring likes from the localStorage
        if (storage) this.likes = storage;
    }

    addLike(objectnumber, division) {
        const obj = {
            object: objectnumber,
            division: division
        }
        this.likes.push(obj);

        // Store data in local storage
        this.storeData();

        return obj;
    }

    removeLike(objectnumber) {
        const index = this.likes.findIndex(likes => likes.object === objectnumber);
        
        // Store data in local storage
        this.storeData();

        this.likes.splice(index, 1);
    }

}
{% endhighlight %}

Then in our init function we will add the following so a likes state gets created on page load:

{% highlight js %}
 // Render default likes
state.likes = new Likes();

// Restore likes
state.likes.readStorage();

console.log(state.likes);
{% endhighlight %}

Next up we need to render a liked heart if the painting has been liked by the user. We have to update our init function for this to work.

We pass the state.likes in our renderPaintings async function:

{% highlight js %}
// Retrieve paintings
try {
    // Search for paintings
    await state.search.getPaintings();

    // Render results
    paintingView.renderPaintings(state.search.result, state.likes);

    //Remove loader 
    paintingView.removeLoader();

    return true;

} catch (err) {
    console.log(err);
}
{% endhighlight %}

Then in our renderPaintings function itself we search in each painting for a match between the painting objectnumber and the objectnumber we have stored in our likes state:

{% highlight js %}
// Check if painting isLiked
let obj = likes.likes.find(o => o.object == objectnumber);
if (obj) {
    elements.likeButtons[i].name = 'heart';
    const likeBtnArr = Array.from(elements.likeButtons[i].childNodes);
    console.log(likeBtnArr);
    likeBtnArr[0].name = 'heart';
}
{% endhighlight %}

That’s it!

### 8.3 Recommending better artworks based on likes

It would be great if we could recommend the user artwork based on his likes. We will do that using our stores likes state which contains division information about each artwork, e.g. “Modern and Contemporary Art”.

- to be added