---
Title: Refactoring
---

Most of the material in this unit is derived from:

* [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/B08X8ZXT15/) by Robert C. Martin, a book that I highly recommend and dramatically improved my own outlook on designing and writing software.

* [Refactoring.Guru](https://refactoring.guru/) - a great and highly recommended website for thinking about refactoring and software design. We will reference this website again, especially during design patterns

# Refactoring

**Refactoring** is the process of making changes to the code to improve the **internal** software quality, without making any changes to the functionality of the software. Typically, we use refactoring to improve the design, analyzability, testability, and maintainability of our software.

For example, in the unit on Analyzability, we turned this:

```java
public List<int[]> asdfasdf(){List<int[]> dfghdfgh=new ArrayList<>();for(int[] ouertioert:kjsdfgklkjsdfg){if(ouertioert[0]==4)dfghdfgh.add(ouertioert);}return dfghdfgh;}
```

Into this:

```java
    public List<Cell> getFlaggedCells() {
        List<Cell> flaggedCells = new ArrayList<>();
        for (Cell cell : gameGrid.getCells()) {
            if (cell.isFlagged()) {
                flaggedCells.add(cell);
            }
        }
        return flaggedCells;
    }
```

This wouldn't change the **external** behavior of our program, but it would make the code more understandable and more tolerant to implementation changes.

---

## Refactoring in IntelliJ

### Extract constants

### Renaming Identifiers

### Renaming Classes

### Moving classes between packages

### Change Signature

## Other Refactoring Techniques

### Replace Array with Object

### Simplifying conditional logic

### Replace Error Code with Exception

### Preserve Whole Object

### Getters for mutable collections return copies