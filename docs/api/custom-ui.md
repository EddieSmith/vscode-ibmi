
# Custom UI in Code for IBM i

To make it easy for users to write custom UI, we have a CustomUI class that allows the users to define the form fields and handle the submission via a callback function. 

* Each form needs at least 1 submit field.
* You must `panel.dispose()` in the callback function.

You can find the source for this API at `src/api/CustomUI.js`.

### `CustomUI` class

* `constructor()` creates an instances
* `addField(Field)` adds a field to the CustomUI
* `loadPage(title: string): Promise<{panel: vscode.WebviewPanel, data: {...}}>` is called when you're ready to render the page.
  1. `panel: vscode.WebviewPanel` which is the panel being used to render the form.
  2. `data: {...}` which returns the form data, where the field ID's are the properties

### `Field` class

* `constructor(type: "input"|"password"|"checkbox"|"submit"|"buttons"|"hr"|"tree"|"select"|"paragraph", id: string, label: string)` to create an instance of a field.
* `field.description` (`string`) can be used to set text about the field for information about the field.
* `field.default` can be used to set the initial value of the field. If the field is a checkbox, you can use the value of `checked` to have it checked by default.
* `field.items` is used only by `tree` and `select`
  * for `tree`: `{label: string, value: string}[]`. The chosen `value` is passed back in the `loadPage` result. Note that `tree` is mostly used for a list. See `src/webviews/actions/index.js` for an example.
  * for `select`: `{selected?: boolean, value: string, description: string, text: string}[]`. Used for the user to select an item from a dropdown list.
  * For `buttons`: `{id: string, label: string}[]`. When `buttons` is used, no `id` is needed on `Field`.
* `field.readonly` can be usd with `input` only, allowing a readonly, not editable field.

Generally you should use `buttons` over `submit` - `submit` will never be updated since `buttons` is more flexable.

## Example

### Simple example

```js
const vscode = require(`vscode`);
const {CustomUI, Field} = require(`./api/CustomUI`);

context.subscriptions.push(
  vscode.commands.registerCommand(`code-for-ibmi.runMyThing`, async function () {
    let ui = new CustomUI();

    ui.addField(new Field(`input`, `name`, `Your name`));
    ui.addField(new Field(`submit`, `submitButton`, `Connect`));

    const {panel, data} = await ui.loadPage(`IBM i Login`)
    if (data) {
      if (data.name.length > 0) {
        vscode.window.showInformationMessage(`Hello ${data.name}!`);
        panel.dispose(); //Must be called to close the panel!
      } else {
        vscode.window.showErrorMessage(`Name cannot be blank.`);
      }
    } else {
      vscode.window.showInformationMessage(`Panel was closed by user.`);
    }
  })
);
```

### Many buttons

```js
field = new Field(`buttons`);
field.items = [
  {
    id: `newAction`,
    label: `New Action`,
  },
  {
    id: `duplicateAction`,
    label: `Duplicate`,
  }
];
ui.addField(field);
```