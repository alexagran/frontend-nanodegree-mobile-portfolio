# Web Optimization

Web Optimization shows progress through Udacity's Front-end Web Developer course by demonstrating the ability to
1. Fix a website to pass Google's Pagespeed Insights mobile and desktop tests; and 
2. Improve website animation to achieve 60 frames per second (fps)


### Installation

Web Optimization requires [source code](https://github.com/alexagran/web-optim.git) to be installed under one folder.
The source code can be installed in any local directory. However, in order to test the page speed required in #1, you'll need a public url for Google's Pagespeed Insights tool to hit. I recommend a tool like [ngrok](https://ngrok.com/) or using [Github Pages](https://pages.github.com/). 

To test my site against Google's Pagespeed Insights
1. Navigate to https://developers.google.com/speed/pagespeed/insights/ 
2. Enter the following url in the "Enter a web page URL" box: https://alexagran.github.io/web-optim/index.html
3. Click the "Analyze" button

To test my site's frame rate optimizations
1. Navigate to the local directory where you downloaded the source code and into the "views" folder
2. Launch pizza.html
3. Use Chrome's developer tools to monitor the frame rate as well as the time to resize the pizzas

### How I optimized page speed

The following are the steps that I took to improve the page speed above 90 for both desktop and mobile

##### Resizing images
profilepic.jpg was compressed
pizzeria.jpg had its size reduced and was compressed

#### Inline and minify CSS
Because my CSS was relatively small, I chose to inline and minify it. It ended up taking up one line in index.html

~~~~ html
<style>html{font-size:100%;overflow-y:scroll;-webkit-tap-highlight-color:rgba(0,0,0,0);-ms-text-size-adjust:100%;-webkit-text-size-adjust:none}body{margin:0;font-size:14px;....</style>
~~~~~

##### Asynchronously loaded javascript where possible
Add the "async" attribute to javascript that could be run asynchronously due to its purpose.

~~~~ javascript
<script async>
  (function(w,g){w['GoogleAnalyticsObject']=g;
  w[g]=w[g]||function(){(w[g].q=w[g].q||[]).push(arguments)};w[g].l=1*new Date();})(window,'ga');

  // Optional TODO: replace with your Google Analytics profile ID.
  ga('create', 'UA-XXXX-Y');
  ga('send', 'pageview');
</script>
<script async src="http://www.google-analytics.com/analytics.js"></script>
<script async src="js/perfmatters.js"></script>
~~~~

##### Prevented unnecessary CSS from loading
Using media queries, I was able to remove the print.css from loading
~~~~ html
<link href="css/print.css" rel="stylesheet" media="print">
~~~~~
##### Google Fonts blocking above the fold rendering
In order to solve the issue with Google fonts impacting the critical rendering path, I used Googles Web Font Loader API to make the call asynchronously.
~~~~ javascript
<script src="https://ajax.googleapis.com/ajax/libs/webfont/1.5.18/webfont.js"></script>
<script>
     WebFont.load({
        google: {
          families: ['Open Sans:400,700']
        }
      });
</script>
~~~~

### How I improved the frame rate of pizza.html

The following are the steps that I took to improve the frame rate of pizza.html as well as the pizza resize time

#### Reduced the number of pizzas on the page
200 pizzas were being added as background to the pages onload when only 32 ever show at a time.

~~~~ javascript
// CHANGES: not sure why it needed 200 pizzas. Made it 32 which 
// seems like is all that is ever on a page at a time
for (var i = 0; i < 32; i++) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
    }
~~~~

#### Separate background pizzas into their own layers
Using will-change CSS
~~~~ css
.mover {
  position: fixed;
  width: 256px;
  z-index: -1;
  will-change: transform;
}
~~~~

#### Used requestAnimationFrame to smooth the animation
~~~~ javascript
// CHANGES: Calling requestAnimationFrame to smooth out the calls to updatePositions
window.addEventListener('scroll', function () {
    window.requestAnimationFrame(updatePositions);
});
~~~~~

#### Updated the updatePositions function
~~~~ javascript
// CHANGES: Moved these out of the for loop as they only needed to be
// determined once
var items = document.getElementsByClassName('mover');
var scrolltop = (document.body.scrollTop / 1250);

for (var i = 0; i < items.length; i++) {
    var phase = Math.sin(scrolltop + (i % 5));
    var translate = items[i].basicLeft + 100 * phase + "px";
    items[i].style.left = translate;
    }
~~~~

#### Cleaned up the initial appending of pizzas onto the page
~~~~ javascript
// CHANGES: pizzasDiv only needs to be determined once and then used in the for loop
var pizzasDiv = document.getElementById("randomPizzas");
for (var i = 2; i < 100; i++) {
    pizzasDiv.appendChild(pizzaElementGenerator(i));
}
~~~~

#### Cleaned up the changePizzaSlices function
~~~~ javascript
function changePizzaSizes(size) {
    // Iterates through pizza elements on the page and changes their widths
    // CHANGES: Moved the newwidth calculation outside of the for loop as it's only needed once
    // Moved the randomPizzaContainer length calc outside as well
    var newwidth = determineDx(size) + '%';
    var len = document.querySelectorAll(".randomPizzaContainer").length
    for (var i = 0; i < len; i++) {
        document.querySelectorAll(".randomPizzaContainer")[i].style.width = newwidth;
    }
}
~~~~

#### Simplified the determinDx function
~~~~ javascript
// CHANGES: Simplified the calculation to determineDx. The width calculations weren't adding anything meaningful 
// to the calculation
function determineDx(size) {
    switch (size) {
        case "1":
            return 25;
        case "2":
            return 33;
        case "3":
            return 50;
        default:
            return ("bug in sizeSwitcher");
    }
}
~~~~

