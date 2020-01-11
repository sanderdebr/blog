---
layout: post
title:  "Creating an art recommending web app using the Harvard Art API - part 6: Code review & Deployment"
date:   2019-12-20 15:12:58 +0100
categories: vanillajs
---

## 9. Code review

### 9.1 Removing empty paintings

In our renderPaintings function we’ll check for each painting if the imgPath is known, if not the painting will be set to display=none. We have to set the other paintings back to display=block:

{% highlight js %}
if(imgPath) {
    elements.paintings[i].style.display = 'block';
    elements.paintingImg[i].src = imgPath;
    elements.paintingImg[i].parentNode.setAttribute('data-year', year);
    elements.paintingImg[i].parentNode.setAttribute('data-desc', desc);
    elements.paintingImg[i].parentNode.setAttribute('data-artist', artist);
    elements.paintingImg[i].parentNode.setAttribute('data-division', division);
    elements.paintingImg[i].parentNode.setAttribute('data-objectnumber', objectnumber);
} else {
    x++;
    elements.paintings[i].style.display = 'none';
    console.log(elements.paintings.length);
    if (x == elements.paintings.length) detailView.noResults();
}
{% endhighlight %}

### 9.2 Showing painting loader spinner

Add class ‘loading’ to each image in the index.html and we will add a background to this image, so the user will see the loading spinner untill the image is ready and will overlay the background.

{% highlight scss %}
.loading {
    background: transparent url('/img/spinner.gif') center no-repeat;
}
{% endhighlight %}

### 9.3 Showing loading spinner on startup

For this we will add an overlay div on top of everything else, and remove the overlay when the window has loaded fully.

{% highlight html %}
<div class="overlay"><h1>Loading Smartart application</h1></div>
{% endhighlight %}

{% highlight scss %}
.overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 999;
    background: #f4f4f4;
    display: flex;
    align-items: center;
    justify-content: center;
}
{% endhighlight %}

At the end of our index file we add:

{% highlight js %}
// Remove layer when all content has loaded
window.addEventListener('load', function() {
    elements.overlay.style.display = "none";
})
{% endhighlight %}

Done!

## 10. Deployment

Let’s deploy our app to an AWS server with continuous deployment. Meaning every time we commit to our github repository, our live app gets updated as well.

### 10.1 AWS Amplify

AWS amplify makes sure every time you’ll commit to you github repository, the webapp get updated. THis is called continuous deployment.

### 10.2 FTP

Upload all the contents from your /dist folder to your FTP and you are done!
