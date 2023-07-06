# Google-Form-Edit-Response-Link
Creating this repo for easy understanding of Google Form Edit Response

###  Reference Code

```
The Script to Use

function assignEditUrls() {
  var form = FormApp.openById('Your form key goes here');

  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Your responses Google Sheet name goes here - The tab name, not the file name');

  var data = sheet.getDataRange().getValues();
  var urlCol = Column number where URLs get entered goes here; 
  var responses = form.getResponses();
  var timestamps = [], urls = [], resultUrls = [];
  
  for (var i = 0; i < responses.length; i++) {
    timestamps.push(responses[i].getTimestamp().setMilliseconds(0));
    urls.push(responses[i].getEditResponseUrl());
  }
  for (var j = 1; j < data.length; j++) {

    resultUrls.push([data[j][0]?urls[timestamps.indexOf(data[j][0].setMilliseconds(0))]:'']);
  }
  sheet.getRange(2, urlCol, resultUrls.length).setValues(resultUrls);  
}
```

###  Reference Image

![Screenshot 2023-01-30 105626](https://github.com/Revanth-13/Google-Form-Edit-Response-Link/assets/123372740/2bec6185-d607-408f-abb4-2acf7ff3c08a)
