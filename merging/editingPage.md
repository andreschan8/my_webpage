# Implementation of the Editing Page
Starting from the files of version 1.0 from the previous md file, we'd like to add a third page that allows us to edit a given sentence pair. The user should be able to click a button in the Sentence Manager page that redirects them to a page essentially identical to that of the Structure Guesser, with the difference that the _Add sentence_ button will now be an _Update sentence_ button. The features of the Structure Guesser page should allow the user to perform a quick and efficient editing of the sentence pair, since most of the issues encountered when a new sentence pair is added are related to its performance in the Structure Guesser.
## (1) Redirecting to the Editing Page
  Start by adding the _Edit sentence pair_ button to the Sentence Manager page. Include the following snippets in the `renderPairs()` function in the `data3.js` file: `let editButton   = document.createElement('button');`, `editButton.textContent   = 'Edit sentence pair';`, `div2.appendChild(editButton);`. Include also the following block:
```javascript
editButton.addEventListener('click', function() {
		goToEdit(doc);
});
```
Here we have called a function named `goToEdit()`, and is analogous to the `goToStructGuess()` function used to send a chosen sentence pair to the Structure Guesser page. The definition of `goToEdit()` is as follows.
```javascript
function goToEdit(doc) {
    db.collection('userLogs').add({
        english: doc.data().english,
        spanish: doc.data().spanish
    })
    .then(function() {
        window.location.href = 'edit.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};
```
As we know, this function writes the corresponding sentence pair to the `userLogs` collection and redirects the user to a new page. In this case the page is `edit.html`. 
## (2) Sentence Pair Editing
The initial content of the `edit.html` file is identical to that from the `theory.html` file. Copy and paste this content. **REMEMBER: change the CSS and JS file references to 'edit' in each case.** Change the text content of the _Go to Sentence Manager_ button and write _Update sentence pair_ or something alike.
```javascript
<button type="button" onclick="updateAndGoToSenMan()">Update sentence pair</button><br>
```
**WE NOW MAKE AN IMPORTANT CHANGE. WE ADD A NEW FIELD TO THE USER LOGS COLLECTION, WHICH IS `doc_id`, AND ITS INCLUSION DOES NOT SEEM TO AFFECT THE FUNCTIONALITY OF THE OTHER PAGES.**
The important change just mentioned makes it possible to keep track of the document that we want to modify. Add `doc_id: doc.id` to the `goToEdit()` function in the `data3.js` file.
```javascript
function goToEdit(doc) {
    db.collection('userLogs').add({
	doc_id: doc.id,
        english: doc.data().english,
        spanish: doc.data().spanish
    })
    .then(function() {
        window.location.href = 'edit.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};
```
Now we focus on the `edit.js` file. Create a variable named `editID` immediately after `dummyID`, that is, in the second line of the file, and set it equal to an empty string. Inside the function of the `window.onload` variable add the snippet `editID = doc.data().doc_id;` just before `enSen.textContent = doc.data().english;`. This stores the ID of the document we want to modify inside the `editID` variable. 

Change the `goToSenMan()` function to `updateAndGoToSenMan()`. This function will be analogous to the others `goTo` functions we've defined. The function is as follows.
```javascript
function updateAndGoToSenMan(doc) {
    db.collection('sentencePairs').doc(doc).update({
        english: document.querySelector('#englishText').value,
        spanish: document.querySelector('#spanishText').value
    })
    .then(function() {
        window.location.href = 'data3.html';
    })
    .catch(function(error) {
        console.error("Error writing to Firestore: ", error);
    });
};
```
**_Important: First we tried using `textContent` instead of `value` and without the `.then()` and `catch()` methods, which didn't work. Then we used `value`, which resulted in a faulty functionality, and finally we implemented the previously mentioned methods, which solved the problem entirely._**



















