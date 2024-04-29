# Further improvements 1
We start from the checkpoint files of v1.0. The first improvement we want to implement is to detect the punctuation marks of the English sentence and add placeholders in their corresponding textboxes.
## (1) Detection of punctuation marks
We have to modify the `theory.js` file. Insert the following dictionary just before the definition of the `printStructure()` function.
```javascript
//We create this variable here since it is
//used in the next function.
var punctChars = {
	".":"dot",",":"comma","-":"hyphen",
	"?":"question","!":"exclamation",
	":":"colon",";":"semicolon",
	"(":"lpar",")":"rpar","[":"lsqb",
	"]":"rsqb","\"":"quot", "'":"apos"
};
```
This dictionary summarizes the punctuation marks we'll be able to detect and their corresponding placeholders. Next, we need to define a function that checks if a word has any of these punctuation marks and includes its placeholder. Place the following function between  the `input.autocapitalize = "none"` and `document.getElementById("englishStructure").appendChild(input)` lines:
```javascript
// Check for punctuation in the word
const punctuation = words[i].match(/[.,-?!:;\(\)\[\]\"\'']/g);
if (punctuation) {
input.placeholder = punctuation.map(p => punctChars[p]).join(', ');
}
```
**TODO: **EXPLAIN THIS FUNCTION (Right now we just now that it works!)****

## (2) Word reveal command
Now we'd like to be able to reveal a word when the user types in a command in a textbox, which will show the corresponding word. We define this command as the string `'$r'`. 

In the `theory.js` file, inside the `textboxManager()` function, locate the event listener with an `'input'` event. This event handles the content of the textboxes at all times, from what I understand. Insert the following if statement at the very beginning of the event listener.
```javascript
if (textbox.value === '$r') {
  textbox.value = words[index];
};
```
This if statement reveals the word of the corresponding textbox, and the focus is changed to the next textbox automatically due to the fact that this statement is placed before the if statement that adds the `right` class to the textboxes.





















