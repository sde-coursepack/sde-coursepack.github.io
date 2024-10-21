---
Title: Building JSON Objects
---

# Building JSON Objects

In the last module we looked at how to use the `org.json` library to parse JSON data into a Java `Quiz` object. In this unit, we will do the opposite: take our existing quiz and dump it into a `JSONObject`

* TOC
{:toc}

# QuizQuestion to JSONObject

First, let's consider how to store our `QuizQuestion` as a `JSONObject`. Specifically, our `QuizQuestion` has the following fields:

```java
public class QuizQuestion {
    private String question;
    private List<String> options;
    private String answer;

    // methods and constructors hidden
}
```

To write our object to JSON, we need to store this fields in a `JSONObject`. In our last part, we only "read" from a `JSONObject`. Here we will be "writing" to a JSONObject. The follow `QuizQuestion` method does just that:

```java
    public JSONObject toJSON() {
        JSONObject questionJSON = new JSONObject();

        // store question and answer
        questionJSON.put("question", question);
        questionJSON.put("answer", answer);
        
        // store options as JSONArray
        JSONArray options = new JSONArray();
        for (String option : getOptions()) {
            options.put(option);
        }
        
        // store options array in JSON
        questionJSON.put("options", options);

        return questionJSON;
    }
```

Now, if I take the quizQuestion using the following question:
```text
Question: 12 - 8 = ?
Choices: 
    1
    2
    3
    4
Answer: 4
```

and I call toJSON() on it and print, I get:

```
{"question":"5 + 7 = ?","answer":"12","options":["10","11","12","13"]}
```

From there, we can add linebreaks and tabs when printing a JSONObject using the `.toString(int indentFactor)` method, so if I print `questionObject.toString(2)` I get:

```json
{
  "question": "12 - 8 = ?",
  "answer": "4",
  "options": [
    "1",
    "2",
    "3",
    "4"
  ]
}
```

Which you'll notice is the same format of JSON we took in during the last module!

## JSONArray of a List
We can simply our `toJSON()` method above a bit, using just:

```java
    public JSONObject toJSON() {
        JSONObject questionJSON = new JSONObject();
        questionJSON.put("question", question);
        questionJSON.put("answer", answer);
        questionJSON.put("options", options);
        return questionJSON;
    }
```

The reason is that `org.json` is smart enough to now that a `List<String>` can be stored as a `JSONArray` of Strings.

## QuizCategory to JSONArray

From there, we can similarly write a `toJSON()` method for `QuizCategory`, which as the following fields:

```java
public class QuizCategory {
    private String categoryName;
    private List<QuizQuestion> questions;
    
    // methods and constructor hidden
}
```

Specifically here, to match our input format, we simply want to return a `JSONArray` of our questions, and so, we can do:

```java
    public JSONArray toJSON() {
    List<JSONObject> questionObjects = questions.stream()
            .map(q -> q.toJSON())
            .toList();
    return new JSONArray(questionObjects);
}
```

We're not including the category name, because the `Quiz` has the list of category names directly, so we will handle that there.

## Quiz to JSON Object

Here is the reminder of our Quiz.java field:

```java
public class Quiz {
    private Map<String,QuizCategory> quiz;
```

From there, we can get our entire Quiz as JSON using, in Quiz.java

```java
    public JSONObject toJSON() {
        JSONObject quizJSON = new JSONObject();
        for (String categoryName: quiz.keySet()) {
            QuizCategory quizCategory = quiz.get(categoryName);
            quizJSON.put(categoryName, quizCategory.toJSON());
        }
        return quizJSON;
    }
```

If we now run `toJSON()` on a quiz built from our starting json in the last module, and print that JSONObject with Indent Factor 2, we get:

```json
{
  "sports": [{
    "question": "Which one is a correct team name in NBA?",
    "answer": "Houston Rockets",
    "options": [
      "New York Bulls",
      "Los Angeles Kings",
      "Golden State Warriros",
      "Houston Rockets"
    ]
  }],
  "math": [
    {
      "question": "5 + 7 = ?",
      "answer": "12",
      "options": [
        "10",
        "11",
        "12",
        "13"
      ]
    },
    {
      "question": "12 - 8 = ?",
      "answer": "4",
      "options": [
        "1",
        "2",
        "3",
        "4"
      ]
    }
  ]
}
```

Which is **almost** the same as our starting JSON. However, specifically, we want this entire object mapped to the key "quiz" in some outer JSONObject, and so, we change our method:

```java
public JSONObject toJSON() {
    JSONObject quizJSON = new JSONObject();
    for (String categoryName: quiz.keySet()) {
        QuizCategory quizCategory = quiz.get(categoryName);
        quizJSON.put(categoryName, quizCategory.toJSON());
    }

    JSONObject root = new JSONObject();
    root.put("quiz", quizJSON);

    return root;
}
```

And now, we'll get:

```json
{"quiz": {
  "sports": [{
    "question": "Which one is a correct team name in NBA?",
    "answer": "Houston Rockets",
    "options": [
      "New York Bulls",
      "Los Angeles Kings",
      "Golden State Warriros",
      "Houston Rockets"
    ]
  }],
  "math": [
    {
      "question": "5 + 7 = ?",
      "answer": "12",
      "options": [
        "10",
        "11",
        "12",
        "13"
      ]
    },
    {
      "question": "12 - 8 = ?",
      "answer": "4",
      "options": [
        "1",
        "2",
        "3",
        "4"
      ]
    }
  ]
}}
```

And you'll notice this matches our input exactly! Sure, there may be some different line breaks, and map elements in different orders (neither of which matters). But otherwise, it is the same content!