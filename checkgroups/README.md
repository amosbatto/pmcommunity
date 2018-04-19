## Accessing Checkgroups with JavaScript
 
**Author:**    Amos Batto (amos@processmaker.com)  
**Tested in:** ProcessMaker 3.1.3 Community in Firefox 51 (probably will work in all 3.*X* versions)  
**License:**   Public Domain  

### Getting a checkgroup's list of options 

A checkgroup consists of a set of checkboxes. The list of options (i.e., checkboxes) in a checkgroup 
can be obtained in the <a href="http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.getInfo">model information</a> with the 
following JavaScript code: 
```javascript
$("#checkgroup-id").getInfo().options
```
For example, the following checkgroup has the ID "selectServices":


The following code gets the list of options in the "selectServices" checkgroup:
```javascript
var aOpts = $("#selectServices").getInfo().options
```
Now the variable `aOpts` has the following content:
```javascript
[
  {
    label:    "Accounting & Auditing",
    value:    "accounting", 
    selected: false
  }, 
  { 
    label:    "Cleaning Service",
    value:    "cleaning",  
    selected: false
  },
  { 
    label:    "Maintenance & grounds",  
    value:    "maintenance",  
    selected: false
  }
]
```
The `selected` property is set to `true` if the checkbox is marked *by default* when the Dynaform was first loaded.
  
Use the `.length` property of the `options` array to get the number of checkboxes in a checkgroup:
```javascript
var numberOptions = $("#selectServices").getInfo().options.length
```
### Getting currently selected options

If needing to get the current state of the checkboxes, then use 
[*control*.getValue()](http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.getValue), 
not the `selected` property. The `*control*.getValue()` method 
returns an array of the values of the marked checkboxes in the checkgroup.
 
For example:
```javascript
var aValues = $("#selectServices").getValue();
```
Now the `aValues` variable contains the array `["accounting", "maintenance"]`, 
since the first and third checkboxes in the checkgroup are marked.

Likewise, use the [*control*.getText()](http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.getText) method to get 
a JSON string holding an array of the marked checkboxes' labels. [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) 
can be used to convert the string into an array of the selected labels. 

For example:
```javascript
var aLabels = JSON.parse($("#selectServices").getText());
```
Now the `aLabels` variable contains the array `["Accounting & Auditing", "Maintenance & grounds"]````.

For example, the following JavaScript code uses the 
[*control*.setOnchange()](http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.setOnchange) 
method to set an event handler to check whether the "cleaning" checkbox is marked when the checkgroup changes. 
[$.inArray()](https://api.jquery.com/jQuery.inArray/) is used to search for "cleaning" in 
the array of selected values of the checkgroup. If it is found, then the textarea "areasToClean" is shown; 
otherwise it is hidden.
```javascript
function hideShowAreasToClean(newServices, oldServices) {
   //if "cleaning" is selected, then show "areasToClean" textarea:
   if ($.inArray("cleaning", newServices) != -1) {
      $("#areasToClean").show();
   }
   else {
      $("#areasToClean").hide();
   }
}
$("#selectServices")setOnchange(hideShowAreasToClean);      //when checkgroup changes
hideShowAreasToClean($("#selectServices").getValue(), []);  //when DynaForm loads
```
### Setting selected options

The [*control*.setValue()](http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.setValue) 
method can be used to mark checkboxes in a checkgroup. `*control*.setValue()` takes 
an array of the checkbox values to mark in the checkgroup. Note that the values 
have to be listed in the array *in the same order* as they appear in the checkgroup. 

For example, the following code marks the first and third options in the "selectServices" checkgroup:
```javascript
$("#selectServices").setValue(["accounting", "maintenance"]);
```
If the order is reversed, and the third option is listed first in the array and the first option is listed last, 
then the checkboxes will not be marked correctly. This code will **NOT** will not mark the 
checkboxes, because the options are listed out of order:
```javascript
$("#selectServices").setValue(["maintenance", "accounting"]);
```
If needing set just one checkbox, but leave the other checkboxes unchanged in the checkgroup, then use 
`$.inArray()` to first check whether the checkbox is not already marked. If unmarked, 
then mark it using [*array*.push()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push) 
to add the value of the checkbox to the array. 
Then, loop through the list of `getInfo().options` to recreate the array in the proper order and 
pass that array to `setValue()`.

For example, the following JavaScript code marks the "accounting" option in the 
"selectServices" checkgroup, if it is not already marked:
```javascript
var aVals = $("#selectServices").getValue();
if ($.inArray("accounting", aVals) != -1) {
  aVals.push("accounting");
  var aOrderedVals = [];
  var aOpts = $("#selectServices").getInfo.options;
  for (i in aOpts) {
     if ($.inArray(aOpts[i].value, aVals)) {
        aOrderedVals.push(aOpts[i].value);
     }
  }
  $("#selectServices").setValue(aOrderedVals);
}
```
Similarly, the following code uses `setValue()` to unmark (unselect) the "accounting" checkbox in 
the "selectServices" checkgroup if the checkbox is not already unmarked. It uses the 
[*array*.splice]()(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) 
method to remove the value from the array of selected values.
```javascript
var aVals = $("#selectServices").getValue();
var pos = $.inArray("accounting", aVals);
if (pos != -1) {
  aVals.splice(pos, 1);
  $("#selectServices").setValue(aVals);
}
```
### HTML structure of checkgroups

Checkgroups have the following HTML structure:
```html
<div id="selectServices" class="pmdynaform-field-checkgroup  form-group col-sm-12 col-md-12 col-lg-12  
    pmdynaform-edit-checkgroup pmdynaform-field">
  <label for="form[selectServices]" class="col-sm-2 col-md-2 col-lg-2 control-label pmdynaform-label">
    <span class="textlabel">Select Services</span>
  </label>
  <div class="col-sm-10 col-md-10 col-lg-10 pmdynaform-field-control">
    <div class="pmdynaform-control-checkbox-list form-control" style="height: auto;">
      <div class="pmdynaform-checkbox-items">
        <div class="checkbox">
          <label>
            <input id="form[selectServices][accounting]" name="form[selectServices][]" 
                class="pmdynaform-control-checkgroup" value="accounting" type="checkbox">
            <span>Lawn Service</span>
          </label>
        </div>
        <div class="checkbox">
          <label>
            <input id="form[selectServices][cleaning]" name="form[selectServices][]" 
                class="pmdynaform-control-checkgroup" value="cleaning" type="checkbox">
            <span>Cleaning Service</span>
          </label>
        </div>
        ...
      </div>
    <input id="form[selectServices_label]" name="form[selectServices_label]" 
        value="[&quot;Accounting&quot;,&quot;,&quot;Maintenance&quot;]" type="hidden">
  </div>
</div>
```
Each checkbox in the checkgroup is placed inside a `<div>`, whose class is named "checkbox". 

The following JavaScript code hides the first checkbox and shows the third checkbox in the "selectServices" checkgroup:
```javascript
$("#selectServices").find("div.checkbox").eq(0).hide()
$("#selectServices").find("div.checkbox").eq(2).show()
```
In this example, [.find()](https://api.jquery.com/find/) returns an array of the `div`s 
holding checkboxes and [.eq()](https://api.jquery.com/eq/) is used to select one of them. 

Note that arrays start counting from the number `0`, so the first checkbox is 
element `0` in the array and the third checkbox is element `2`. 

The checkboxes in checkgroups can also be found by searching for their value. 
The following code has the same effect as the previous code example:
```javascript
$("#selectServices").find("input[value=accounting]").parent().hide()
$("#selectServices").find("input[value=maintenance]").parent().show()
```
Note that hiding a checkbox does not prevent its value from being saved when the DynaForm is submitted. 

To disable a checkbox in a checkgroup, set its `disabled` property to `true` 
and set the color of the `<span>` holding the checkbox's label to grey. 

The following code disables the "accounting" checkbox and enables the "maintenance" checkbox:
```javascript
$("#selectServices").find("input[value=accounting]").prop("disabled", true)
$("#selectServices").find("input[value=accounting]").siblings("span").css("color", "grey")
$("#selectServices").find("input[value=maintenance]").prop("disabled", false)
$("#selectServices").find("input[value=accounting]").siblings("span").css("color", "") //return to default color
```
### Accessing checkboxes in a checkgroup

The [*control*.getControl()](http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.getControl) 
method will return an array of all the checkboxes in a checkgroup. 

Use [.eq()](https://api.jquery.com/eq/) to access a particular checkbox in that array and 
remember that counting in arrays starts from `0`. 

For example, the following code sets the label of the second checkbox in the 
"selectServices" checkgroup. It uses [.siblings()](https://api.jquery.com/siblings/) 
to select the `<span>` element next to the checkbox which holds the checkbox's label. 
```javascript
$("#selectServices").getControl().eq(1).siblings("span").html('Cleaning Service')
```
It is also possible to do the same thing by searching for the checkbox's value:
```javascript
$("#selectServices").find("input[value=cleaning]").siblings("span").html('Cleaning Service')
```

### Using `*checkgroup*.setOnchange()`

The [*control*.setOnchange()[http://wiki.processmaker.com/3.1/JavaScript_Functions_and_Methods#control.setOnchange]
for checkgroups has its own idiosyncrasies. The `newValue` in the event's handler function will be an
array of the values of the currently selected options. However, the `oldValue` will be a JSON string holding an
array the values of the previously selected options. In order to use that array, 
it needs to first be decoded with [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse).

For example, a checkgroup with the ID "selectCountries" needs to limit the
number of selected options to 3 or less. If the user has selected more than 3 options,
then, the following JavaScript returns the checkgroup to its the previous value. 
Notice how the code decodes the `oldVal` before using it:
```javascript
$("#selectCountries").setOnchange(function(newVal, oldVal) {
  //three selected options is the maximum allowed:
  if (newVal.length > 3) { 
    previousVals = JSON.parse(oldVal);
    $("#selectCountries").setValue(previousVals);
  }
}) 
``` 
