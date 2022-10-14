# Csv-import
Allows your end users to import a CSV and post the results to a table or custom query.

[New Recording - 14_10_2022, 13_56_38.webm](https://user-images.githubusercontent.com/3524181/195852636-5fe013e0-b9c2-49eb-bab0-b7715a4cd638.webm)



## Properties

### Table
You must select a table whose schema exactly matches that of the import file.

If you need to transform the data into a different schema, you should create a Budibase automation based on a "Create Row" trigger on your import table.

### onImport

Choose which actions you want to perform once the import is complete

### Text

To allow for non-English language, the labels of the Buttons and Dropzone may be changed.

## Instructions

To build your new  plugin run the following in your Budibase CLI:
```
budi plugins --build
```

You can also re-build everytime you make a change to your plugin with the command:
```
budi plugins --watch
```

