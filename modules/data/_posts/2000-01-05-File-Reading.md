---
Title: Plaintext File Reading
---

* TOC
  {:toc}

# Reading from a CSV File

This article will give a code example for reading from a plaintext .csv files using a `BufferedReader`. Be aware that this is considered **pre-requisite knowledge** for the course. It is included here, however, as an example for your benefit.

For this example, we will be using [this color dataset .csv file.](https://raw.githubusercontent.com/codebrainz/color-names/master/output/colors.csv)

Specifically, we want to create program where the user can type a color name and get the RGB values of that color.

## `Color.java`

First, we will create a class to store the color RGB values:

```java
public class Color {
    private int red, green, blue;

    public Color(int red, int green, int blue) {
        this.red = red;
        this.green = green;
        this.blue = blue;
    }

    @Override
    public java.lang.String toString() {
        return "Color{" +
                "red=" + red +
                ", green=" + green +
                ", blue=" + blue +
                '}';
    }
}
```

We will also want a map of Names to Color objects. We don't need much functionality, just a way to add name/color pairs, check if a color (by name) exists, and get the RGB colors given a name. We also want to write our map such that it ignores case (which is why we normalize case using .toLowerCase() below).

## `NameColorMap`

```java
public class NameColorMap {
  private final Map<String, Color> nameColorMap;

  public NameColorMap() {
    nameColorMap = new HashMap<>();
  }

  public void addColor(String name, Color color) {
    nameColorMap.put(name.toLowerCase(), color);
  }

  public boolean hasColor(String name) {
    return nameColorMap.containsKey(name.toLowerCase());
  }

  public Color getColor(String name) {
    return nameColorMap.get(name.toLowerCase());
  }
}
```

Now, we need to read in [our .csv file](https://raw.githubusercontent.com/codebrainz/color-names/master/output/colors.csv), building a color from the RGB values, and assigning a name from the name value. Here is a snippet of the .csv file:

```text
alabama_crimson,"Alabama Crimson",#a32638,163,38,56
alice_blue,"Alice Blue",#f0f8ff,240,248,255
alizarin_crimson,"Alizarin Crimson",#e32636,227,38,54
```

Using this, we can determine the columns we need. For this, assume the first column is column 0:
* Name - Column 1
* Red value (0-255) - Column 3
* Green value (0-255) - Column 4
* Blue value (0-255) - Column 5

Using this information, we can write a class that parses our .csv file.

## `CSVColorReader.java`

```java
public class CSVColorReader {
    private String filename;

    public CSVColorReader(String filename) {
        this.filename = filename;
    }

    public NameColorMap getNameColorMap() {
        try (BufferedReader bufferedReader = new BufferedReader(new FileReader(filename))){
            NameColorMap colorMap = new NameColorMap();
            for (String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine())
            {
                addLineToColorMap(line, colorMap);
            }
            return colorMap;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private void addLineToColorMap(String line, NameColorMap colorMap) {
        String[] splitLine = line.split(",");
        String name = getName(splitLine);
        Color color = getColor(splitLine);
        colorMap.addColor(name, color);
    }

    /**
     * Returns the color name (index 1) with quotation marks removed from the line array split by commas
     */
    private String getName(String[] splitLine) {
        return splitLine[1].replace("\"","");
    }

    private Color getColor(String[] splitLine) {
        int red = Integer.parseInt(splitLine[3]);
        int blue = Integer.parseInt(splitLine[4]);
        int green = Integer.parseInt(splitLine[5]);
        return new Color(red, green, blue);
    }

    
}
```

A quick note that the line:

```java
  try (BufferedReader bufferedReader = new BufferedReader(new FileReader(filename))){
```

Is an example of a **try-with-resource** block. The try-with-resources block will automatically close the resource created (in this case, the BufferedReader) once the `try` block is finished.

This also sets me up to handle checked File I/O Exceptions. Because opening a file in Java requires handling `FileNotFoundException`s and reading from a file requires handling `IOExceptions`, I'm using the `try-catch` to "handle" those. Since I can't handle them in a meaningful way, I just throw them as `RuntimeException`. This is to avoid using the `throws` declaration on the function, which I discuss in the [Exceptions Best Practices](https://sde-coursepack.github.io/modules/refactoring/Exceptions-Best-Practices/#checked-exceptions) unit.

Specifically, the function `addLineToColorMap` is where we handle turning a line like:

```text
alizarin_crimson,"Alizarin Crimson",#e32636,227,38,54
```

We split the line on commas using the `String.split(",")` function call. This would mean that the `splitLine` variable is a `String[]` array:

```text
  ["alizarin_crimson", "\"Alizarin Crimson\"", "#e32636", "227", "38", "54"]
```

We get the color's name from `splitLine[1]`, removing the quotation marks in the data, with the `getName` function.

```java
    private String getName(String[] splitLine) {
        return splitLine[1].replace("\"","");
    }
```

We build our `Color` object using the `getColor` function. Notice that we have to use the `Integer.parseInt` function to get the numeric values of `red`, `green` and `blue`. This is because the numbers are still `String`s, even after we split the array on commas. Be aware that you will always need to use a parsing function when extracting numeric data from a text source.

```java
    private Color getColor(String[] splitLine) {
        int red = Integer.parseInt(splitLine[3]);
        int blue = Integer.parseInt(splitLine[4]);
        int green = Integer.parseInt(splitLine[5]);
        return new Color(red, green, blue);
    }
```

Below is an example run of the program.

![color_png_demo.png](..%2Fimages%2F2%2Fcolor_png_demo.png)
