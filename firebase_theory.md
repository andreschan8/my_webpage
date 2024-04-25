# Firebase Theory
We have previously written some docs using StackEdit.io. I'm transitioning to github as a result of my ever lasting search for better features for documentation.

##  Remark: we continue from Section (5) (Checkpoint files) of the document _Structure Guesser Firestore Database_ in Alessandro's StackEdit account.

## (5) Checkpoint files
These are our files so far. For the HTML file:
```html
<!DOCTYPE html>
<html>
	<head>
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app-compat.js"></script>  
        <script  src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore-compat.js"></script>
		<link href="data2.css" rel="stylesheet">
	</head>
	<body>
        <form id="entry-id">
            <textarea id="en-id" name="enName" placeholder="Enter your English sentence"></textarea>
            <textarea id="es-id" name="esName" placeholder="Enter your Spanish sentence"></textarea>
            <button>Add sentence pair</button>
        </form>
        <div id="table-id">
            <ul id="table-ulist-id">
            </ul>
        </div>
		<script src="config2.js"></script>
		<script src="data2.js"></script>
	</body>
</html>
```
For the JavaScript file:
```javascript
const ulist = document.querySelector('#table-ulist-id')

function renderPairs(doc) {
	//Create new elements for the database entry
	let li = document.createElement('li');
	let english = document.createElement('span');
	let spanish = document.createElement('span');
	let cross = document.createElement('button');

	//Set the new attribute to li and write the
	//contents of each field
	li.setAttribute('data-id', doc.id)
	english.textContent = doc.data().english
	spanish.textContent = doc.data().spanish
	cross.textContent = 'Delete'

	//Include the <span> elements to the list item
	//and the item to the unordered list
	li.appendChild(english)
	li.appendChild(spanish)
	li.appendChild(cross)
	ulist.appendChild(li)

	//Deleting functionality, after the ulist.appendChild(li) line
	cross.addEventListener('click', (e) => {
		e.stopPropagation();
		let id = e.target.parentElement.getAttribute('data-id');
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
			let li = document.querySelector('[data-id=\"' + change.doc.id + '\"]')
			ulist.removeChild(li)
		}
	});
});
```

Our CSS file is empty right now.
