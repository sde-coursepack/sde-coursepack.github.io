---
Title: Simple JavaFX App
---

# Simple JavaFX App - Square Root

In this module, we will build a simple GUI application that calculates the square root of a number using the JavaFX Framework.

## Code

Note that I include import statements and comments in the code below for illustration purposes. Generally, I would break up the `start` method into several smaller methods, but for this module I want to walk through each step in order clearly.

```java
import javafx.application.Application;
import javafx.event.ActionEvent;
import javafx.event.EventHandler;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.control.TextField;
import javafx.scene.layout.FlowPane;
import javafx.stage.Stage;

public class SqrtCalculator extends Application {
    //A label field defined at the class level so the action listener can access it
    TextField input;
    Label result;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage stage) {
        stage.setTitle("Square Root Calculator");
        //Create the labels
        Label text = new Label("Enter a number:");

        //create TextField
        input = new TextField();

        //Create our placeholder - initially blank, but the button listener
        //will update it
        result = new Label("");
        result.setVisible(false);

        //Create the button
        Button button = new Button();
        button.setText("Calculate"); //sets the button's text
        button.setOnAction(new ButtonHandler()); //defines what happens when the button is clicked
        //in this case, we are calling "handle" in our
        //ButtonHandler class

        FlowPane root = new FlowPane();

        //add our elements to the pane in the order we want them displayed
        root.getChildren().addAll(text, input, button, result);
        
        //create a scene from the Pane and make it active
        Scene scene = new Scene(root, 300, 200);
        stage.setScene(scene);
        stage.show();
    }

    /**
     * Event Handler class
     * This inner class defines an action to perform when the button is hit
     */
    private class ButtonHandler implements EventHandler<ActionEvent> {

        @Override
        public void handle(ActionEvent actionEvent) {
            result.setVisible(true);
            try {
                double x = Double.parseDouble(input.getText());
                double sqrt = Math.sqrt(x);
                result.setText("Answer: " + sqrt);
            } catch (NumberFormatException e) {
                result.setText("Invalid Entry");
            }
        }
    }
}
```