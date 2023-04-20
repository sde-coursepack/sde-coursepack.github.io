---
Title: Buttons and Events
---

# Buttons and Events

In this unit, we will build on our code from the last module to add a Button and an EventHandler.

## Starting Code

We will begin with the same starter code as the last module. However, for space, I have removed the import statements and comments, as we have already covered these basics. If you wish to follow along, you can either copy the import statements from the last unit or allow IntelliJ to add the correct imports for you (though make sure you are importing the classes in the `javafx` package).

```java
public class HelloWorld extends Application {
    @Override
    public void start(Stage primaryStage) throws Exception {
        Label helloLabel = new Label("Hello World!");
        
        Pane root = new FlowPane();
        root.getChildren().add(helloLabel);
        
        Scene scene = new Scene(root);
        primaryStage.setScene(scene);
        primaryStage.show();
    }
}
```

## Adding a button

Now, let's create a Button and add it to our application. As yet, our Button won't *do* anything yet, but that will be the next step.

## Button EventHandler

Clicking a button generates an Event object (think event-driven programming!). Specifically, this is an `ActionEvent` object. This object contains information about the Event, but for the sake of this app, we don't need to dive into its inner workings. In order to "do something" with the button, we want to tell our application how to **handle** that ActionEvent. This involves creating an `EventHandler` class.

The `EventHandler<T>` is an *interface*, so we are creating a class that `implements` this interface. Specifically, we are creating a class that implements the `EventHandler<ActionEvent>` interface. The interface has only one method:

```java
    public void handle(ActionEvent event);
```

This method defines the procedure that is run whenever the particular event tied to that handler occurs. In this case, when our `Button` is clicked, the `handle` function is called. Like the `start` method, we never directly call this method. Rather, it is invoked by the JavaFX framework.

## Creating an EventHandler

There are several ways to create an EventHandler and attach it to our button. In this module, we will show 4 different ways:

1) Public External Class
2) Private Inner Class
3) In-Line Anonymous Class
4) In-Line Lambda Body

There is another way to attach a function to an EventHandler using a JavaFX format file (.fxml file). We will cover that strategy in the upcoming FXML and MVC with Java unit.

### External Class

### Inner Class

### Anonymous Class

### Lambda Body

## Final Code