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
### 2.1. Checkpoint files from Theory and Data3
We start from the last checkpoint files from the docs of the `theory` (along with the small improvement preventing autocapitalization) and `data3` with stylized printing files. **IMPORTANT: WE'VE RENAMED** `config3.js` **AS** `config.js` **IN THIS SECTION**. For easier reference, here they are:
`theory.html`:
```html
<!DOCTYPE  html>
<html>
	<head>
		<link  href="theory.css"  rel="stylesheet"  />
	</head>
	<body>
		Enter the original English text:<br>
		<textarea  id="englishText"></textarea><br>
		<button  type="button"  id="englishButton" onclick=printStructure()>Done</button><br>
		Enter the translated Spanish text:<br>
		<textarea  id="spanishText"></textarea><br>
		<button  type="button"  id="spanishButton"  onclick="printSpanish()">Done</button><br>
		<p  id="spanishLegend"></p>
		<p  id="englishStructure"></p>
		<!--Our files share the name 'theory'-->
		<script  src="theory.js"></script>
	</body>
</html>
```
`theory.js`:
```javascript
function printSpanish() {
	document.getElementById("spanishLegend").innerHTML = ""
	var text = document.getElementById("spanishText").value
	document.getElementById("spanishLegend").innerHTML = text
}

function printStructure() {
	var text = document.getElementById("englishText").value
	document.getElementById("englishStructure").innerHTML = ""
	if (text == "") {
		var errorString = "You have entered no English text"
		document.getElementById("englishStructure").innerHTML = errorString
	}
	else {
		words = text.split(" ")
		word_array = words.slice(0)
		for (var i = 0; i<words.length; i++) {
			var input = document.createElement("input")
			input.type = "text"
			input.size = words[i].length
			input.className = "indivBox"
            input.autocapitalize = "none"
			document.getElementById("englishStructure").appendChild(input)
		}
	}
	textboxManager()
}

function textboxManager() {
	const textboxes = document.querySelectorAll(".indivBox")
	textboxes.forEach((textbox, index) => {
		let  lastValue  =  textbox.value;
		textbox.addEventListener('keydown', (event) => {
			if (event.key  ===  'Backspace'  &&  textbox.selectionStart  ===  0) {
				const  previousTextbox  =  textboxes[index  -  1];
				if (previousTextbox) {
					if (!(event.repeat)){
						event.preventDefault();
					}
					previousTextbox.focus();
					previousTextbox.selectionStart  =  previousTextbox.value.length;
				}
			}
		})
		textbox.addEventListener('input', (event) => {
			if (textbox.value === words[index]) {
				textbox.classList.add("right")
		
				const nextTextbox = textboxes[index + 1]
				if (nextTextbox) {
					nextTextbox.focus()
				}
			}
			else {
				textbox.classList.remove("right")
			}
			if (textbox.value !== lastValue + ' ' && textbox.value !== lastValue.slice(0, -1)) {
				lastValue = textbox.value;
				return;
			}
	
			if (textbox.value === lastValue + ' ') {
				textbox.value = lastValue.trim();
				const nextTextbox = textboxes[index + 1];
				if (nextTextbox) {
					nextTextbox.focus();
				}
			}
			lastValue = textbox.value;
		});
	
		textbox.addEventListener('keydown', (event) => {
			if (event.key  ===  ' '  &&  textbox.selectionStart  !==  textbox.value.length) {
				event.preventDefault();
			}
		});
	})
}

let word_array
```
`theory.css`:
```css
.indivBox {
	text-align: center
}

input.right {
	border: 2px  solid  rgb(33, 254, 85)
}
```
`data3.html`:
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
		<script src="config.js"></script>
		<script src="data3.js"></script>
	</body>
</html>
```
`data3.js`:
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
`data3.css`:
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
### 2.2. Merging
We add the line `<button type="button" onclick="goToSenMan()">Go to Sentence Manager</button>` between the button with ID `spanishButton` and the next `<br>` element. That is,
```javascript
<button  type="button"  id="spanishButton"  onclick="printSpanish()">Done</button>
<button type="button" onclick="goToSenMan()">Go to Sentence Manager</button><br>
```
At the very top of `theory.js`, place the corresponding function:
```javascript
function goToSenMan() {
    window.location.href = 'data3.html';
};
```
Now we can access the Sentence Manager, that is, the `data3` files. In these same files, we need to add functionality to the _Choose sentence pair_ button. We do this as follows. Inside the `renderPairs()` functions in `data3.js`, immediately after the event listener of the `toggleButton`, add this block:
```javascript
chooseButton.addEventListener('click', function() {
        goToStructGuess(doc);
});
```
Now place the definition of this function before that of the `toggleVisibility()` function.
```javascript
function goToStructGuess(doc) {
    db.collection('userLogs').add({
        english: doc.data().english,
        spanish: doc.data().spanish
    })
    .then(function() {
        window.location.href = 'theory.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};
```
This function adds the English and Spanish sentences to a new document in the `userLogs` collection and redirects the user to the Structure Guesser page (i.e. the `theory.html` file).

Now we have to retrieve the user log from the firestore database immediately after the Structure Guesser page loads. In order to do this we have to include the `<script>` snippets in the head of the `theory.html` file and the `config.js` in the body.
```javascript
//Before the link tag
<script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
<script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
```
```javascript
<script src="config.js"></script>  //Before the theory.js script tag
```
Now, in the `theory.js` file, include this block of code at the very top.
```javascript
let dummyID = 'dummy-id';

window.onload = function () {
	db.collection('userLogs').where(firebase.firestore.FieldPath.documentId(),'!=',dummyID).get().then((snapshot)=>{
        snapshot.docs.forEach(doc=>{
            let enSen = document.querySelector('#englishText');
	    let esSen = document.querySelector('#spanishText');
            enSen.textContent = doc.data().english;
	    esSen.textContent = doc.data().spanish;
            let logID = doc.id;
            db.collection('userLogs').doc(logID).delete();
        });
    });
};
```
This function comes from the `about.js` file, and inserts the English and Spanish sentences in the textareas with IDs `'englishText'` and `'spanishText'`. Of course, after it has inserted the sentences, it deletes the user log in the firestore collection.

This finally completes the basic functionality of the Structure Guesser web app. The following docs will discuss further improvements.

## (3) Checkpoint files (v1.0)

The files of the first functioning version are divided into the `theory` and `data3` files. For the `theory` files:
HTML:
```html
<!DOCTYPE  html>
<html>
	<head>
		<script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
		<link  href="theory.css"  rel="stylesheet"  />
	</head>
	<body>
		Enter the original English text:<br>
		<textarea  id="englishText"></textarea><br>
		<button  type="button"  id="englishButton" onclick=printStructure()>Done</button><br>
		Enter the translated Spanish text:<br>
		<textarea  id="spanishText"></textarea><br>
		<button  type="button"  id="spanishButton"  onclick="printSpanish()">Done</button>
        <button type="button" onclick="goToSenMan()">Go to Sentence Manager</button><br>
		<p  id="spanishLegend"></p>
		<p  id="englishStructure"></p>
		<!--Our files share the name 'theory'-->
		<script src="config.js"></script>
		<script  src="theory.js"></script>
	</body>
</html>
```
JS:
```javascript
let dummyID = 'dummy-id';

window.onload = function () {
	db.collection('userLogs').where(firebase.firestore.FieldPath.documentId(),'!=',dummyID).get().then((snapshot)=>{
        snapshot.docs.forEach(doc=>{
            let enSen = document.querySelector('#englishText');
			let esSen = document.querySelector('#spanishText');
            enSen.textContent = doc.data().english;
			esSen.textContent = doc.data().spanish;
            let logID = doc.id;
            db.collection('userLogs').doc(logID).delete();
        });
    });
};

function goToSenMan() {
    window.location.href = 'data3.html';
};

function printSpanish() {
	document.getElementById("spanishLegend").innerHTML = ""
	var text = document.getElementById("spanishText").value
	document.getElementById("spanishLegend").innerHTML = text
}

function printStructure() {
	var text = document.getElementById("englishText").value
	document.getElementById("englishStructure").innerHTML = ""
	if (text == "") {
		var errorString = "You have entered no English text"
		document.getElementById("englishStructure").innerHTML = errorString
	}
	else {
		words = text.split(" ")
		word_array = words.slice(0)
		for (var i = 0; i<words.length; i++) {
			var input = document.createElement("input")
			input.type = "text"
			input.size = words[i].length
			input.className = "indivBox"
            input.autocapitalize = "none"
			document.getElementById("englishStructure").appendChild(input)
		}
	}
	textboxManager()
}

function textboxManager() {
	const textboxes = document.querySelectorAll(".indivBox")
	textboxes.forEach((textbox, index) => {
		let  lastValue  =  textbox.value;
		textbox.addEventListener('keydown', (event) => {
			if (event.key  ===  'Backspace'  &&  textbox.selectionStart  ===  0) {
				const  previousTextbox  =  textboxes[index  -  1];
				if (previousTextbox) {
					if (!(event.repeat)){
						event.preventDefault();
					}
					previousTextbox.focus();
					previousTextbox.selectionStart  =  previousTextbox.value.length;
				}
			}
		})
		textbox.addEventListener('input', (event) => {
			if (textbox.value === words[index]) {
				textbox.classList.add("right")
		
				const nextTextbox = textboxes[index + 1]
				if (nextTextbox) {
					nextTextbox.focus()
				}
			}
			else {
				textbox.classList.remove("right")
			}
			if (textbox.value !== lastValue + ' ' && textbox.value !== lastValue.slice(0, -1)) {
				lastValue = textbox.value;
				return;
			}
	
			if (textbox.value === lastValue + ' ') {
				textbox.value = lastValue.trim();
				const nextTextbox = textboxes[index + 1];
				if (nextTextbox) {
					nextTextbox.focus();
				}
			}
			lastValue = textbox.value;
		});
	
		textbox.addEventListener('keydown', (event) => {
			if (event.key  ===  ' '  &&  textbox.selectionStart  !==  textbox.value.length) {
				event.preventDefault();
			}
		});
	})
}

let word_array
```
CSS:
```css
.indivBox {
	text-align: center
}

input.right {
	border: 2px  solid  rgb(33, 254, 85)
}
```
For the `data3` files:
HTML:
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
		<script src="config.js"></script>
		<script src="data3.js"></script>
	</body>
</html>
```
JS:
```javascript
const table = document.querySelector('#table-id')

placeholder = "*****************"

function goToStructGuess(doc) {
    db.collection('userLogs').add({
        english: doc.data().english,
        spanish: doc.data().spanish
    })
    .then(function() {
        window.location.href = 'theory.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};

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
    chooseButton.addEventListener('click', function() {
        goToStructGuess(doc);
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
Finally, for the `config.js` file:
```javascript
const firebaseConfig = {
    apiKey: "AIzaSyAuCMK3DsjYAbsycXam3PzcWSJdL33dTW4",
    authDomain: "guessstructure.firebaseapp.com",
    projectId: "guessstructure",
    storageBucket: "guessstructure.appspot.com",
    messagingSenderId: "1087487664626",
    appId: "1:1087487664626:web:049799ace8fc2f86880b6b",
    measurementId: "G-VMHN2FHF61"
};

firebase.initializeApp(firebaseConfig);

const db = firebase.firestore();
db.settings({ timestampsInSnapshots: true, merge: true });
```

















