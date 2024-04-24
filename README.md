# Practical 17 (week 9, practical 1): sessionStorage and indexedDB
This practical includes guided tutorials for sessionStorage and indexedDB. In stage 1, you will create a counter application that uses sessionStorage to save different counter states across multiple tabs. In stage 2, you will create a similar application that stores multiple counters in indexedDB. By the end of these exercises, you should have a good sense of the differences between the two approaches to storage. The third stage will help you to make a plan for how you will store data in your assessment.

## Stage 1: sessionStorage counter
1. Create a folder for this exercise then add an HTML file and a JavaScript file. Connect the JS file to the HTML file.
2. In the HTML file, add a text input with id "name" and a "save" button. Also add an element with id "currentValue" that will display the current value of the counter. Below this element, add a button called "count".
3. You will store the counter information in as a single object in `sessionStorage`. Each object will have the following properties: `name` and `value`. Choose a key name for the stored counter (e.g. "counter").
4. Write a function called `getCounter()` that returns the object stored at the key you chose in step 3, if it exists. If there is no saved counter, the function should return an object with default values for the `name` and `value` properties. Your function should look something like this:
```
function getCounter() {
    const storedCounter = sessionStorage.getItem("counter");
    if (storedCounter !== null) {
        // Don't forget to convert saved data to JSON!
        return JSON.parse(storedCounter);
    } else {
        return {
            "name": "default counter",
            "value": "0",
        }
    }
}
```
5. At the bottom of your JavaScript file, call the `getCounter()` function and use its return value to populate the name input field and the `innerText` of the element that will display the counter value. You can see an example below (`nameInput` is a reference to the name input text field and `valueOut` is a reference to the "currentValue" element). We haven't implemented saving anything to storage yet so run your code to check that the default values are displayed in the right places.
```
const savedCounter = getCounter();
nameInput.value = savedCounter.name;
valueOut.innerText = savedCounter.value;
```
6. Write a function called `saveCounter()` that creates an object literal with two properties, `name` and `value`. These properties should be populated with the value of the name input field and the `innerText` of the "currentValue" element. Save this object literal to `sessionStorage` using the key you chose in step 3. Your function should look something like this:
```
function saveCounter() {
    sessionStorage.setItem("counter", JSON.stringify({
        "name": nameInput.value,
        "value": valueOut.innerText
    }));
}
```
7. Add an event listener to call `saveCounter()` when the save button is clicked.
8. Test your code by giving the counter a new name then clicking the save button. If you refresh the page, you should see the saved name in the input field. If you open the URL in a new tab, you should see the default name. If anything looks odd, use Chrome Developer Tools to view the tab's stored data (go to the Application tab in Developer Tools, click "Sesssion Storage" and select the URL for the current tab). You can manually delete the stored data if necessary.
9. Finally, write code to increment the value of the counter when the count button is pressed. The value should be updated in `sessionStorage` and on screen. The implementation details are left for you to figure out. Hint: the value of the counter retrieved from storage and displayed on screen will be a string. Use the `parseInt()` function to convert it to a number before changing it's value.

## Stage 2: indexedDB counter
In this exercise, you will create a slightly different version of the counter app. In the previous exercise, the user could have multiple counters by opening the app in multiple tabs, but only a singler counter was stored per tab. In the new version, you will use indexedDB to enable the user to store multiple counters.
1. Create a new folder for this exercise and add an HTML file. The HTML should include a text input (to enter a counter name) and a save button. You will also need a div with id "counters", which will display all existing counters.
2. Add a JavaScript file and connect it to the HTML file.
3. You will need to choose two names: one for the database and one for the data store, which will store the information about each counter. In the JavaScript file, create constants to store the two names e.g.:
```
const DB_NAME = "CounterDB";
const STORE_NAME = "Counters";
```
4. You will also need a global variable, `db`, that will eventually store a reference to the database. Declare it near the top of the file but don't assign it a value.
5. Copy and paste the following function into your file (don't call it yet!):
```
function openDB() {
    const request = indexedDB.open(DB_NAME);

    request.addEventListener("success", function (event) {
        
    });

    request.addEventListener("upgradeneeded", function (event) {
        
    });
}
```
The function attempts to open a database. The outlines of event handlers for two outcomes are included. The "success" event handler will run if an existing database is opened. The "upgradeneeded" event handler will run if there is no database matching the request. Notice that the event handlers for both events have a parameter called `event`. This is an object that stores the outcome of the request. Add the following line inside *both* event handlers to assign the opened/created database to the `db` variable:
```
db = event.target.result;
``` 
6. The "upgradeneeded" event handler is responsible for creating the structure of the database, including any data stores that are needed. This application is simple and only needs one store. To create the store, add the following line inside the "upgradeneeded" event handler:
```
db.createObjectStore(STORE_NAME, {keyPath: "name", autoIncrement: false});
```
This line will create a new store with the name defined in step 3. The `keyPath` property for each stored counter will be `name`, meaning that each stored counter must be an object containing a unique `name`.

7. At the bottom of the JavaScript file, outside all functions, call `openDB()`. Run the code then go to the indexedDB section of the Application tab in Chrome Developer Tools to check that the database has been created. To see the contents of the database, click the triangle next to its name.

8. Next, add the function below to your JavaScript file. Replace `<VALUE OF NAME INPUT FIELD>` with code to get the value of the name input field in your HTML. It will add to the store a new counter with the name provided by the user and a default value of zero.

```
function addCounter() {
    const transaction = db.transaction(STORE_NAME, "readwrite");
    const store = transaction.objectStore(STORE_NAME);
    
    const request = store.add({name: <VALUE OF NAME INPUT FIELD>, value: 0});

    request.addEventListener("success", function (event) {

    });
}
```
9. Call `addCounter()` when the save button is clicked. If all has gone well, you should see a new counter in the data store each time you change the counter name and click the save button. Repeatedly clicking the save button without changing the name will not overwrite or change the existing data.

**The remaining steps of this stage are quite challenging! You will need to make use of the lecture slides and example code from lecture. It also assumes you remember how to add and update HTML elements using JavaScript.**

10. Add the function below to your JavaScript file. This function will retrieve the counter object with a name matching the `name` parameter. Fill in the missing code to display the counter's name and value, and a button labeled "Count". The new button will need a click event handler, but don't worry about implementing it just yet.
```
function showCounter(name) {
    const transaction = db.transaction(STORE_NAME, "readonly");
    const store = transaction.objectStore(STORE_NAME);

    const request = store.get(name);

    request.addEventListener("success", function (event) {
        const data = event.target.result;
        if (data) {
            // YOUR CODE HERE
            // Add HTML elements to the element with id "counters":
            // Show the name, its current value, and a count button
        }
    });
}
```
11. Call `showCounter()` from the `request` "success" handler inside `addCounter()` and pass it the name of the new counter. Now, if you add a new counter with a unique name, you should see it appear on the page.

12. Write a new function called `getAllCounters()`. It should use the cursor approach to get each existing counter and draw it on the page using the `showCounter()` function from step 10. Use the slides (beginning with slide 43) and the complete indexedDB example code from lecture to help you implement the cursor. 

13. Call `getAllCounters()` when the database is successfully opened. 

14. Finally, implement the click event handler for each count button created in the `showCounter()` function. To do this, you will need to use the store method [`put()`](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB#updating_an_entry_in_the_database). 

You should now be able to add new counters and update the value of each one independently. If you open the page in a different tab, you should see the same counters.

## Stage 3: Choose a data storage approach for assessment 2
By now, you should have a good idea of what you're building for assessment 2, and the type of data you want to store. With this practical, you have tried out all three data storage approaches that are available to you. Now is a good time to figure out which approach will work for your assessment. 

- `sessionStorage` - best for storing "work-in-progress" data such as partially completed forms or blog posts. Remember that the data is tab-specific and will be removed once the tab is closed. `sessionStorage` is not suitable for data that should be remembered across multiple sessions.
- `localStorage` - best for storing simple data that should be remembered across multiple sessions e.g. user preferences. `localStorage` can only store data that can be "stringified", such as JSON objects, but not images / files.
- `indexedDB` - suitable for the same purposes as `localStorage` but has the added benefit of better search, more storage space, and the ability to store more complex data such as images.

In most cases, it is likely that `localStorage` will suffice. If in doubt, talk it through with the module staff.