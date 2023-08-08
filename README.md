# Google-Form-Edit-Response-Link
Creating this repo for easy understanding of Google Form Edit Response


###  Step 1. Prepare your form and spreadsheet

   * You have a spreadsheet connected with the form as a response destination.
     
     ![image](https://github.com/Revanth-13/Google-Form-Edit-Response-Link/assets/123372740/55ef385d-80ca-4b4d-a4b0-746edf925384)

###  Step 2. Set up the magic Apps Script in Your Spreadsheet

   * Copy the apps script in the Source code section below
   * Open up the spreadsheet with which the form is associated.
   * Click Extensions in the menu bar, followed by the App Script.
   * In the popped Script Editor tab
       * Give the script editor a name, say Add Form Response Edit URLs
       * Paste the code.
       * Click save.
         
![image](https://github.com/RamaNaidu89/Google-Form-Edit-Response-Link/assets/128470511/7ac368cd-7e1a-42dc-bcd2-bbad7548e3df)

   * Now go back to the Spreadsheet tab. **Refresh the page.**
   * You will see new Menu Forms > Add Form Response Edit URLs by the end of the menu bar.

![image](https://github.com/RamaNaidu89/Google-Form-Edit-Response-Link/assets/128470511/ba9ac309-8e62-4e67-8b70-00ee9c846392)

###  Step 3. Automation

   * Click the Forms > Add Form Edit Response URLs menu.
   * Wait for the script to run. If the Authorization window popups, don't panic, just go through it. The Apps Script is safe. Also, if nothing happens, you may need to click Forms > Add Form Edit Response URLs menu again.
   * You will see the script adds a new column Form Response Edit URL at the far right of the sheet (If you don't see it, scroll right). It then adds one-by-one the URLs to the responses. Amazing, huh?
If your sheet is large (thousands of responses) and you see an error Exceeded maximum execution, run the script again by clicking the menu Forms > Add Form Edit Response URLs. 

![image](https://github.com/RamaNaidu89/Google-Form-Edit-Response-Link/assets/128470511/f733cceb-d5ee-4435-9413-1bc089d3753c)

###  Step 4. Edit responses in the spreadsheet!

   *  Now you have a response edit URL for every submission, you are free to click those links and make changes to them.
   *  The changes will be reflected soon in the spreadsheet.
     
###  Reference Code

```
/**
 * @license MIT
 * 
 * © 2019-2020 xfanatical.com. All Rights Reserved.
 *
 * @since 1.1.2 interface fix
 * @since 1.1.1 Optimize performance (continued)
 * @since 1.1.0 Optimize performance
 * @since 1.0.0 Add all edit response urls and update new urls for new submissions
 */
function registerNewEditResponseURLTrigger() {
  // check if an existing trigger is set
  var existingTriggerId = PropertiesService.getUserProperties().getProperty('onFormSubmitTriggerID')
  if (existingTriggerId) {
    var foundExistingTrigger = false
    ScriptApp.getProjectTriggers().forEach(function (trigger) {
      if (trigger.getUniqueId() === existingTriggerId) {
        foundExistingTrigger = true
      }
    })
    if (foundExistingTrigger) {
      return
    }
  }

  var trigger = ScriptApp.newTrigger('onFormSubmitEvent')
    .forSpreadsheet(SpreadsheetApp.getActive())
    .onFormSubmit()
    .create()

  PropertiesService.getUserProperties().setProperty('onFormSubmitTriggerID', trigger.getUniqueId())
}

function getTimestampColumn(sheet) {
  for (var i = 1; i <= sheet.getLastColumn(); i += 1) {
    if (sheet.getRange(1, i).getValue() === 'Timestamp') {
      return i
    }
  }
  return 1
}

function getFormResponseEditUrlColumn(sheet) {
  var form = FormApp.openByUrl(sheet.getFormUrl())
  for (var i = 1; i <= sheet.getLastColumn(); i += 1) {
    if (sheet.getRange(1, i).getValue() === 'Form Response Edit URL') {
      return i
    }
  }
  // get the last column at which the url can be placed.
  return Math.max(sheet.getLastColumn() + 1, form.getItems().length + 2)
}

/**
 * params: { sheet, form, formResponse, row }
 */
function addEditResponseURLToSheet(params) {
  if (!params.col) {
    params.col = getFormResponseEditUrlColumn(params.sheet)
  }
  var formResponseEditUrlRange = params.sheet.getRange(params.row, params.col)
  formResponseEditUrlRange.setValue(params.formResponse.getEditResponseUrl())
}


function onOpen() {
  var menu = [{ name: 'Add Form Edit Response URLs', functionName: 'setupFormEditResponseURLs' }]
  SpreadsheetApp.getActive().addMenu('Forms', menu)
}

function setupFormEditResponseURLs() {
  var sheet = SpreadsheetApp.getActiveSheet()
  var spreadsheet = SpreadsheetApp.getActive()
  var formURL = sheet.getFormUrl()
  if (!formURL) {
    SpreadsheetApp.getUi().alert('No Google Form associated with this sheet. Please connect it from your Form.')
    return
  }
  var form = FormApp.openByUrl(formURL)

  // setup the header if not existed
  var headerFormEditResponse = sheet.getRange(1, getFormResponseEditUrlColumn(sheet))
  var title = headerFormEditResponse.getValue()
  if (!title) {
    headerFormEditResponse.setValue('Form Response Edit URL')
  }

  var timestampColumn = getTimestampColumn(sheet)
  var editResponseUrlColumn = getFormResponseEditUrlColumn(sheet)
  
  var timestampRange = sheet.getRange(2, timestampColumn, sheet.getLastRow() - 1, 1)
  var editResponseUrlRange = sheet.getRange(2, editResponseUrlColumn, sheet.getLastRow() - 1, 1)
  if (editResponseUrlRange) {
    var editResponseUrlValues = editResponseUrlRange.getValues()
    var timestampValues = timestampRange.getValues()
    for (var i = 0; i < editResponseUrlValues.length; i += 1) {
      var editResponseUrlValue = editResponseUrlValues[i][0]
      var timestampValue = timestampValues[i][0]
      if (editResponseUrlValue === '') {
        var timestamp = new Date(timestampValue)
        if (timestamp) {
          var formResponse = form.getResponses(timestamp)[0]
          editResponseUrlValues[i][0] = formResponse.getEditResponseUrl()
          var row = i + 2
          if (row % 10 === 0) {
            spreadsheet.toast('processing rows ' + row + ' to ' + (row + 10))
            editResponseUrlRange.setValues(editResponseUrlValues)
            SpreadsheetApp.flush()
          }
        }
      }
    }
    
    editResponseUrlRange.setValues(editResponseUrlValues)
    SpreadsheetApp.flush()
  }

  registerNewEditResponseURLTrigger()
  SpreadsheetApp.getUi().alert('You are all set! Please check the Form Response Edit URL column in this sheet. Future responses will automatically sync the form response edit url.')
}

function onFormSubmitEvent(e) {
  var sheet = e.range.getSheet()
  var form = FormApp.openByUrl(sheet.getFormUrl())
  var formResponse = form.getResponses().pop()
  addEditResponseURLToSheet({
    sheet: sheet,
    form: form,
    formResponse: formResponse,
    row: e.range.getRow(),
  })
}
```
