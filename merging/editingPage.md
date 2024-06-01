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
The initial content of the `edit.html` file is identical to that from the `theory.html` file. Copy and paste this content. **REMEMBER: change the CSS and JS file references to 'edit' in each case.** Change the text content of the _Go to Sentence Manager_ button and write _Update sentence pair_ or something alike. Also, erase the `onclick="goToSenMan()"` attribute of this button.



















