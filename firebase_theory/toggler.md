# Toggler Theory
We'd like a better menu from which we can select the sentence pair we want to study. We will now create this menu along with some cool functionalities.

## (1) Initial files
We'll use this HTML template. Create three documents: `toggler.html`, `toggler.css` and `toggler.js`.
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <link href="toggler.css" rel="stylesheet" />
  </head>
  <body>
    <script src="toggler.js"></script>
  </body>
</html>
```
## (2) Toggling functionality
Now create a parent `<div>` element and two sibling ones. The parent has a class attribute called `rectangle`; the first child should have the class `region` and the second child should have the class list `buttons region`. Also, include two `<p>` elements inside the first `<div>` and a button in the other one. Write your sample sentence in the first `<p>` element and leave the second one empty, but include an ID for this one, such as `toggled-id`. This `<p>` element will be the toggling target and we leave it empty since the JS code will write a placeholder in it when the page loads. Finally, add the `onclick` attribute to the button. Call the function `toggleVisibility()` and pass in the ID of the English sentence `<p>` element.
```html
<div class="rectangle">
        <div class="region">
            <p>This will be my Spanish sentence</p>
            <p id="toggled-id"></p>
        </div>
        <div class="buttons region">
            <button type="button" onclick="toggleVisibility('toggled-id')">My Button</button>
        </div>
</div>
```
For the CSS file paste the following code. Note that I barely care about this file.
```css
.rectangle {
    display: flex;
    justify-content: space-between;
    border: 1px solid black;
    margin-bottom: 10px;
    padding: 10px;
  }
  
  .region {
    flex: 1;
    margin-right: 10px;
  }
  
  .region p {
    margin-bottom: 10px;
  }
  
  .buttons {
    display: flex;
    justify-content: space-between;
  }
```
As for the JS file, we create two string variables. The first one is our English sentence. **In future implementations we'll retrieve the value of this variable from our database.** The second variable will be the placeholder for when the English sentence is hidden.
```javascript
let my_en_sen = "This is my English sentence";
let placeholder = "*******************";
```
We now need to include the placeholder in the second `<p>` element when the page loads. To do this we use an event listener to the DOM. The event is `'DOMContentLoaded'`. As its name suggests, this event triggers when the DOM elements have fully loaded (according to Copilot). Next, we store the `<p>` element with ID `toggled-id` inside a variable called `hiddenOnLoad` and we change its text content, setting it equal to `placeholder`.
```javascript
document.addEventListener('DOMContentLoaded', (event) => {
    let hiddenOnLoad = document.getElementById('toggled-id');
    hiddenOnLoad.textContent = placeholder;
});
```
Now we need to add the toggling functionality. Paste the following function below the event listener.
```javascript
function toggleVisibility(id) {
    var x = document.getElementById(id);
    if (x.textContent === placeholder) {
      x.textContent = my_en_sen;
    } else {
      x.textContent = placeholder;
    }
}
```
The `x` variable references the `<p>` element corresponding to the English sentence. If its text content is the placeholder, we reveal the English sentence. Otherwise, we hide it.
















