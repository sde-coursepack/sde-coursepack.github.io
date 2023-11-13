---
Title: Hello World GUI
---

# Hello World GUI

In this module, we will iteratively create a very simple application, introducing JavaFX basics one at a time.

Note that in this unit I will paste code containing a large number comments. Generally, this is bad style, and I am not encouraging students to duplicate it. However, because JavaFX is likely a first experience for my studnets with GUI development, I want to ensure each step of the way that students understand why a particular line of code was written.

* TOC
{:toc}

## Getting started

First, let's simply get a Window that displays Hello World. To do that, we'll use the following code:

```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Label;
import javafx.scene.layout.FlowPane;
import javafx.scene.layout.Pane;
import javafx.stage.Stage;

public class HelloWorld extends Application {
    @Override
    public void start(Stage primaryStage) throws Exception {
        //create a Label to say Hello World
        Label helloLabel = new Label("Hello World!");

        //create a pane to act as the root Pane
        Pane root = new FlowPane();

        //add the Label to the root Pane
        root.getChildren().add(helloLabel);

        //Create a Scene containing our root Pane
        Scene scene = new Scene(root);

        //Put our scene on stage
        primaryStage.setScene(scene);

        //display the stage
        primaryStage.show();
    }
}
```

If we run this code, you will get the following window.

![img.png](../img/hello_world_1.png)

Not exactly the most exciting thing, but this is just a first step.

Let's walk through what we did do here. First, we created a Label to display the text HelloWorld. That label is created in the line:

```java
    Label helloLabel = new Label("Hello World!");
```

The Label control has a constructor that takes in a String you wish to display. In this case, we are displaying the String `"Hello World!"`

We add this Label to our application by first creating the root pane:

```java
    Pane root = new FlowPane();
```

And then adding the label to that Pane's child Nodes.

```java
    root.getChildren().add(helloLabel);
```

Next, we create our Scene using a constructor that lets us define that scene's root pane (in this case, `root`, the FlowPane we just created and populated):

```java
    Scene scene = new Scene(root);
```

Set the `primaryStage` to show the scene:

```java
    primaryStage.setScene(scene);
```

And the finally show the Stage (application window)

```java
    primaryStage.show();
```

## But Wait... Where is `main`?

You might notice there is no `main` function in this app. That's because the main is *inherited* from the imported `Application` class. In JavaFX, the `Application` class will automatically call the function `start` when the app is initialized. The Application class actually creates the Stage object (that is, the Window) and passes it to the `start` function. This is an example of the framework doing a lot of the lower-level stuff for you, so you can focus simply on building your application.

In short, you don't define `main`, you can simply start with...well...`start`. However, you should **never directly call the `start` method yourself!** You should always allow the framework to handle it.

## Next Module - Buttons and Events

In the next module, we will build on this application by adding a button that causes our text to change.