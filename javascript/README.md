There are a number of undocumented JavaScript methods that can called in BPMN Dynaforms found in ProcessMaker version 3.0 and later.

------------
### *form*.isValid()

The __*form*.isValid()__ function defined in workflow/public_html/lib/pmdynaform/build/js/PMDynaform.js:5962 
can be called to validate all the fields in the Dynaform. It shows red error messages under blank fields whose
**required** property is enabled and under fields whose value does not match their **validate** 
regular expression.

**getFormById("_form-id_").isValid()**

**Return value:**  
It returns `true` if all fields in the Dynaform are good and `false` if any are . 

**Example:**  
The following JavaScript code first checks that all the fields are valid, before calling
the *form*.saveForm() method.
```javascript
var formId = $("form").prop("id");
if (getFormById(formId)isValid == true) {
   $("#"+formId).saveForm();
}
```

------------------
### *field*.validate()
The __*field*.validate()__ function validates the value in a specified field. 
It shows the red error message for an empty required field or a text field which doesn't match its 
validate regular expression. Otherwise is shows nothing in the Dynaform. 

**getFieldById("_field-id_").validate()**

**Return value:**  
The function returns the field's model object, regardless of whether the field's value if validated or not. 

**Example:**  
When the value of the "howContact" dropdown changes to "home_visit", validate the "address" 
field to make sure that it isn't left empty.
```javascript
$("#howContact").setOnchange( function(newVal, oldVal) {
   if (newVal == "home_visit") {
      getFieldById("address").validate();
   }
}
```

------------------
### *field*.showRequired() and *field*.hideRequired()

The __*field*.showRequired()__ method shows the red asterisk (`*`) next to a specified field's label to show that it is required. then use the following methods:

__getFieldById("*field-id*").showRequired()__

The __*field*.hideRequired()__ hides the red asterisk (`*`) for required fields:
getFieldById("field-id").hideRequired()

These methods can only be used on fields that are marked as **required** or have a defined **validate** 
property. Unless there is a compelling reason to hide the asterisk without effecting the validation,
it is recommended to use the enableValidation() and disableValidation() instead of these methods.

---------------------

### *form*.items.asArray()

__*form*.items.asArray()__ returns an array of all the fields in a Dynaform. 

__getFormById("*form-id*").items.asArray()__

Example:
```javascript
var aFields = getFormById( $("form").prop("id") ).items.asArray();
for (var i = 0; i < aFields.length; i++) {
   aFields[i].validate();
}
```
