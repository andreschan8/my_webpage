# Data3 Files: Implementation of database operations and stylized printing
Now we are going to join the database operations from the `data2` files and the entry printing from the `toggler` files. We'll call these files `data3`. Our generic HTML template looks like this:
```html
<!DOCTYPE html>
<html>
	<head>
		<link href="data3.css" rel="stylesheet">
	</head>
	<body>
		<script src="config3.js"></script>
		<script src="data3.js"></script>
	</body>
</html>
```
## (1) Setting up Firestore
We proceed to set up Firestore following the steps mentioned in our previous docs. The dependencies are given by these lines:
```javascript
<script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
<script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
```
Paste them in the `<head>` just before `<link>`. The remaining configuration code is as follows. Paste it in your `config3.js` file.
```javascript
const firebaseConfig =  {
	apiKey: "AIzaSyAuCMK3DsjYAbsycXam3PzcWSJdL33dTW4", 
	authDomain: "guessstructure.firebaseapp.com",  
	projectId: "guessstructure",  
	storageBucket: "guessstructure.appspot.com",  
	messagingSenderId: "1087487664626",
	appId: "1:1087487664626:web:049799ace8fc2f86880b6b",
	measurementId: "G-VMHN2FHF61"  
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore(); // Reference to a firestore database
db.settings({timestampsInSnapshots: true, merge: true}); // Database settings
```
## (2) Adding sentence pairs
Include the mentioned HTML and JS code to add sentence pairs. HTML:
```html
<form id="entry-id">
		<textarea id="en-id" name="enName" placeholder="Enter your English sentence"></textarea>
		<textarea id="es-id" name="esName" placeholder="Enter your Spanish sentence"></textarea>
		<button>Add sentence pair</button>
</form>
```
JS:
```javascript
form = document.querySelector('#entry-id')
// Add sentence pair
form.addEventListener('submit', (e) => {
	e.preventDefault();
	db.collection('sentencePairs').add({
		english: form.enName.value,
		spanish: form.esName.value
	});
});
```
## (3) Print your database
Here is where we have to start merging both works. The `toggler` files deal only with a single rectangle (a single database entry or sentence pair), but we need to create the necessary number of entries. To do this, create a `<div>` element below the `<form>` element and assign it the ID `'table-id'`. This element will be the equivalent of the `<div>` element in the `data2` files.
```html
<div id="table-id">
</div>
```
The next part of the plan is as follows. We select the previous `<div>` element using the `querySelector()` method, we then create the (global) `placeholder` variable, which represents the string we want to hide the English sentences with. We define the functions `renderPairs()` and `toggleVisibility()`. This last one is referenced by the first one. In turn, `renderPairs()` will be called for each real time change made to the database. We'll track these changes using the `onSnapshot()` callback function found in Tutorial #8 of Net Ninja's series. Of course, we've explained this function in previous docs.

The real time functionality code is as follows.
```javascript
db.collection('sentencePairs').orderBy('english').onSnapshot(snapshot => {
	let changes = snapshot.docChanges(); //Track any changes
	changes.forEach(change => {
		if (change.type == 'added') {
			renderPairs(change.doc);
		}
	});
});
```
Paste it below the `form` variable and its event listener. Next, we select the `<div>` element with ID `'table-id'` and do everything else we mentioned.
```javascript
const table = document.querySelector('#table-id')

placeholder = "*****************"
```
Paste the previous block at the top of the file. The `toggleVisibility()` function is
```javascript
function toggleVisibility(rectangle, doc) {
    var x = rectangle.children[0].children[1];
    if (x.textContent === placeholder) {
      x.textContent = doc.data().english;
    } else {
      x.textContent = placeholder;
    }
}
```
Don't worry, we'll explain this function along with the next one, which is the `renderPairs()` function.
```javascript
function renderPairs(doc) {

	let rectangle = document.createElement('div');
	let div1 = document.createElement('div');
	let div2 = document.createElement('div');
	let pEs = document.createElement('p');
	let pEn = document.createElement('p');
	let toggleButton = document.createElement('button');
	let chooseButton = document.createElement('button');
	let deleteButton = document.createElement('button');

	rectangle.classList.add("rectangle");
	div1.classList.add("region");
	div2.classList.add("region");
	div2.classList.add("button");

	pEs.textContent = doc.data().spanish;
	pEn.textContent = placeholder;
	toggleButton.textContent = 'Hide English sentence';
	chooseButton.textContent = 'Choose sentence pair';
	deleteButton.textContent = 'Delete sentence pair';

	rectangle.appendChild(div1);
	rectangle.appendChild(div2);
	div1.appendChild(pEs);
	div1.appendChild(pEn);
	div2.appendChild(toggleButton);
	div2.appendChild(chooseButton);
	div2.appendChild(deleteButton);

	toggleButton.setAttribute('type','button');
	toggleButton.addEventListener('click', function() {
		toggleVisibility(rectangle, doc);
	});

	table.appendChild(rectangle);
}
```
All right. Let's explain the two previous functions. The `renderPairs()` function takes a doc (i.e. a sentence pair) as an argument. This function creates the necessary HTML structure to print a sentence pair. It creates a parent `<div>` element called `rectangle`, which has two children `<div>` elements (`div1` and `div2`). `div1` will contain both sentences, each one located in a different `<p>` element (created shortly after). Meanwhile, `div2` will contain the three required buttons. We then add the class attributes for each of these elements. Note that `div2` has two classes, which are necessary in order for the CSS file to work properly. We then add the text content of each element. For the Spanish sentence we use the `doc` parameter, and for the English sentence we add the `placeholder` variable. Next, we append each child to its corresponding location. Note that the `rectangle` variable is added last. Also, as a good habit, we've added the `'button'` type to the `toggleButton`.

For the toggling functionality we add an event listener to the `toggleButton` for the event `'click'`. We call the `toggleVisibility()` function, which takes `rectangle` and `doc` as its parameters. This function uses the `rectangle` element as a reference to the `<p>` element in which the English sentence to be hidden/unhidden is located. (We tried it using IDs with no luck.) Said `<p>` element is accessed using the `children[]` indexation notation. Finally, we hide or unhide the sentence accordingly.
## (4) Deleting functionality
We'll follow the steps indicated in previous docs. We have already creating our Delete button, but we haven't set the `'data-id'` attribute to our `rectangle`. Add the following snippet just before the `setAttribute()` method applied to the `toggleButton`.
```javascript
rectangle.setAttribute('data-id',doc.id);
```
Now we implement the following function, which adds the deleting functionality to our database printing function. Paste it after the `table.appendChild(rectangle)` line, inside the `renderPairs()` function.
```javascript
//Deleting functionality, after the table.appendChild(rectangle) line
deleteButton.addEventListener('click', (e) => {
e.stopPropagation();
let id = e.target.parentElement.parentElement.getAttribute('data-id');
//I think the following line adds the 'removed' status
db.collection('sentencePairs').doc(id).delete();
});
```
Note that we use the attribute `parentElement` twice, since the `deleteButton` is inside two nested `<di>` elements, and we needed the exterior one (the `rectangle`). Now we should be able to delete sentence pairs from the front end, but we won't see it in real time. We have to modify the real time listener in order to see the deleting process taking place. The real time listener is modified as follows.
```javascript
db.collection('sentencePairs').orderBy('english').onSnapshot(snapshot => {
	let changes = snapshot.docChanges(); //Track any changes
	changes.forEach(change => {
		if (change.type == 'added') {
			renderPairs(change.doc);
		} else if (change.type == 'removed') {
			let rectangle = document.querySelector('[data-id=\"' + change.doc.id + '\"]')
			table.removeChild(rectangle)
		}
	});
});
```
Note that we've added an `else if` statement (which was not present when we first included the real time listener). The way it works is of course explained in previous docs.
Lastly, as for the CSS file, add the same contents as those from the `toggler` files:
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

.button {
  display: flex;
  justify-content: space-between;
}
```
_Note: in our_ `toggler` _files, I think I added the_ `button` _class instead of_ `buttons` _, but whatever, it doesn't change too much. Here I've modified the CSS file to affect our buttons by changing '.buttons' to '.button'._
## (6) Checkpoint files
These are our current files. HTML:
```html
<!DOCTYPE html>
<html>
	<head>
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
		<link href="data3.css" rel="stylesheet">
	</head>
	<body>
        <form id="entry-id">
            <textarea id="en-id" name="enName" placeholder="Enter your English sentence"></textarea>
            <textarea id="es-id" name="esName" placeholder="Enter your Spanish sentence"></textarea>
            <button>Add sentence pair</button>
        </form>
        <div id="table-id">
        </div>
		<script src="config3.js"></script>
		<script src="data3.js"></script>
	</body>
</html>
```
JS:
```javascript
const table = document.querySelector('#table-id')

placeholder = "*****************"

function toggleVisibility(rectangle, doc) {
    var x = rectangle.children[0].children[1];
    if (x.textContent === placeholder) {
      x.textContent = doc.data().english;
    } else {
      x.textContent = placeholder;
    }
}

function renderPairs(doc) {

	let rectangle = document.createElement('div');
	let div1 = document.createElement('div');
	let div2 = document.createElement('div');
	let pEs = document.createElement('p');
	let pEn = document.createElement('p');
	let toggleButton = document.createElement('button');
	let chooseButton = document.createElement('button');
	let deleteButton = document.createElement('button');

	rectangle.classList.add("rectangle");
	div1.classList.add("region");
	div2.classList.add("region");
	div2.classList.add("button");

	pEs.textContent = doc.data().spanish;
	pEn.textContent = placeholder;
	toggleButton.textContent = 'Hide English sentence';
	chooseButton.textContent = 'Choose sentence pair';
	deleteButton.textContent = 'Delete sentence pair';

	rectangle.appendChild(div1);
	rectangle.appendChild(div2);
	div1.appendChild(pEs);
	div1.appendChild(pEn);
	div2.appendChild(toggleButton);
	div2.appendChild(chooseButton);
	div2.appendChild(deleteButton);

	rectangle.setAttribute('data-id',doc.id);
	toggleButton.setAttribute('type','button');
	toggleButton.addEventListener('click', function() {
		toggleVisibility(rectangle, doc);
	});

	table.appendChild(rectangle);
	//Deleting functionality, after the table.appendChild(rectangle) line
	deleteButton.addEventListener('click', (e) => {
	e.stopPropagation();
	let id = e.target.parentElement.parentElement.getAttribute('data-id');
	//I think the following line adds the 'removed' status
	db.collection('sentencePairs').doc(id).delete();
	});
}

form = document.querySelector('#entry-id')
// Add sentence pair
form.addEventListener('submit', (e) => {
	e.preventDefault();
	db.collection('sentencePairs').add({
		english: form.enName.value,
		spanish: form.esName.value
	});
});

db.collection('sentencePairs').orderBy('english').onSnapshot(snapshot => {
	let changes = snapshot.docChanges(); //Track any changes
	changes.forEach(change => {
		if (change.type == 'added') {
			renderPairs(change.doc);
		} else if (change.type == 'removed') {
			let rectangle = document.querySelector('[data-id=\"' + change.doc.id + '\"]')
			table.removeChild(rectangle)
		}
	});
});
```
CSS:
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
  
  .button {
    display: flex;
    justify-content: space-between;
  }
```
