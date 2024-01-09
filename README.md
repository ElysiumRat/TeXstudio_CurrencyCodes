# TeXstudio_CurrencyCodes

A macro for TeXstudio that allows the user to look up a country, currency, or ISO 4217 code, and will tell them what all corresponding information (current as of Jan 1, 2023) for that country's currency is, giving them the option to insert the code where the cursor is.

The search functionality works by first checking against selected text, or, if it can't find anything, by allowing the user to input the name of a country, currency, or an ISO code before checking. While most countries have complex names in the ISO database, the macro works with wildcards to ensure that common names (or even just snippets thereof) can be used in the search. Various common alternate English names for countries, as well as abbreviations, are also listed, though this may be incomplete.

# Basic Information

This section provides a basic description of how the macro works.

## Running the Macro

When you run a macro, if nothing has been selected, you will see a popup asking for input. Simply type in what you wish to search for (the search is not case-sensitive, so capitalisation does not matter), click “OK”, and the macro will find it for you. Otherwise, if something has been selected, the macro will search for that selection.

The results will come up in a popup box. If you click “Yes”, the macro will end. If you click “No”, the macro will keep searching through its database until you indicate you have found what you were looking for, or it reaches the end with no results. If the macro reaches the end of the database without finding anything, a popup stating that nothing was found will be displayed.

## Writing the ISO Currency Code

This macro specifically allows for inserting of the ISO 4217 currency code into the text, as a need to place this into the document is the most likely reason the user would be using this macro in the first place. So, when an item has been found by this macro, the popup box will ask if you'd lie to insert the code. “Yes” will end the macro by inserting the code, and “No” will perform the same as described above, searching for the next matching entry.

## Answers to Common Concerns

When writing the currency code to the document, the macro does not erase the selected text, but rather inserts the text afterwards. This is by design.

# Technical Information

This section provides a technical overview of the code of the macro and its functionality. This is not necessary for the average user to read, but rather gives an overview of how the macro works. The next section deals with specific code in the macros and explains how they work in detail.

## Arrays

To create an array, you need to call it this way:

`var arrayname = ["Input1", "Input2"]`

All items in the array need to be placed within the square brackets. Individual items within the array need to be placed between double quotation marks and separated with commas. Unfortunately, TeXstudio only seems to allow for one-dimensional arrays, hence these macros all use multiple arrays with the matching data at the same index. The first item in an array is always at the index point 0.

To call a value from an array, you do so with the following code:

`examplevariable = arrayname[i]`

So, using the above example, `arrayname[0]` would return “Input1” and store it in `examplevariable`.

## The UniversalInputDialog

The `UniversalInputDialog` is used to allow the user to input information into the program for the macro to use. However, the structure of it within the code is not exactly intuitive, so a brief description of how it works will be provided here.

Firstly, you need to create it within the code in the following way:

`dlg = new UniversalInputDialog()`

Thereafter, you need to add input fields to it in the following manner:

`dlg.add(DEFAULT, MESSAGE, TYPE)`

In this code, `DEFAULT` is what is within the textbox that the macro uses, and is sometimes needed to be within quotation marks. `MESSAGE`, within quotation marks, is the label that the textbox has, telling the user what to input. Finally, `TYPE` (again, in quotation marks) is the type of input box. These macros only use textbox, which is structured as follows:

`dlg.add("","Please input:", "textbox")`

In this case, `DEFAULT` needs to be in inverted commas. The other possible types will be discussed now. First, checkbox:

`dlg.add(false, "Please check:", "checkbox")`

Whereby `false` sets the checkbox as unchecked, and `true` would set it as checked.

Second, `selectbox`:

`dlg.add(["1","2"], "Please select:", "selectbox")`

Whereby the values are contained within an array, as was discussed above.

Finally, there is `numberbox`:

`dlg.add(0, "Please type:", "numberbox")`

This is, naturally, used when only a number is required.

# Specific Cases in the Code

Finally, this section gives a detailed description of how specific aspects of the code achieve specific outcomes.

The first thing that the macros attempt to do is grab the text that the user has selected. First, the macros determine whether there is a selection, and calls a `UniversalInputDialog` box if there is none, allowing the user to input their own string, which is stored in the search variable: 

```
if (cursor.hasSelection() == false)
{
	dlg = new UniversalInputDialog()
	dlg.add("","Please input [insert what's being looked up]:","textbox")
	dlg.exec()
	search = dlg.get("textbox")
}
```

However, if the cursor has a selection, the macro simply copies what was selected to the search variable:

```
else
{
	search = cursor.selectedText()
}
```

Next, the macro checks if search is still empty. If so, it changes `noinput` to true, and this will be used later. Otherwise, if it has text, then it changes the text to uppercase. The structure here is to ensure that an error does not occur if search is empty, and the need to uppercase the entire search phrase will be explained later.

```
if (search == "")
{
	noinput = true
}
else
{
	search = search.toUpperCase()
}
```

Next, the macros run through the arrays and compare `search` to the contents thereof.

## Looping through the Arrays

In short, the macro tests the variables in the same position in each array in every loop. This is so that the macro works efficiently, not needing to run through each array in a separate loop. The loop simply runs this way:

`while (i < UPPER)`

with `UPPER` referring to the highest value of the arrays. The tests are done in the following fashion:

```
temp1 = array1[i].toUpperCase()
match1 = temp1.includes(search)
```

What this code does is saves the value at position i of the array into a temporary string, which is made uppercase so that it can be compared to search. This is done by checking whether the temporary string from the array includes the contents of search, as the former will generally contain more information than the latter.

Thereafter, it performs the following test:

`if (match1 > 0 && alreadyfound == false)`

This test should be self-explanatory: so long as the match contains the search item at least once, and the item has not already been found, then the code contained within the if statement runs. The code within is as follows:

```
alreadyfound = true
num = i
```

`num` is used to call values from the arrays for the message box, just so that the macro does not try to use `i` for that. It’s also necessary in the Currency Code macro for reasons that will be explained below.

## The Message Box

Naturally, the message box is the most critical aspect of the macros, as the user is running them to attempt to find something out more quickly and easily than via other methods.

Firstly, the message box only needs to be run if something has been inputted and if something has been found:

`if (alreadyfound == true && noinput == false)`

Thereafter, the macro runs a confirm dialogue and stores the answer in the variable answer.

`answer = confirm('Example 1: ' + ex1[num] + '\nExample 2: ' + ex2[num] + 'Question?')`

Once the user clicks “Yes” (which returns a value of 1) or “No” (which returns 0), the following checks will be run:

```
if (answer == 0)
{
	alreadyfound = false
}
if (answer == 1)
{
	i = UPPER
}
```

In other words, if the user indicates that this was not what they were looking for, the macro flips `alreadyfound` back to false so that it can run again. Otherwise, if it is, the macro sets `i` to the uppermost value of the arrays, so that the loop is closed without running further.

## Writing to the Document

The point of this macro is to write the ISO 4217 currency code to the document. This is achieved in the following manner:

```
if (cursor.hasSelection() == true)
{
	insertion = cursor.selectedText() + " " + currencycode[num] + " "
}
else
{
	insertion = currencycode[num] + " "
}
```

In brief, the idea is to insert the code, not overwrite the selected text (if any text is selected). Thus, if the cursor has a selection, then the macro copies the selection to the `insertion` variable, while also adding a space after it, followed by the required code from the array containing it, followed by another space. Otherwise, it simply sets `insertion` as the code from the array, followed by a space.

Thereafter, it writes the contents of insertion to the document:

`editor.write(insertion)`

The code is thus added to the document.
