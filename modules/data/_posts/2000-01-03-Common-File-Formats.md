---
Title: Common Plaintext File Formats
---

* TOC
{:toc}

# Common file formats

In this article, I briefly mention some common file formats to be aware of as a programmer. All of these file formats are "plain text", meaning you could open these files in virtually any text editor to create, modify, and save them.

## `.txt` - text files

Simple text file, typically will be unstructured, like we have in the above Notepad example.

## `.csv` - Comma-separated values.

Below is a snippet of what could be a .csv file for UVA's football roster for the 2023-2024 season.

```text
LastName,FirstName,Position,Number,Year
Muskett,Tony,QB,11,4
Jones,Perris,RB,2,4
Sanker,Jonas,SPUR,20,3
```

Here, you can think of this structure as like a table, where the commas separate each *column*, and the line breaks separate each *row* (where the first row are headings). So you would interpret the above as a table like:

| FirstName | LastName | Position | Number | Year |
|-----------|----------|----------|--------|------|
| Muskett   | Tony     | QB       | 11     | 4    |
| Jones     | Perris   | RB       | 2      | 4    |
| Sanker    | Jonas    | SPUR     | 20     | 3    |

Where each row is one player, and each column holds a particular type of information.

### Advantages

.csv files are a great way to store tabular data in a plain text, human-readable format. Additional, reading from and writing to .csv files is generally very easy code to write using common File I/O and `String` functions (especially `.split(",")`).

In fact, many websites will give a CSV file when you download tabular data. In addition to the ease, this is because CSV files do not generally require much *metadata* (that is, information that is relevant to the file, but is not itself the data being transmitted). This is compared to spreadsheet files, like `xlsx files`, which is not plain text, and has a significant storage overhead.

### Disadvantages

There are disadvantages

#### Instability

Because .csv files *are* plaintext, they are generally quite limited. For example, a .csv file cannot really store any extra media, only text. Additionally, .csv files are very unstable. Consider, for a minute, that I update the above table to include a suffix for players that have them (such as Sackett Wood Jr. and Jestus Johnson III).

| FirstName | LastName | Suffix | Position | Number | Year |
|-----------|----------|--------|----------|--------|------|
| Muskett   | Tony     |        | QB       | 11     | 4    |
| Jones     | Perris   |        | RB       | 2      | 4    |
| Sanker    | Jonas    |        | SPUR     | 20     | 3    |
| Sackett   | Wood     | Jr.    | TE       | 44     | 4    |
| Jestus    | Johnson  | III    | RG       | 77     | 3    |

Which would translate to:

```text
LastName,FirstName,Suffix,Position,Number,Year
Muskett,Tony,,QB,11,4
Jones,Perris,,RB,2,4
Sanker,Jonas,,SPUR,20,3
Sackett,Wood,Jr.,TE,44,4
Jestus,Johnson,III,RG,77,3
```

While I put the `Suffix` column in a natural place, after last name, any code that used the column indices of the previous format will have issues.

For instance, previous, the position was in column index `2`. So I could get the position using that index. So a snippet of code that prints the name of each player may have looked like (where `line` is a line of data from the file, like `Muskett,Tony,,QB,11,4`):

```java
    String[] row = line.split(",");
    String position = row[2];
    if (position.equals("QB")) {
        // Print format: "[First Name] + [Last Name]"
        System.out.println(row[1] + " " + row[0]);
    }
```

Now, I *must* update the line: `String position = row[2];` because 2 is now the index for `suffix`, not `position`. This means any changes to the structure of your CSV file will almost certainly necessitate you changing the source code that reads that file. This is also why it's a good design idea to hide how a file is read inside a class with an abstract interface, because data formats *will* change!

#### Plaintext limitations

Imagine if you were making a table of data tracking the Academy Awards history for Best Picture Nominees and winners. Something like:

| Year | Title                             | Winner |
|------|-----------------------------------|--------|
| 2022 | Everything Everywhere All At Once | true   |
| 2022 | All Quiet on the Western Front    | false  |
| 2023 | Avatar: The Way of Water          | false  |

You start filling in the data, and you get to the movie Tár (2022). That **á** is going to be a problem, because if you are using many common code editors, they use the ASCII character set. For example, if your file contained:

```java
    String title = "Tár";
```

...this would actually produce a syntax error, because á is not an ASCII character. Instead, you would have to use the unicode encoding of the character á

```java
    String title = "T\u00E1r"
```

When dealing with special characters, especially non-English characters, your editor may not support them, so you will need to save the data as Unicode encoded data. I'm not showing how to do that in this module, as it is not necessary for the class. However, it is a problem to be aware of, especially when reading data from the internet which will often be encoded in Unicode rather than ASCII

#### Data containing commas

So with that problem solved, you keep going back in history, until you get to ["Three Billboards Outside Ebbing, Missouri"](https://en.wikipedia.org/wiki/Three_Billboards_Outside_Ebbing,_Missouri) - a 2018 nominee for Best Picture and one of my personal favorite films. If we add it to our table:


| Year | Title                                     | Winner |
|------|-------------------------------------------|--------|
| 2018 | Three Billboards Outside Ebbing, Missouri | false  |

...we run into a problem. Remember, the above row would be represented as:

`2018,Three Billboards Outside Ebbing, Missouri,false`

If we split this String on commas, we would assum:
* Index 0 is the year - integer
* Index 1 is the title - text
* Index 2 is whether the film won best picture - boolean

However, if we try to get index 2 for the above after splitting on commas (`.split(",")`, we get `" Missouri"`. This is because our `split` call cannot tell the difference between a comma that separates columns, and a comma that is part of a piece of text. The typical solution to this is, when a piece of text contains a comma, to but it inside quotation marks:

`2018,"Three Billboards Outside Ebbing, Missouri",false`

And then, instead of using `.split(",")`, you can use regular expressions to split "on commas not inside of quotation marks". This is...quite a bit more complicated, so I'll just cite [this StackOverflow answer](https://stackoverflow.com/questions/21105360/regex-find-comma-not-inside-quotes), which would mean using `.split("(?!\B"[^"]*),(?![^"]*"\B)")`. 

#### Lack of nesting/listing

## .tsv - Tab separated values

.tsv files are basically the same thing as .csv files, only using Tab characters (`"\t"`) instead of commas. All the advantages/disadvantages of CSV are the same.

## JSON, XML, YAML

We will spend then next several modules looking at and using .json, .xml, and .yaml files. Fundamentally, all three of these files store data in the same way, just using different syntax. Specifically, these files organize data into nested structures of labeled data. We will start by looking at .json in the next unit.