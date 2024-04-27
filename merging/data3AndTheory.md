# Data 3 and Theory files: Linking both pages
We are almost ready to merge both pages, but first we need to know how to send data between two different sites.
## (1) Communication between two pages
In the same folder (which we've called `homeAbout`) create 7 new files, namely, `home.html`, `home.js`, `home.css`, `about.html`, `about.js`, `about.css` and `config.js`. These files represent the _Home_ and _About_ sections of a website.

The plan is to store temporarily the sentence pair selected by the user in order to enter it in the `textarea` elements of the `theory` files. We'll use Firestore for this purpose, since the `localStorage` suggested by Copilot seems to be unreliable while working with the `file:` protocol and even when using the incognito mode in most browsers.

We've created a new Firestore collection called `userLogs` that contains a dummy document that prevents the collection from being deleted automatically when it is empty. The fields of its documents are `english` and `spanish`, similar to the `sentencePairs` collection.

### 1.1. Initial files
The `home.html` and `about.html` files share a common initial structure, where the two `script` tags in the `head` are present, since both files will interact with the database. We show here the `home.html` file only. **Just replace "home" with "about" when referencing the CSS and JS files to get the HTML file of the _About_ section.**
```html
<!DOCTYPE html>
<html>
	<head>
		<link href="home.css" rel="stylesheet">
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
	</head>
	<body>
		<script src="config.js"></script>
		<script src="home.js"></script>
	</body>
</html>
```
The `config.js` file is the same as in previous docs. All other files remain empty for now.
### 1.2. Sending user logs to the database
In the `home.html` file, paste these lines as the first part of the `body`.
```html
<input type="text" id="userInput">
<button onclick="goToAbout()">Print your input in the About section</button>
```
We have a textbox and a button, and we want to send the user's input to the database so we can retrieve it later from the _About_ section and print it on the screen. Add the following code to your `home.js` file.
```javascript
function goToAbout() {
    db.collection('userLogs').add({
        english: document.getElementById('userInput').value,
        spanish: ""
    })
    .then(function() {
        window.location.href = 'about.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};
```
This function adds the value of the `<input>` element as the value of the `english` field of the `userLogs` collection. We have left the `spanish` field empty since we only need one sentence. It's important to mention that the `add()` method has an asynchronous nature, meaning that takes some considerable time to complete (a.k.a. a _promise_). For this reason, it's possible that the following lines fire before this method has ended. Actually, the `window.location.href = 'about.html'` line was placed initially without the `.then()` method, and the page only redirected to the _About_ section, without adding the input to the database. Note the unusual syntax, where both the `.then()` and `.catch()` methods are placed in new lines.
_**Interesting:** first I tried using the_ `textContent` _attribute instead of_ `value` _but it wouldn't work._
### 1.3. Retrieving user logs from the database
Now we'll focus on the `about` files. We want to print the user's input when the page loads. We create a `<p>` element with ID `display-id` in the `about.html` file.
```html
<p id="display-id"></p>
```
Next, paste the following block of code in the `about.js` file.
```javascript
let dummyID = 'dummy-id';

function displayNDelete () {
    
    db.collection('userLogs').where(firebase.firestore.FieldPath.documentId(),'!=',dummyID).get().then((snapshot)=>{
        snapshot.docs.forEach(doc=>{
            let p = document.querySelector('#display-id');
            p.textContent = doc.data().english;
            let logID = doc.id;
            db.collection('userLogs').doc(logID).delete();
        });
    });
};

displayNDelete();
```
In Firestore we defined a custom ID for the dummy document, called `dummy-id`. We create the `dummyID` variable containing this ID, and then we define the function `displayNDelete()`, which displays the input in the _About_ section and then deletes the user log. Note that we used the `where()` method to specify the documents with an ID different to `'dummy-id'` by means of the `firebase.firestore.FieldPath.documentId()` expression, found in Stack Overflow. Right now I don't have the time to search more information about this expression. The `get()` method returns a promise handled by the `then()` method, and we then iterate over the only document different from the dummy one using a `forEach` statement. We select the `<p>` element and display the user log in it. We store its ID and use it to delete the document using `db.collection('userLogs').doc(logID).delete();`. Note that something like `doc.delete()` wouldn't work (although there was no error thrown), since it would be affecting the snapshot and not the database itself. And, finally, we call the function.

## (2) Merging of Data3 and Theory files.
We start from the last checkpoint files from the docs of the `theory` and `data3` files.






















