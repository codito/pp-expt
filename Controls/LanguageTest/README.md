# Language Test

Solution will add a main form which tests this control. Open the Test App in the
dynamics org instance.

## Challenge

The entity has a `test_languagecode` field which is of type `Whole.Language`.
PCF doesn't allow bound properties of this type.

Objective: the caller of this form will provide value for this field. We want
the form to set this field value.

## Explorations

Use the following command in the Chrome Developer Tools to navigate to the form.

```javascript
var entityOptions = {};
entityOptions["entityName"] = "test_languagetest";
entityOptions["formId"] = "25B957FB-667A-408D-83C0-15BB90B33068"

var formParameters = {};
formParameters["test_languagecode"] = 1036;  // French
formParameters["test_languagetext"] = "abc";
formParameters["test_name"] = "record3";

Xrm.Navigation.openForm(entityOptions, formParameters).then(function (lookup)
{ console.log("Success"); }, function (error) { console.log("Error"); });
```

1. PCF restricts all inputs in `context.parameters` dictionary to only the bound
   field values. This approach will not work. Even input parameters can only be
   statically bound to constants or to different field values.
2. Can we use `context.page` to access the Page attributes and set the value?
   No.
3. Can we use the `Xrm.Page` APIs to access the attribute and set value? Yes.
   ```javascript
   Xrm.Page.getAttribute("test_languagecode").setValue(1031); // German
   ```
4. But `Xrm.Page` is deprecated. An alternative is to bind a Web Resource to the
   `form.onLoad` event to set a global `formContext`. Now within the PCF, use
   this API to set any value as necessary. See this blog post: <https://debajmecrm.com/getting-formcontext-in-power-apps-custom-component-framework-gotchas/>
   As a pattern, this is actually creating a communication bridge between the
   main form and the PCF control.
   See more on `formContext` here: <https://docs.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/clientapi-form-context>
5. By default, the platform uses a out of box `LanguagePickerControl` to bind to
   the `Whole.Language` fields. Can a PCF have this control as a child?
6. `Whole.Language` is stored as a number in the database. How does the
   LanguagePicker derive the displayed value? `Whole.Language` field has
   a `LanguageByCode` attribute which is a map of all language codes (installed
   in the system) to the formatted values. Display (updateView) method uses this
   to find the display string.
7. Is there another way to find the formatted string? Yes. Use
   `context.formatting.formatLanguage(1031) to display "German (German)"`.