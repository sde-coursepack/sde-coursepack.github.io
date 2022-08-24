---
Title: Gradle Example with Poi
---

## [Source Code Example](https://github.com/sde-coursepack/NBAExcelTeams)

# poi

Apache Poi is a library for reading and writing Microsoft
Office File Formats (like Word, Excel, etc.) in Java. For
the sake of this tutorial, we are going to focus on
writing to and reading Excel files.

In this example, we will be writing information about NBA
teams to an Excel spreadsheet.

## org.apache.poi

There are a number of Apache Poi libraries. The specific packages
we want for reading Excel files are:

* `org.apache.poi.ss` - Library for reading and writing an abstract SpreadSheet
* `org.apache.poi.xssf` - Library for reading and writing a `.xlsx`, or Excel 2007 and later, spreadsheet.

XSSF stands for "XML Spreadsheet Format" because the file 
format is XML based. There is also a library for reading from
older, .xls spreadsheets, which is HSSF. However, we are going
to focus strictly on the newer and better `.xlsx` file format.

## Adding poi with Gradle

Both of these packages are in the Apache POI-OOXML library, [which
can be found here.](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)
However, you do not have to download or install anything on
your own to make this work.

Instead, we will go to the link, click the most recent version (as
of this writing, 5.2.2), and then from the tabbed area below the
version information, click the Gradle tab:

![Shows the "Gradle" tab clicked on the mvnrepository website.](../images/demo/gradle_poi_import.png)

From there, we copy the text that says:

`implementation group: 'org.apache.poi', name: 'poi-ooxml', version: '5.2.2'`

...and then simply paste that into our build.gradle file
inside of the `dependencies` closure. Thus, our dependencies will
like: 

```groovy
dependencies {
    implementation group: 'org.apache.poi', name: 'poi-ooxml', version: '5.2.2'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.9.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.9.0'
}
```

Note that the org.junit.jupiter libraries are loaded by default
in IntelliJ because of how popular the testing framework is. Leave
those libraries in there!

### log4j

Now, our dependencies may have dependencies of their own. In the
case of `poi-ooxml`, it uses the Apache logging tool `log4j`. However, the
poi-ooxml does not contain this dependency. So we need to include it
separately. It is worth noting our code *can* work without `log4j`,
but it will give an error message if log4j is not available at runtime.

To include it, we simply search for log4j, [which can be found here.](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)
Once again, we select the latest version (2.18.0 at the time of this writing),
and copy the Gradle import statement from [that version's web-page.](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api/2.18.0)

`implementation group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.18.0'`

And drop it in our dependencies as well:

### implementation vs. runtime

None of our code will actually use `log4j` in this demo. We only
need to include `log4j` because it is used by `poi-ooxml` at runtime.
As such, we can change our build dependencies to:

```groovy
dependencies {
    implementation group: 'org.apache.poi', name: 'poi-ooxml', version: '5.2.2'
    runtimeOnly group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.18.0'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.9.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.9.0'
}
```

The specific difference is that `log4j` is now listed as `runtimeOnly`
rather than `implementation`. The advantage here is that when
we are compiling, but not running, our code, we won't have to
load in the `log4j` library, making the compiling process slightly
more efficient. The change is negligible in this case, but if
you have a project pulling from dozens of libraries, this can
start to add up.

The short way to understand this:
* If you ever import a package from a library in your code, it must be in `implementation`
* If you depend on a package, but do not import it in your code, it can be in either `implementation` or `runtimeOnly`
  * `runtimeOnly` is generally preferred in this case, but not required.

If you're ever unsure, it's fine to just leave your dependencies as `implementation`

### implementation vs testImplementation

You'll notice that the JUnit libraries are listed as:
* `testImplementation`
* `testRuntimeOnly`

This is because we are separating the process of **testing our
code** from the process of **running our code**. 

**When we run this program**, we want to have the program either write
information about NBA teams to an Excel file.

**When we test this program**, we are running our battery of tests
against the code to ensure there are no known defects.

In fact, we can run our tests without building using:

./gradlew test

We will talk more about testing in the Testing unit.

### Connecting with IntelliJ

Sometimes, even after building, IntelliJ's Java editor may not
immediately recognize the libraries you've imported. If this happens,
you can either close and re-open the project, or, on the right side
of the screen, open the Gradle Tab and click the "Refresh" symbol
for "Reload all Gradle Projects".

![Showing the location of "Reload all Gradle Projects" button](../images/demo/reload_gradle.png)




### Relevant Classes

We're going to look at the classes we need to use poi first. We will
break this into the abstract interfaces, and the concrete classes
that implement the interfaces for an `.xlsx` Spreadsheet

#### Abstract interfaces

Package: `org.apache.poi.ss.usermodel.*`

`Workbook` - represents a generic Spreadsheet style workbook
`Sheet` - represents an individual generic Spreadsheet in a Workbook
`Row` - represents a row in a generic spreadsheet
`Cell` - represents a single cell in a generic spreadsheet

You'll note there is no class for Column. We typically access columns
via a getColumn(int index) function on Sheet or Row

#### Concrete interfaces

Package: `org.apache.poi.xssf.usermodel.*`

`XSSFWorkbook` - represents a `.xlsx` Workbook
`XSSFSheet` - represents an individual Spreadsheet in an `.xlsx` spreadsheet
`XSSFRow` - represents a row in an `.xlsx` spreadsheet
`XSSFCell` - represents a cell in an `.xlsx` spreadsheet

#### Which to use

As a general design principle, we want our code to be as abstract as possible.
This is because if, for example, Excel made a new file format, we could expect in
time `org.apache.poi` to have packages for that new file format. For example
if, say, Excel may a JSON based format, it might be something 
like `org.apache.poi.jssf.usermodel.*`. If, as much as possible, we use the
abstract classes, then our code is much more reusable.

Consider the following function

```java
    private void populateSaveAndCloseWorkbook(List<NBATeam> nbaTeamList, Workbook workbook) throws IOException {
        Sheet worksheet = workbook.createSheet("NBA Teams");
        Row titleRow = getNextRow(worksheet);
        putStringArrayInRow(titleRow, COLUMN_HEADERS);
        addTeamsToWorksheet(nbaTeamList, worksheet);
        resizeColumns(worksheet);
        saveAndCloseWorkbook(workbook);
    }
```

This function takes in a List of NBA Teams, and a Workbook. Is this Workbook
for '.xls', `.xlsx`, or something cool we haven't even heard of yet? It doesn't
matter! In the same way we don't care what *kind* of List the first parameter is (ArrayList
Vector, LinkedList), we don't care what *kind* of Workbook the second parameter is.

The only time we need to reference the concrete-class is at creation. Thus,
we can just create the WorkBook, and then pass it to the above function.

```java
    Workbook myWorkbook = new XSSFWorkbook();
    populateSaveAndClose(myWorkbook);
```

After that, we can treat it like a generic workbook to ensure our code
is reusable and flexible.

## Writing to an Excel Workbook

To create a new `.xlsx` Excel Workbook, we have to do the following steps:
1) Create a new XSSFWorkbook Object
2) Populate the workbook with data
3) Save the Workbook object with the intended filename

Let's say we wanted to generate the following spreadsheet. We've already covered
1, so now let's dive into the [class `NBATeamsXSLXWriter`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/NBATeamXSLXWriter.java)
the function `populateSaveAndCloseWorkbook` in our code. We will go line by
line, diving into the helper-functions as needed.

### Worksheet

```java
    Sheet worksheet = workbook.createSheet("NBA Teams");
```

This creates a new worksheet. In an excel workbook, you can have multiple
sheets if you want. But we must create our sheet first. Every sheet has a
name, so our name is "NBA Teams". This worksheet is initially blank.

### Row

When looking at the next line, we are calling the private helper function 
getNextRow(). This uses the private class field `rowCount`, which we are
using to keep track of the next row that we can write to.

```java
    private Row getNextRow(Sheet worksheet) {
        Row newRow = worksheet.createRow(rowCount);
        rowCount++;
        return newRow;
    }
```

The above method gets a new Row object using the function 
`worksheet.createRow(rowCount)`. It then increments rowCount so that
the next time we call this function, we get the next row below the
one we just created. This new Row, like our Worksheet and Workbook, is
initially empty. So now we need to know how to create Cells:

### Cell

Looking at the function `putHeaderStringArrayInFirstRow`, we see:

```java
    private void putHeaderStringArrayInFirstRow(Row titleRow, String[] COLUMN_HEADERS) {
        for (int columnIndex = 0; columnIndex < COLUMN_HEADERS.length; columnIndex++) {
            Cell newCell = titleRow.createCell(columnIndex);
            newCell.setCellValue(COLUMN_HEADERS[columnIndex]);
        }
    }
```

Here, for each String we want to put into the Row `titleRow`, we:
1) Create a new Cell with the column number we want, where 0 is the first column 
2) Use setCellValue to set the contents of the Cell.

Note that setCellValue will also set the cells type. For example:

* If you input a String, the cell type will be Text
* If you input an int or double, the cell type will be Numeric
* If you input a java.time.Date class, the cell type will be Date, etc.
* Separately, you can enter formula Strings with setCellFormula(String formula) if you are familiar with Excel formula syntax


### Styling

There are also a number of options to enter styling, but that's a pretty
deep rabbit hole with a ton of options, and beyond the scope of what we need here.


I do want to briefly point out one element of styling in the function `resizeColumns`

```java
    private void resizeColumns(Sheet worksheet) {
        for (int columnIndex = 0; columnIndex < COLUMN_HEADERS.length; columnIndex++) {
            worksheet.autoSizeColumn(columnIndex);
        }
    }
```

This is just a simple for-loop that goes through each column and autoSizes them so no text is cut off.
This is about as far into styling as I want to go, but it's worth doing so your spreadsheet is readable.


### Saving the file

Finally, let's look at how our file is saved:

```java
    private void saveAndCloseWorkbook(Workbook workbook) throws IOException {
        FileOutputStream fileOut = new FileOutputStream(excelFilename);
        workbook.write(fileOut);
        fileOut.close();
        workbook.close();
    }
```

Here, I'm just using a standard file-writing approach with a `FileOutputStream`.
The `poi` library handles actually writing the file, I just need to tell
the `Workbook` object **where** I want the file written. In this case,
I'm writing the filename specified in the classes constructor.

Always make sure to close both the `FileOutputStream` and the `Workbook`! If you
don't close, the file may appear to not be written, as it's still open in
your program's memory.

## Reading from XSSF Workbook



## Useful functions