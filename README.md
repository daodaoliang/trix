# Trix
### A Rich Text Editor for Everyday Writing

**Compose beautifully formatted text in your web application.** Trix is a WYSIWYG editor for writing messages, comments, articles, and lists—the simple documents most web apps are made of. It features a sophisticated document model, support for embedded attachments, and outputs terse and consistent HTML.

Trix is an open-source project from [Basecamp](https://basecamp.com/), the creators of [Ruby on Rails](http://rubyonrails.org/). Millions of people trust their text to Basecamp, and we built Trix to give them the best possible editing experience. See Trix in action in the [all-new Basecamp 3](https://basecamp.com/3-is-coming).

### Built for the Modern Web

Trix supports all evergreen, self-updating desktop and mobile browsers.

![Browser Test Status](https://saucelabs.com/browser-matrix/basecamp-trix.svg?auth=2a3e69c3aef98784a8828ddd533ec064)

Trix is built with emerging web standards, notably [Custom Elements](http://www.w3.org/TR/custom-elements/), [Mutation Observer](https://dom.spec.whatwg.org/#mutation-observers), and [Promises](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-promise-objects). Eventually we expect all browsers to implement these standards. In the meantime, Trix includes [polyfills](https://en.wikipedia.org/wiki/Polyfill) for missing functionality.

# Getting Started

Include the bundled `trix.css` and `trix.js` files in the `<head>` of your page.

```html
<head>
  …
  <link rel="stylesheet" type="text/css" href="trix.css">
  <script type="text/javascript" src="trix.js"></script>
</head>
```

`trix.css` includes default styles for the Trix toolbar, editor, and attachments. Skip this file if you prefer to define these styles yourself.

To use your own polyfills, or to target only browsers that support all of the required standards, include `trix-core.js` instead.

## Creating an Editor

Place an empty `<trix-editor></trix-editor>` tag on the page. Trix will automatically insert a separate `<trix-toolbar>` before the editor.

Like an HTML `<textarea>`, `<trix-editor>` accepts `autofocus` and `placeholder` attributes. Unlike a `<textarea>`, `<trix-editor>` automatically expands vertically to fit its contents.

## Integrating With Forms

To submit the contents of a `<trix-editor>` with a form, first define a hidden input field in the form and assign it an `id`. Then reference that `id` in the editor’s `input` attribute.

```html
<form …>
  <input id="x" type="hidden" name="content">
  <trix-editor input="x"></trix-editor>
</form>
```

Trix will automatically update the value of the hidden input field with each change to the editor.

## Populating With Existing Content

To populate a `<trix-editor>` with existing content, include that content in the associated input element’s `value` attribute.

```html
<form …>
  <input id="x" value="Editor content goes here" type="hidden" name="content">
  <trix-editor input="x"></trix-editor>
</form>
```

Always use an associated input element to safely populate an editor. Trix won’t load any HTML content inside a `<trix-editor>…</trix-editor>` tag.

## Storing Attached Files

Trix automatically accepts files dragged or pasted into an editor and inserts them as attachments in the document. Each attachment is considered _pending_ until you store it remotely and provide Trix with a permanent URL.

To store attachments, listen for the `trix-attachment-add` event. Upload the attached files with XMLHttpRequest yourself and set the attachment’s URL attribute upon completion. See the [attachment example](…) for detailed information.

If you don’t want to accept dropped or pasted files, call `preventDefault()` on the `trix-file-accept` event, which Trix dispatches just before the `trix-attachment-add` event.

# Editing Text Programmatically

You can manipulate a Trix editor programmatically through the `Trix.Editor` interface, available on each `<trix-editor>` element through its `editor` property.

```js
var element = document.querySelector("trix-editor")
element.editor  // is a Trix.Editor instance
```

## Understanding the Document Model

The formatted content of a Trix editor is known as a _document_, and is represented as an instance of the `Trix.Document` class. To get the editor’s current document, use the `editor.getDocument` method.

```js
element.editor.getDocument()  // is a Trix.Document instance
```

You can convert a document to an unformatted JavaScript string with the `document.toString` method.

```js
var document = element.editor.getDocument()
document.toString()  // is a JavaScript string
```

### Immutability and Equality

Documents are immutable values. Each change you make in an editor replaces the previous document with a new document. Capturing a snapshot of the editor’s content is as simple as keeping a reference to its document, since that document will never change over time. (This is how Trix implements undo.)

To compare two documents for equality, use the `document.isEqualTo` method.

```js
var document = element.editor.getDocument()
document.isEqualTo(element.editor.getDocument())  // true
```

## Getting and Setting the Selection

Trix documents are structured as sequences of individually addressable characters. The index of one character in a document is called a _position_, and a start and end position together make up a _range_.

To get the editor’s current selection, use the `editor.getSelectedRange` method, which returns a two-element array containing the start and end positions.

```js
element.editor.getSelectedRange()  // [0, 0]
```

You can set the editor’s current selection by passing a range array to the `editor.setSelectedRange` method.

```js
// Select the first character in the document
element.editor.setSelectedRange([0, 1])
```

### Collapsed Selections

When the start and end positions of a range are equal, the range is said to be _collapsed_. In the editor, a collapsed selection appears as a blinking cursor rather than a highlighted span of text.

For convenience, the following calls to `setSelectedRange` are equivalent when working with collapsed selections:

```js
element.editor.setSelectedRange(1)
element.editor.setSelectedRange([1])
element.editor.setSelectedRange([1, 1])
```

### Directional Movement

To programmatically move the cursor or selection through the document, call the `editor.moveCursorInDirection` or `editor.expandSelectionInDirection` methods with a _direction_ argument. The direction can be either `"forward"` or `"backward"`.

```js
// Move the cursor backward one character
element.editor.moveCursorInDirection("backward")

// Expand the end of the selection forward by one character
element.editor.expandSelectionInDirection("forward")
```

### Converting Positions to Pixel Offsets

Sometimes you need to know the _x_ and _y_ coordinates of a character at a given position in the editor. For example, you might want to absolutely position a pop-up menu element below the editor’s cursor.

Call the `editor.getClientRectAtPosition` method with a position argument to get a DOM [`ClientRect`](…) instance representing the left and top offsets, width, and height of the character at the given position.

```js
var rect = element.editor.getClientRectAtPosition(0)
[rect.left, rect.top]  // [17, 49]
```

## Inserting and Deleting Text

The editor interface provides methods for inserting, replacing, and deleting text at the current selection.

To insert or replace text, begin by setting the selected range, then call one of the insertion methods below. Trix will first remove any selected text, then insert the new text at the start position of the selected range.

### Inserting Plain Text

To insert unformatted text into the document, call the `editor.insertString` method.

```js
// Insert “Hello” at the beginning of the document
element.editor.setSelectedRange([0, 0])
element.editor.insertString("Hello")
```

### Inserting HTML

To insert HTML into the document, call the `editor.insertHTML` method. Trix will first convert the HTML into its internal document model. During this conversion, any formatting that cannot be represented in a Trix document will be lost.

```js
// Insert a bold “Hello” at the beginning of the document
element.editor.setSelectedRange([0, 0])
element.editor.insertHTML("<strong>Hello</strong>")
```

### Inserting a File

To insert a DOM [`File`](…) object into the document, call the `editor.insertFile` method. Trix will insert a pending attachment for the file as if you had dragged and dropped it onto the editor.

```js
// Insert the selected file from the first file input element
var file = document.querySelector("input[type=file]").file
element.editor.insertFile(file)
```

### Deleting Text

If the current selection is collapsed, you can simulate deleting text before or after the cursor with the `editor.deleteInDirection` method.

```js
// “Backspace” the first character in the document
element.editor.setSelectedRange([1, 1])
element.editor.deleteInDirection("backward")

// Delete the second character in the document
element.editor.setSelectedRange([1, 1])
element.editor.deleteInDirection("forward")
```

To delete a range of text, first set the selected range, then call `editor.deleteInDirection` with either direction as the argument.

```js
// Delete the first five characters
element.editor.setSelectedRange([0, 4])
element.editor.deleteInDirection("forward")
```

## Working With Attributes and Indentation

Trix represents formatting as sets of _attributes_ applied across ranges of a document.

By default, Trix supports the inline attributes `bold`, `italic`, `href`, and `strike`, and the block-level attributes `quote`, `code`, `bullet`, and `number`.

### Applying Formatting

To apply formatting to the current selection, use the `editor.activateAttribute` method.

```js
element.editor.insertString("Hello")
element.editor.setSelectedRange([0, 5])
element.editor.activateAttribute("bold")
```

To set the `href` attribute, pass a URL as the second argument to `editor.activateAttribute`.

```js
element.editor.insertString("Trix")
element.editor.setSelectedRange([0, 4])
element.editor.activateAttribute("href", "http://trix-editor.org/")
```

### Removing Formatting

Use the `editor.deactivateAttribute` method to remove formatting from a selection.

```js
element.editor.setSelectedRange([2, 4])
element.editor.deactivateAttribute("bold")
```

### Formatting With a Collapsed Selection

If you activate or deactivate attributes when the selection is collapsed, your formatting changes will apply to the text inserted by any subsequent calls to `editor.insertString`.

```js
element.editor.activateAttribute("italic")
element.editor.insertString("This is italic")
```

### Adjusting the Indentation Level

To adjust the indentation level of block-level attributes, call the `editor.increaseIndentationLevel` and `editor.decreaseIndentationLevel` methods.

```js
element.editor.activateAttribute("quote")
element.editor.increaseIndentationLevel()
element.editor.decreaseIndentationLevel()
```

## Using Undo and Redo

Trix editors support unlimited undo and redo. Successive typing and formatting changes are consolidated together at five-second intervals; all other input changes are recorded individually in undo history.

Call the `editor.undo` and `editor.redo` methods to perform an undo or redo operation.

```js
element.editor.undo()
element.editor.redo()
```

Changes you make through the editor interface will not automatically record undo entries. You can save your own undo entries by calling the `editor.recordUndoEntry` method with a description argument.

```js
element.editor.insertString("Hello")
element.editor.recordUndoEntry("Insert Text")
```

## Loading and Saving Editor State

Serialize an editor’s state with `JSON.stringify` and restore saved state with the `editor.loadJSON` method. The serialized state includes the document and current selection, but does not include undo history.

```js
// Save editor state to local storage
localStorage["editorState"] = JSON.stringify(element.editor)

// Restore editor state from local storage
element.editor.loadJSON(JSON.parse(localStorage["editorState"]))
```

## Observing Editor Changes

The `<trix-editor>` element emits several events which you can use to observe and respond to changes in editor state.

* `trix-initialize` fires when the `<trix-editor>` element is attached to the DOM and its `editor` object is ready for use.

* `trix-change` fires whenever the editor’s contents have changed.

* `trix-selection-change` fires any time the selected range changes in the editor.

* `trix-focus` and `trix-blur` fire when the editor gains or loses focus, respectively.

* `trix-file-accept` fires when a file is dropped or inserted into the editor. You can access the DOM `File` object through the `file` property on the event. Call `preventDefault` on the event to prevent attaching the file to the document.

* `trix-attachment-add` fires after an attachment is added to the document. You can access the Trix attachment object through the `attachment` property on the event. If the `attachment` object has a `file` property, you should store this file remotely and set the attachment’s URL attribute. See the [attachment example](…) for detailed information.

* `trix-attachment-remove` fires when an attachment is removed from the document. You can access the Trix attachment object through the `attachment` property on the event. You may wish to use this event to clean up remotely stored files.


---

© 2015 Basecamp, LLC. Trix is distributed under an MIT-style license; see `LICENSE` for details.
