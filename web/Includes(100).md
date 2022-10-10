**Description**

Author: LT 'syreal' Jones

Can you get the flag? Go to this website and see what you can discover.

Hints
1) Is there more code than what the inspector initially shows?

**Inpection**

\* This problem will be done in Firefox, however any modern browser should be able to complete the task at hand. Some modifications of the steps will be necessary, but the general premise is the same.

Use inspect element on the webpage to find the page source.
In the document, it imports two other documents
```HTML
<link rel="stylesheet" href="style.css">
```
and
```HTML
<script src="script.js"></script>
```
So, why don't we check out these two other documents to see what they contain.

style.css:
```CSS
body {
  background-color: lightblue;
}

/*  picoCTF{1nclu51v17y_1of2_  */
```
script.js
```javascript
function greetings()
{
  alert("This code is in a separate file!");
}

//  f7w_2of2_4d305f36}
```
Well, there is our flag! Just combine the two parts to get picoCTF{1nclu51v17y_1of2_f7w_2of2_4d305f36}
