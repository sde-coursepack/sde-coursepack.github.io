---
Title: Common JavaFX Controls and Panes
---

* TOC
{:toc}

# Common Controls and Panes

This module provides an overview of some common JavaFX controls and panes. Think of this article as more of a reference, rather than something you will be expected to know inside and out. Of course, there are far more controls/panes than I can cover here, but this should be sufficient for building apps in this course. 

On this page, you can click on each component's name to learn more about the component (via Oracle's JavaFX library reference) and how to use it in your code. 

## Controls
Controls are interactive components that allow users to view or manipulate data in a JavaFX application. They include elements like buttons, text fields, sliders, and checkboxes-- anything the user can interact with or that displays information. 

### Label
A [Label](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/Label.html) is a text display used to show titles, descriptions, or short messages next to other components. They are often used to identify the purpose of nearby controls, such as text fields or buttons. Unlike a TextField or TextArea, the user cannot directly modify the Label's text.

### Button
A [Button](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/Button.html) is a clickable control that triggers an action or event when pressed. Buttons are typically associated with event handlers (like setOnAction) to perform specific tasks, such as submitting a form, opening a dialog, or changing scenes.

### TextField 
A [TextField](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/TextField.html) a single line text input field. These are commonly used for forms, search bars, or any place where the a small amount of text should be entered by the user.

When the user presses the Enter key, a TextField can trigger an event. This is useful for actions like submitting a query or validating input without requiring a separate button click.

### TextArea
A [TextArea](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/TextArea.html) is a multi-line text input field. These are commonly used for longer text entries, such as comments, and can be configured to have a certain text-wrapping behavior.

### PasswordField
A [PasswordField](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/PasswordField.html) is similar to a TextField, but hides the input characters as they are entered. This is useful in cases where the entered text needs to be kept confidential, like password or SSN entry fields.

### Separator
A [Separator](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/Separator.html) is a simple line (horizontal or vertical) used to visually divide sections of the GUI, improving readability and organization.

### Slider
A [Slider](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/Slider.html) allows users to select a numeric value by moving a thumb along a track. This is commonly used to represent adjusting a setting, such as volume or brightness.

### CheckBox
A [CheckBox](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/CheckBox.html) represents an option that can be either selected or unselected. This is useful to allow the user to configure a setting that has only two possible values, such as on/off or true/false.

### RadioButton
A [RadioButton](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/RadioButton.html) represents one choice within a set of mutually exclusive options—only one button in a group can be selected at a time.

Button Groups (using ToggleGroup) ensure that only one RadioButton in the group is selected at any time. This is commonly used in multiple-choice questions. 

### DatePicker
A [DatePicker](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/DatePicker.html) provides a text field and a dropdown calendar, allowing users to select a date easily. You can call the `getValue()` instance method to retrieve the LocalDate object representing the date that was entered/selected.

### FileChooser
A [FileChooser](https://docs.oracle.com/javase/8/javafx/api/javafx/stage/FileChooser.html) opens a standard operating system dialog to prompt the user to select a file from their computer. 

### ProgressBar
A [ProgressBar](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/ProgressBar.html) can be used to communicate the progress of a task to the user. It can be configured to display the exact progress as a percentage. 

### ListView
A [ListView](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/control/ListView.html) displays a scrollable list of items. It supports selecting an item from the list, and can be used with event handling to perform some action when an item is clicked/selected.

## Panes
Panes are layout containers that organize and position controls and other components within the scene. They determine how elements are arranged on the screen-— this may be horizontally, vertically, in a grid, or in some other custom layout. Different pane types are optimized for different kinds of layouts, and more complex layouts can be built by nesting multiple panes together. 

The components contained within a pane are often referred to as the children of the pane.

### FlowPane
In a [FlowPane](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/layout/FlowPane.html), the children are arranged in a flow that wraps to the next line or column when there’s not enough space. This is useful for dynamic layouts, like a toolbar or a list of tags or pill buttons.

### HBox
In an [HBox](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/layout/HBox.html), the children are arranged horizontally in a single row. You can control spacing and alignment to customize the exact positioning of the elements.

### VBox
In a [VBox](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/layout/VBox.html), the children are arranged vertically in a single column. This is commonly used to stack elements in the GUI, such as the labels, text fields, and buttons in a form.

### GridPane
In a [GridPane](https://docs.oracle.com/javase/8/javafx/api/javafx/scene/layout/GridPane.html), the children are organized in a flexible grid of rows and columns. This is useful when you want the GUI to have a structured layout, such as in a form or dashboard, where the controls should be aligned consistently in a tabular format.
