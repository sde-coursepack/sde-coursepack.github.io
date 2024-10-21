---
Title: JSON Object Mapping
---

# JSON Object Mapping
In the last couple modules, we manually parsed json to Java Objects and wrote java objects back to json. Doing so manually is fine, but there are some built in tools to help make this more efficient.

* TOC
{:toc}

# Serialize and Deserialize

Two words worth introducing are **serialize** and **deserialize**. For the sake of this module, we will refer to **serialize** as meaning "Going from Java Object to JSON" and **deserialize** as "Going from JSON to Java Object"

There is a clear advantage to being able to go back and forth, and an ObjectMapper can make this process simpler.

## QuizQuestion

For now, let's consider using QuizQuestion only from our last two modules. For context, here is the class, as we left it (minus the `toString` and `toJSON` method)

```java
public class QuizQuestion {
    private String question;
    private List<String> options;
    private String answer;

    public QuizQuestion(String question, List<String> options, String answer) {
        if (!options.contains(answer)) {
            throw new IllegalArgumentException("Error: answer provided not in choices: " + answer + "\n" +
                    "\tchoices: " + options);
        }
        this.question = question;
        this.options = options;
        this.answer = answer;
    }

    public String getQuestion() {
        return question;
    }

    public List<String> getOptions() {
        return options;
    }

    public String getAnswer() {
        return answer;
    }
}
```

When we consider our code for mapping to and from JSON in the last two modules, it's really trivial boilerplate code. Are there tools for doing this faster? The answer is yes!

For instance, let's consider the following:

```java
    public static void main(String[] args) {
        QuizQuestion quizQuestion = new QuizQuestion(
                "Which one is a correct team name in NBA?",
                List.of(
                        "New York Bulls",
                        "Los Angeles Kings",
                        "Golden State Warriros",
                        "Houston Rockets"
                ),
                "Houston Rockets");

        JSONObject jsonObject = new JSONObject(quizQuestion);
        System.out.println(jsonObject.toString(2));
    }
```

You'll notice this code **does not** use the toJSON() method we wrote in the last module. Instead, we are using the constructor of `JSONObject` directly as the "mapper".

The code above runs, and prints:

```json
{
  "answer": "Houston Rockets",
  "question": "Which one is a correct team name in NBA?",
  "options": [
    "New York Bulls",
    "Los Angeles Kings",
    "Golden State Warriros",
    "Houston Rockets"
  ]
}
```

By default, whenever we put an object in a json constructor, it will simply create a map of the objects **field names** to **values** above. How does it detect field names? By looking for **getter methods**. Specifically, methods whose names are getX() with no parameters are assumed to be simple getters for fields.

However, this can result in unintended behavior. For example, lets now take the `Quiz` object we made in the parsing JSONObject unit, and try to turn it into a JSON Object:

```java
public class QuizParser {
    public static void main(String[] args) throws IOException {
        QuizParser parser = new QuizParser();
        JSONObject rootJSON = parser.getRootJsonObject("quiz.json");
        var quiz = parser.getQuizFromJSON(rootJSON);

        System.out.println(new JSONObject(quiz).toString(2));
    }
    // other methods hidden
}
```

If we run this code, we get:

```json
{"numberOfQuestions": 3}
```

...and that's it! But wait! Where are our categories and questions? Well, for that, we have to look at the methods we gave the Quiz class. For space, I removed all method implementations and simply show their names. I also removed the `toJSON()` method from the last module.

```java
public class Quiz {
    private Map<String,QuizCategory> quiz;

    public Quiz() { }
    public void addCategory(QuizCategory category) { }
    private boolean quizHasCategory(String category) { }
    public int getnumberOfQuestions() { }
    public QuizQuestion getRandomQuestion(String category) { }
}
```

If you look, you'll notice that only **one** method fits the pattern of a method named "get_______" with no parameters: `public int getnumberOfQuestions()`. While we also have `QuizQuestion getRandomQuestion(String category)`, this method requires a parameter, and so the built-in object mapper ignores it.

**So, it's not actually "fields" that the JSONObject constructor looks for, it's getter methods with zero paramaters**

However, let's add the following method:

```java
public class Quiz {
    private Map<String,QuizCategory> quiz;

    public Map<String, QuizCategory> getQuiz() {
        return Collections.unmodifiableMap(quiz);
    }
```

By adding this method, and running the same code:

```java
public class QuizParser {
    public static void main(String[] args) throws IOException {
        QuizParser parser = new QuizParser();
        JSONObject rootJSON = parser.getRootJsonObject("quiz.json");
        var quiz = parser.getQuizFromJSON(rootJSON);

        System.out.println(new JSONObject(quiz).toString(2));
    }
    // other methods hidden
}
```

We now get the following:

```json
{
  "numberOfQuestions": 3,
  "quiz": {
    "sports": {
      "randomQuestion": {
        "answer": "Houston Rockets",
        "question": "Which one is a correct team name in NBA?",
        "options": [
          "New York Bulls",
          "Los Angeles Kings",
          "Golden State Warriros",
          "Houston Rockets"
        ]
      },
      "numberOfQuestions": 1,
      "categoryName": "sports"
    },
    "math": {
      "randomQuestion": {
        "answer": "4",
        "question": "12 - 8 = ?",
        "options": [
          "1",
          "2",
          "3",
          "4"
        ]
      },
      "numberOfQuestions": 2,
      "categoryName": "math"
    }
  }
}
```

This looks the same as our starting JSON. However, you'll notice there are some extra pieces of data, namely we still just "numberOfQuestions", both for `Quiz` and `QuizCategory`. Additionally, each quiz also lists its own category when it previously didn't.

By default, you can't really override this behavior without writing your own "toJSON()" method like we did in the last unit and using that.

However, using the popular library Jackson, we can gain access to more control:

# Jackson library

The [Jackson databind library](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind) is useful here, and can help us out. 

## Java to JSON

When going from Java objects in memory to JSON, we can use Jackson's `ObjectMapper`, as shown below:

```java
    public static void main(String[] args) throws IOException{
        // parse existing quiz file
        QuizParser parser = new QuizParser();
        JSONObject rootJSON = parser.getRootJsonObject("quiz.json");
        var quiz = parser.getQuizFromJSON(rootJSON);

        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println(objectMapper
                        .writerWithDefaultPrettyPrinter() // print with line breaks/tabs
                        .writeValueAsString(quiz) // print the contents of quiz
        );
    }
```

Be aware that `objectMapper.writeValueAsString` **throws** `JsonProcessingException`, which is a type of IOException. As such, when using this mapper, you will need to consider what to do if the JSON format is bad. Here, I just declare main `throws IOException`, so I don't need a try-catch. However, if I can meaningfully handle the error in the context of a program I'm writing, I should.

If we run this code, we get similar to what we've had:

```java
{
  "quiz" : {
    "sports" : {
      "categoryName" : "sports",
      "randomQuestion" : {
        "question" : "Which one is a correct team name in NBA?",
        "options" : [ "New York Bulls", "Los Angeles Kings", "Golden State Warriros", "Houston Rockets" ],
        "answer" : "Houston Rockets"
      },
      "numberOfQuestions" : 1
    },
    "math" : {
      "categoryName" : "math",
      "randomQuestion" : {
        "question" : "12 - 8 = ?",
        "options" : [ "1", "2", "3", "4" ],
        "answer" : "4"
      },
      "numberOfQuestions" : 2
    }
  },
  "numberOfQuestions" : 3
}
```

Note that in the above code, all `.writerWithDefaultPrettyPrinter()` is just add tabs and linebreaks. If we remove it, but leave `writeValueAsString(quiz)`, we get:

```java
{"quiz":{"sports":{"categoryName":"sports","randomQuestion":{"question":"Which one is a correct team name in NBA?","options":["New York Bulls","Los Angeles Kings","Golden State Warriros","Houston Rockets"],"answer":"Houston Rockets"},"numberOfQuestions":1},"math":{"categoryName":"math","randomQuestion":{"question":"12 - 8 = ?","options":["1","2","3","4"],"answer":"4"},"numberOfQuestions":2}},"numberOfQuestions":3}
```

All of that being only one line being hard to read for a human, but it contains the same content and format.

The problem here is we still have `numberOfQuestions` present when we don't want it. But we can fix that with the **annotation** `@JsonIgnore`

```java
public class Quiz {
    private Map<String, QuizCategory> quiz;

    public Quiz() {
        quiz = new HashMap<>();
    }

    public Map<String, QuizCategory> getQuiz() {
        return Collections.unmodifiableMap(quiz);
    }

    @JsonIgnore
    public int getNumberOfQuestions() {
        return quiz.values().stream()
                .mapToInt(category -> category.getNumberOfQuestions())
                .sum();
    }
    
    // other methods hidden
}
```

Add the same annotation to `getNumberOfQuestions` in `QuizCategory.java`, and then re-run the code above and we get:

```json
{
  "quiz" : {
    "sports" : {
      "categoryName" : "sports",
      "randomQuestion" : {
        "question" : "Which one is a correct team name in NBA?",
        "options" : [ "New York Bulls", "Los Angeles Kings", "Golden State Warriros", "Houston Rockets" ],
        "answer" : "Houston Rockets"
      }
    },
    "math" : {
      "categoryName" : "math",
      "randomQuestion" : {
        "question" : "5 + 7 = ?",
        "options" : [ "10", "11", "12", "13" ],
        "answer" : "12"
      }
    }
  }
}
```

You'll see we no longer have our `numberofQuestions` values in the JSON. It's important to remove these because these are **derived** values. That is, they aren't **part** of the data, merely a summary of other data.

The reason this is important is that when we want to reverse the process (go from JSON to Java), these derived values become a problem.

## JSON to Java

Before looking at Quiz, let's look at a simpler example of how to go "back and forth" between Java and JSON. Specifically, QuizQuestion:

```java
public class QuizQuestion {
    private String question;
    private List<String> options;
    private String answer;

    public QuizQuestion(String question, List<String> options, String answer) {
        if (!options.contains(answer)) {
            throw new IllegalArgumentException("Error: answer provided not in choices: " + answer + "\n" +
                    "\tchoices: " + options);
        }
        this.question = question;
        this.options = options;
        this.answer = answer;
    }

    public String getQuestion() {
        return question;
    }

    public List<String> getOptions() {
        return options;
    }

    public String getAnswer() {
        return answer;
    }

    public String toString() {
        return "QuizQuestion{" +
                "question='" + question + '\'' +
                ", options=" + options +
                ", answer='" + answer + '\'' +
                '}';
    }
}
```

Specifically, if we made a quiz question, then use our ObjectMapper, we can go from Java to JSON easily:

```java
public class MappingDemo {
    public static void main(String[] args) throws IOException {
        QuizQuestion quizQuestion = new QuizQuestion(
                "Which one is a correct team name in NBA?",
                List.of(
                        "New York Bulls",
                        "Los Angeles Kings",
                        "Golden State Warriros",
                        "Houston Rockets"
                ),
                "Houston Rockets");

        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println(objectMapper.writeValueAsString(quizQuestion));
    }
}
```

Which gives us:

```json
{"question":"Which one is a correct team name in NBA?","options":["New York Bulls","Los Angeles Kings","Golden State Warriros","Houston Rockets"],"answer":"Houston Rockets"}
```

But now, let's try to reverse that process. That is, go from our json *to* our QuizQuestion using `objectMapper.readValue` to read into an instance of `QuizQuestion.class`

```java
public class MappingDemo {
    public static void main(String[] args) throws IOException {
        QuizQuestion quizQuestion = new QuizQuestion(
                "Which one is a correct team name in NBA?",
                List.of(
                        "New York Bulls",
                        "Los Angeles Kings",
                        "Golden State Warriros",
                        "Houston Rockets"
                ),
                "Houston Rockets");

        ObjectMapper objectMapper = new ObjectMapper();
        String jsonText = objectMapper.writeValueAsString(quizQuestion);
        QuizQuestion readQuestion = objectMapper.readValue(jsonText, QuizQuestion.class);
    }
}
```

We get an exception:

```java
Exception in thread "main" com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `QuizQuestion` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
```

Let's explain *why* `QuizQuestion.java` can't be built. Specifically, in the same way **ObjectMapper** uses `get` methods to retrieve information about the object when going from **Java to JSON**, it will expect `set` methods for the same data when going from **JSON to Java**.

As such, we need to add two things to **QuizQuestion** for us to be able to use the simple ObjectMapper.

1) setters for each of our 3 properties  
2) A zero argument constructor for QuizQuestion

Let's add that to QuizQuestion

```java
public class QuizQuestion {
    private String question;
    private List<String> options;
    private String answer;

    public QuizQuestion(String question, List<String> options, String answer) {
        if (!options.contains(answer)) {
            throw new IllegalArgumentException("Error: answer provided not in choices: " + answer + "\n" +
                    "\tchoices: " + options);
        }
        this.question = question;
        this.options = options;
        this.answer = answer;
    }
    
    public QuizQuestion() {}

    public String getQuestion() {
        return question;
    }
    
    public void setQuestion(String question) {
        this.question = question;
    }
    
    public List<String> getOptions() {
        return options;
    }
    
    public void setOptions(List<String> options) {
        this.options = options;
    }

    public String getAnswer() {
        return answer;
    }
    
    public void setAnswer(String answer) {
        this.answer = answer;
    }

    public String toString() {
        return "QuizQuestion{" +
                "question='" + question + '\'' +
                ", options=" + options +
                ", answer='" + answer + '\'' +
                '}';
    }
}
```

Now, when we run our code to go from JSON to Java, and print the `QuizQuestion` object (which uses the `toString()` method), we don't crash! And we get:

```text
QuizQuestion{question='Which one is a correct team name in NBA?', options=[New York Bulls, Los Angeles Kings, Golden State Warriros, Houston Rockets], answer='Houston Rockets'}
```

So it works! We could already go from Java to JSON, and now we can go from JSON back to Java!

## What this allows

We now have an easy way to send data to potentially physically separate systems. While we can't send Java Objects over a network directly, we can **serialize** them into some kind of byte-string (in this case, effectively text).

Now, we can build out something like a client-server system, where the server has a large database of questions, and the client can request to receive a question. The server-side can take the question, convert it to a JSON String, send it over the network to the client, and the client can reconstruct that object as a Quiz question!

## A major downside

Notice that, by adding a zero argument constructor and setters, we have potentially introduced some problems:

1) Because the zero-argument constructor exists, people can create a QuizQuestion **without** using the three-argument constructor.  
2) This means clients can "bypass" the constructor check that ensures that the answer is in the list of choices  
3) We would now need to add this logic to our setter...except...
4) What if `setAnswer()` is called by the object mapper before `setChoices()`, meaning the object mapper could end up neccesarily needing to set an answer to an invalid value **before** it becomes valid when choices are set  
5) We now can have a `QuizQuestion` object with (at least temporarily) an invalid state, with `null` fields, or fields that violate the intent of the object  

These problems mean we've given up the assurance of our `QuizQuestion` always being a valid state in exchange for convenience.

## Avoiding this downside

Let's go back to the class we had. We'll remove the setters and the zero-argument constructor, but we'll add some `@JsonProperty` annotations to our 3-argument constructor.

```java
public class QuizQuestion {
    private String question;
    private List<String> options;
    private String answer;

    public QuizQuestion(@JsonProperty("question") String question,
            @JsonProperty("options") List<String> options,
            @JsonProperty("answer")  String answer) {
        if (!options.contains(answer)) {
            throw new IllegalArgumentException("Error: answer provided not in choices: " + answer + "\n" +
                    "\tchoices: " + options);
        }
        this.question = question;
        this.options = options;
        this.answer = answer;
    }

    public String getQuestion() {
        return question;
    }

    public List<String> getOptions() {
        return options;
    }

    public String getAnswer() {
        return answer;
    }

    @Override
    public String toString() {
        return "QuizQuestion{" +
                "question='" + question + '\'' +
                ", options=" + options +
                ", answer='" + answer + '\'' +
                '}';
    }
}
```

Remember, our `QuizQuestion` json representation has 3 properties, "question", "options", and "answer". As we mentioned, by default, ObjectMapper will look for a zero-argument constructor and setter methods for those properties. However, when we use @JSONProperty in the constructor arguments above, ObjectMapper will use those instead.

As such, if we check our mapper usage one more time:

```java
public class MappingDemo {
    public static void main(String[] args) throws IOException {
        QuizQuestion quizQuestion = new QuizQuestion(
                "Which one is a correct team name in NBA?",
                List.of(
                        "New York Bulls",
                        "Los Angeles Kings",
                        "Golden State Warriros",
                        "Houston Rockets"
                ),
                "Houston Rockets");

        ObjectMapper objectMapper = new ObjectMapper();
        String jsonText = objectMapper.writeValueAsString(quizQuestion);
        QuizQuestion readQuestion = objectMapper.readValue(jsonText, QuizQuestion.class);
        System.out.println(readQuestion);
    }
}
```

We can see that it works! The console prints!

```text
QuizQuestion{question='Which one is a correct team name in NBA?', options=[New York Bulls, Los Angeles Kings, Golden State Warriros, Houston Rockets], answer='Houston Rockets'}
```

The advantage is that we still get the convenience of easily adding **serialization** and **deserialization**, but without risking losing control of the validity of our data.

## Next Steps

From there, make Quiz and QuizCategory serializable and deserializable, something we manually did with `QuizParser` (deserialize) and our `toJSON()` methods in the previous unit (serialize) using Jackson's ObjectMapper is going to require the use of Custom Serializers and Deserializers. Given that this goes beyond anything you'll be asked to do in the course, I will direct you to this [Jenkov article on the topic.](https://jenkov.com/tutorials/java-json/jackson-objectmapper.html)

