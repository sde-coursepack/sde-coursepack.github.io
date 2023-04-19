---
Title: Hello World GUI
---

# Hello World GUI

In this module, we will iteratively create a very simple application, introducing JavaFX basics one at a time.

Note that in this unit I will paste code containing a large number comments. Generally, this is bad style, and I am not encouraging students to duplicate it. However, because JavaFX is likely a first experience for my studnets with GUI development, I want to ensure each step of the way that students understand why a particular line of code was written.

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
## Final Code

