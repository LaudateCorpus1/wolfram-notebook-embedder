---
id: NotebookAPI
title: Notebook API
---

This is the documentation of [methods](#methods) on the object returned by [`embed`](./LibraryInterface.md) to control a notebook from JavaScript code. Each method takes a JavaScript object with *parameters* and returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) resolving to a *response* object. Parameters with a default value (specified as "= `...`") are optional. In case of an error, the returned promise is rejected with a standard [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) object with a `message` property; the potential error messages are listed for each command.

The API also exposes some [events](#events) that can be subscribed to using the API method `addEventListener(eventName, callback)`. The `callback` will receive a JavaScript object with the specified *fields* as its single argument. Event listeners can be removed with `removeEventListener(eventName, callback)`. Some events are marked as *singular*, which means that they will usually fire exactly once, and an event listener added after they have already fired will still be executed (right away).


## Methods

### Cell content

#### getCellContent

Gets the textual content of a cell.

+ Parameters

    + `cellId` (`string`) — ID of the cell to read.

+ Response

    + `content` (`string`) — The textual content of the cell.

+ Errors

    + `"CellNotFound"` — The cell ID given by `cellId` was not found.

### Structure

#### getElements

Retrieves the (top-level) elements in the notebook or a cell group.

+ Parameters

    + `groupId` (`?string`) — ID of the cell group. If the parameter is omitted or falsy (e.g. `null` or `""`), the top-level elements in the notebook are returned.

+ Response

    + `elements` (`Array.<{type: "cell"|"group", id: string}>`) — List of elements in the cell group, in the order they appear in the notebook.
    + `isClosed` (`boolean`) — Whether the cell group is closed.
    + `visibleElementIndex` (`?number`) — Index (0-based) of the visible element in case the cell group is closed. `null` if the group is not closed.

+ Errors

    + `"GroupNotFound"` — The group specified by `groupId` was not found.

Example request:

```json
{
    "api": "notebook",
    "version": 1,
    "rid": "1",
    "command": "getElements",
    "groupId": "1"
}
```

Example response:

```json
{
    "rid": "1",
    "success": true,
    "elements": [{"type": "cell", "id": "2"}],
    "isClosed": false,
    "visibleElementIndex": null
}
```

#### getElementParent

Retrieves the parent cell group of a given cell or cell group.

+ Parameters

    + `id` (`string`) — ID of the cell or cell group.

+ Response

    + `groupId` (`?string`) — ID of the group the given element is contained in. `null` if the element is in no group (i.e. it is on the top level of a notebook).

+ Errors

    + `"ElementNotFound"` — The element specified by `id` was not found.

#### getCells

Retrieves all (potentially nested) cells in the notebook or a cell group.

+ Parameters

    + `groupId` (`?string`) — ID of the cell group. If the parameter is omitted or falsy (e.g. `null` or `""`), the cells in the notebook are returned.

+ Response

    + `cells` (`Array.<{type: "cell", id: string}>`) — List of cells in the order they appear in the notebook.

+ Errors

    + `"GroupNotFound"` — The group specified by `groupId` was not found.

#### openGroup

Opens a cell group. If a group is already open, this has no effect.

+ Parameters

    + `groupId` (`string`) — ID of the cell group.

+ Errors

    + `"GroupNotFound"` — The group specified by `groupId` was not found.

#### closeGroup

Closes a cell group, only displaying one of its elements.

+ Parameters

    + `groupId` (`string`) — ID of the cell group.
    + `visibleElementIndex` (`?number`) — Index (0-based) of the remaining visible element. If not specified, the first element is chosen ("forward-close"); unless if there is a whole-cell group opener cell in the group, in which case that cell is chosen. If less than 0, the first element is chosen. If greater than or equal to the total number of elements in the group, the last element is chosen ("reverse-close").

+ Errors

    + `"GroupNotFound"` — The group specified by `groupId` was not found.

### Notebook options

#### setOptions

Sets outer options to be applied to the notebook. Currently, only the `"Magnification"` option is supported.

+ Parameters

    + `options` (`Object.<string, string|number>`) — Dictionary of option names and their respective values.

Example request:

```json
{
    "api": "notebook",
    "version": 1,
    "rid": "1",
    "command": "setOptions",
    "options": {
        "Magnification": 2
    }
}
```

Example response:

```json
{
    "rid": "1",
    "success": true
}
```

#### getOption

Gets the value of an outer option. Currently, only the `"Magnification"` option (which resolves to a `number`) is supported.

+ Parameters

    + `option` (`string`) — Option name.

+ Response

    + `option` (`string`) — Option name.
    + `value` (`?string|number`) — Option value. Might be `null` if no explicit value is set.

Example request:

```json
{
    "api": "notebook",
    "version": 1,
    "rid": "1",
    "command": "getOption",
    "option": "Magnification"
}
```

Example response:

```json
{
    "rid": "1",
    "success": true,
    "option": "Magnification",
    "value": 2
}
```

### Selection

#### getSelection

Gets information about the current selection.

+ Response

    + `elements` (`Array.<{type: "cell"|"group", id: string}>`) — List of elements (cells or cell groups) selected via their cell brackets. Empty array if no cell brackets are selected.
    + `separator` (`?{cellBefore: ?{cellId: string}, cellAfter: ?{cellId: string}}`) — If the selection is between cells, this gives the cell before and the cell after the separator. Either of them or `separator` itself can be `null`.
    + `inCell` (`?{cellId: string}`) — The cell the selection is inside (i.e. on the box level), if any. This is particularly the case when editing a cell. If the selection is on the cell (bracket) level, `inCell` is `null`.
    + `cursorPosition` (`?{left: number, top: number}`) — Cursor position, if a cell is edited. The position is relative to the document offset. This takes into account the scroll position.

Example request:

```json
{
    "api": "notebook",
    "version": 1,
    "rid": "1",
    "command": "getSelection"
}
```

Example response:

```json
{
    "rid": "1",
    "success": true,
    "cells": [],
    "inCell": {"cellId": "1"},
    "cursorPosition": {"left": 80, "top": 300}
}
```

#### selectElements

Selects elements (cell brackets).

+ Parameters

    + `elements` (`Array.<{id: string}>`) — List of IDs of elements to select. Elements that cannot be found are ignored.

+ Response

    + `elements` (`Array.<{type: "cell"|"group", id: string}>`) — List of elements that have been found and are selected now.


#### selectSeparatorBeforeElement

Moves the selection to the separator before a cell or cell group.

+ Parameters

    + `id` (`?string`) — ID of the element before which the separator should be. If `null`, the selection is moved to the end of the notebook.

+ Errors

    + `"ElementNotFound"`

#### selectSeparatorAfterElement

Moves the selection to the separator after a cell or cell group.

+ Parameters

    + `id` (`?string`) — ID of the element after which the separator should be. If `null`, the selection is moved to the beginning of the notebook.

+ Errors

    + `"ElementNotFound"`

### Dimensions

#### getDimensions

Returns the dimensions of the notebook content.

+ Response

    + `width` (`number`) — The width of the notebook content (in pixels). Note that a notebook will always fill any available (maximum) width. If there is a horizontal scrollbar, the reported width may be larger than the maximum width available to the notebook. Providing the reported width as horizontal space to the notebook usually ensures that there is no horizontal scrollbar necessary.
    + `height` (`number`) — The height of the notebook content (in pixels). This does not include any unused vertical space that would be available to the notebook. It does include the height of the horizontal scrollbar, in case there is one. Providing the reported height as vertical space to the notebook usually ensures that there is no vertical scrollbar necessary.

### Scrolling

#### getScrollPosition

Returns the current scroll position in the notebook.

+ Response

    + `left` (`number`) — Number of pixels scrolled in horizontal direction (increases when scrolling to the right).
    + `top` (`number`) — Number of pixels scrolled in vertical direction (increases when scrolling down).

#### setScrollPosition

Scrolls to the specified position in the notebook.

+ Parameters

    + `left` (`number`) — Number of pixels to scroll to in horizontal direction (increases when scrolling to the right).
    + `top` (`number`) — Number of pixels to scroll to in vertical direction (increases when scrolling down).

### Evaluation

#### evaluateExpression

Evaluates a Wolfram Language expression.

The API responds with the resulting expression when the evaluation is finished.

+ Parameters

    + `expression` (`string` or `exprjson`) — Wolfram Language expression to evaluate.
    + `originatingCellId` = `null` (optional, `?string`) — ID of the cell the evaluation is supposed to be originating from (for the purpose of `EvaluationCell[]`).

+ Response

    + `result` (`exprjson`) — The result of the evaluation in JSON expression representation (see below).

+ Errors

    + `"EvaluationError"` — There was some error during the evaluation, e.g. the kernel was not reachable.
    + `"InsufficientPermissions"`

Example request:

    {
        "api": "notebook",
        "version": 1,
        "rid": "1",
        "command": "evaluateExpression"
        "expression": "f[x, 1 + 2]"
    }

Example response:

    {
        "rid": "1",
        "success": true,
        "result": {"head": "f", "args": ["x", 3]}
    }
    
#### evaluateInDynamicModule

Evaluates a Wolfram Language expression inside a [DynamicModule](https://reference.wolfram.com/language/ref/DynamicModule.html), localizing variables as necessary.

+ Parameters

    + `cellId` (`string`) — ID of the cell that contains the `DynamicModuleBox`.
    + `boxId` (`?string`) — `BoxID` value of the `DynamicModuleBox` to search for. If not specified, the first `DynamicModuleBox` in a breadth-first search is chosen.
    + `expression` (`string` or `exprjson`) — Wolfram Language expression to evaluate.

+ Response

    + `result` (`exprjson`) — The result of the evaluation in JSON expression representation (see below).

+ Errors

    + `"EvaluationError"` — There was some error during the evaluation, e.g. the kernel was not reachable.
    + `"InsufficientPermissions"`

#### clickButton

Simulates clicking of a button, i.e. evaluates its `ButtonFunction`.

+ Parameters

    + `cellId` (`string`) — ID of the cell that contains the `ButtonBox`.
    + `boxId` (`?string`) — `BoxID` value of the `ButtonBox` to search for. If not specified, the first `ButtonBox ` in a breadth-first search is chosen.

+ Errors

    + `"NoBox"` — The specified `ButtonBox` was not found.

### DynamicModule variables

#### getDynamicModuleVariable

Retrieves the value of a [DynamicModule](https://reference.wolfram.com/language/ref/DynamicModule.html) variable.

+ Parameters

    + `cellId` (`string`) — ID of the cell that contains the `DynamicModuleBox`.
    + `boxId` (`?string`) — `BoxID` value of the `DynamicModuleBox` to search for. If not specified, the first `DynamicModuleBox` in a breadth-first search is chosen. 
    + `name` (`string`) — Name of the `DynamicModule` variable to retrieve.
    + `useExactName` = `false` (`?boolean`) — Whether to require an exact name match. If set to `false` (the default), the provided name does not have to contain prefixes (such as ```$CellContext` ```) and suffixes (such as `$$`) that are typically added behind the scenes by `DynamicModule`.

+ Response

    + `value` (`exprjson`) — The value of the variable in JSON expression representation (see [below](#expressionjson)).

+ Errors

    + `"CellNotFound"` — The specified cell does not exist.
    + `"NoDynamicModule"` — There is no `DynamicModuleBox` in the specified cell (with the given `BoxID`, if `boxId` is specified).
    + `"UnknownVariableName"` — The `DynamicModuleBox` does not contain the specified variable.

#### setDynamicModuleVariable

Sets the value of a [DynamicModule](https://reference.wolfram.com/language/ref/DynamicModule.html) variable.

+ Parameters

    + `cellId` (`string`) — ID of the cell that contains the `DynamicModuleBox`.
    + `boxId` (`?string`) — `BoxID` value of the `DynamicModuleBox` to search for. If not specified, the first `DynamicModuleBox` in a breadth-first search is chosen. 
    + `name` (`string`) — Name of the DynamicModule variable to change.
    + `useExactName` = `false` (`?boolean`) — Whether to require an exact name match. If set to `false` (the default), the provided name does not have to contain prefixes (such as ```$CellContext` ```) and suffixes (such as `$$`) that are typically added behind the scenes by `DynamicModule`.
    + `value` (`exprjson`) — The new value of the variable in JSON expression representation (see [below](#expressionjson)).

+ Errors

    + `"CellNotFound"` — The specified cell does not exist.
    + `"NoDynamicModule"` — There is no `DynamicModuleBox` in the specified cell (with the given `BoxID`, if `boxId` is specified).
    + `"UnknownVariableName"` — The `DynamicModuleBox` does not contain the specified variable.

Example request (setting a variable `x` to `N[Pi, 30]`):

    {
        "api": "notebook",
        "version": 1,
        "rid": "1",
        "command": "setDynamicModuleVariable"
        "cellId": "42",
        "name": "x",
        "value": ["N", "Pi", 30]
    }

### Cell rendering

#### getCellRenderingMethod

Returns the rendering method (mode) used by a given cell.

There are currently four different rendering methods, as shown here.

| Method   | Meaning |
|----------|---------|
| `boxes`  | live-rendered |
| `static` | rendered as static HTML, from the HTML cache |
| `image`  | rendered as a rasterized image |
| `none`   | rendered as a general placeholer |

+ Parameters

    + `cellId` (`string`) — The ID of the cell.

+ Response

    + `method` (`string`) — The method used to render the given cell.

+ Errors

    + `"CellNotFound"` — The cell ID given by `cellId` was not found.

## Events

### Initial render progress

See [Notebook Loading Phases](./NotebookLoadingPhases.md) for more information.

#### first-paint-done (singular)

Fired when either static HTML is shown for the notebook (see [Server-Side Rendering](./ServerSideRendering.md)) or the notebook starts rendering individual cells.

+ Fields

    + `showingStaticHTML` (`boolean`) — Whether the notebook is showing a piece of static HTML.

#### initial-render-progress

Fired when progress is made during the initial render phase.

+ Fields

    + `cellsRendered` (`number`) — Number of cells that have already been "live-rendered".
    + `cellsTotal` (`number`) — Total number of cells that are visible in the notebook.

#### initial-render-done (singular)

Fired when the initial render phase is done, i.e. all visible cells have been live-rendered.

### Selection

#### selection-change

Fired when the notebook selection changes.

+ Fields

    + `elements` (`Array.<{type: "cell"|"group", id: string}>`) — List of elements (cells or cell groups) selected via their cell brackets.
    + `separator` (`?{cellBefore: ?{cellId: string}, cellAfter: ?{cellId: string}}`) — If the selection is between cells, this gives the cell before and the cell after the separator.
    + `inCell` (`?{cellId: string}`) — The cell the selection is inside (i.e. on the box level), if any. This is particularly the case when editing a cell.

### Evaluation

#### evaluation-start

Fired when an evaluation starts.

Currently, only one kernel evaluation can happen at any time. However, this might change in the future. Then this API might change as well.

+ Fields

    + `isCellEvaluation` (`boolean`) — `true` if the evaluation is a whole-cell evaluation (e.g. from pressing Shift+Enter). `false` if the evaluation is triggered by a `Dynamic` or some other dynamic control (e.g. a `Button`).

#### evaluation-stop

Fired when an evaluation ends.

### Dimensions

#### resize

Fired when the notebook dimensions change. See [`getDimensions`](#getdimensions) for details.

+ Fields

    + `width` (`number`) — Width of the notebook content (in pixels).
    + `height` (`number`) — Height of the notebook content (in pixels).

### Scrolling

#### scroll-position-change

Fired when the scroll position changes.

+ Fields (same as `getScrollPosition`)

    + `left` (`number`) — Number of pixels scrolled in horizontal direction (increases when scrolling to the right).
    + `top` (`number`) — Number of pixels scrolled in vertical direction (increases when scrolling down).


## ExpressionJSON

Some API methods accept or return WL expressions in the [ExpressionJSON](https://reference.wolfram.com/language/ref/format/ExpressionJSON.html) format:

| WL | WL example | JSON | JSON example|
|----|------------|------|-------------|
| Symbol | `foo` | string | `"foo"` |
| String | `"text"` | string with enclosing single or double quotes | `"'text'"` or `"\"text\""` |
| Machine-size Integer (up to 53 bits) | `342` | number | `342` |
| Double-precision Real (up to 64 bits) | `2.5`, `1.073741824*^9` | number | `2.5`, `1.073741824e9` |
| Arbitrary-length Integer | `18014398509481984` | string | `"18014398509481984"` |
| Arbitrary-precision Real | ```3.5074`15.65*^451``` | string | ```"3.5074`15.65*^451"``` |
| Booleans | `True`, `False` | boolean | `true`, `false` |
| Null | `Null` | null | `null` |
| Expression | `f[x, y]` | array with head at index 0, followed by parts | `["f", "x", "y"]` |

Note that there are several ways to represent the same expression, e.g. `True` can be represented as `true` or `"True"` in ExpressionJSON and a number such as `123` can be represented as the JSON number `123` or as the JSON string `"123"`. When the Notebook API returns expression data, it usually serializes it in the shortest form. However, that is not guaranteed, and readers of the format should make sure to accept all possible forms.

As a courtesy, API methods also accept single strings containing the WL expression in [InputForm](https://reference.wolfram.com/language/ref/InputForm.html) (e.g. the string `"1+2"`) or [FullForm](https://reference.wolfram.com/language/ref/FullForm.html) (e.g. `"Plus[1,2]"`; `FullForm` is a subset of `InputForm`). A WL parser implemented in JavaScript is used to parse the given expression. However, there is some performance overhead to that and it has some limitations (e.g. not all operators might be supported), so it is generally recommended to use proper ExpressionJSON instead.

Note that there is no ambiguity between ExpressionJSON and expressions given in InputForm, since all strings in ExpressionJSON contain the InputForm of the corresponding WL expression.
