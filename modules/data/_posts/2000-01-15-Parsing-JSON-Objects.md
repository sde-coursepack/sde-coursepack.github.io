---
Title: Parsing JSON in Java
---

# Parsing JSON in Java

In this module, we will look at how to manually parse a JSON Object using the `org.json` library in Java

* TOC
{:toc}


# Org.JSON

The [org.json library](https://mvnrepository.com/artifact/org.json/json) is a library for using JSON in the Java programming language. The following datatypes are important here:
## [JSONObject](https://stleary.github.io/JSON-java/org/json/JSONObject.html) 
This represents a JSON object (bounded by curly braces). For example: 

```json
{
    "fruit": "Apple",
    "size": "Large",
    "color": "Red"
}
```
The above json objects has the **keys** "fruit", "size", and "color", and the values (respectively in the same order as keys): "Apple", "Large", "Red". The key must always be a "String" datatype, while the value can be a String, a number (either integer or floating point), a boolean, a JSONObject, or a JSONArray. It functions much like a HashMap in terms of usage, but the values can be different types.

JSONObject in Java has several methods we can use to read JSON:

```
    Object jsonObject.get(String key)
```

This is the most basic method, which returns the Java Object class. However, most frequently, we will want to use something like:

```java 
    int getInt(String key)
    double getDouble(String key)
    boolean getBoolean(String key)
    String getString(String key)
    JSONObject getJSONObject(String key)
    JSONArray getJSONArray(String key)
```

There is also `bolean hasKey(String key)` to check if a given key exists, as well as methods like `int optInt(String key)` which return an `Optional<Integer>`, with the value being `Optional.empty()` if the key doesn't exist.

## [JSONArray](https://stleary.github.io/JSON-java/org/json/JSONArray.html)

JSONArray is when a collection of items in JSON bounded by square braces, such as:

```json
"numbers": [1, 2, 3, 4]
```
Here, the above JSONArray is a JSON Array of integers. Much like JSONObject, we can access values, but instead of using a `String key`, since this is an ordered array, we use `int index`.

```java
    int getInt(int index)
    double getDouble(int index)
    boolean getBoolean(int index)
    String getString(int index)
    JSONObject getJSONObject(int index)
    JSONArray getJSONArray(int index)
```

We also have `int length()` to get the size of the JSONArray

# QUIZ JSON Example

Consider the following example json structure for modelling a quiz.

```json
{
  "quiz": {
    "sport": [
      {
        "question": "Which one is a correct team name in NBA?",
        "options": [
          "New York Bulls",
          "Los Angeles Kings",
          "Golden State Warriros",
          "Houston Rockets"
        ],
        "answer": "Houston Rockets"
      }
    ],
    "maths": [
      {
        "question": "5 + 7 = ?",
        "options": [
          "10",
          "11",
          "12",
          "13"
        ],
        "answer": "12"
      },
      {
        "question": "12 - 8 = ?",
        "options": [
          "1",
          "2",
          "3",
          "4"
        ],
        "answer": "4"
      }
    ]
  }
}
```

Let's parse this JSON string into a series of objects that will let us build a quiz from this format.

# Our classes

First, let's define our classes that make up a Quiz.

## Quiz.java

```java
public class Quiz {
    private Map<String,QuizCategory> quiz;

    public Quiz() {
        quiz = new HashMap<>();
    }

    public void addCategory(QuizCategory category) {
        String categoryName = category.getCategoryName();
        if (quizHasCategory(categoryName)) {
            throw new IllegalArgumentException("Category already exists");
        }
        quiz.put(categoryName, category);
    }

    private boolean quizHasCategory(String category) {
        return quiz.containsKey(category);
    }

    public int getNumQuestions() {
        return quiz.values().stream()
                .mapToInt(category -> category.getNumberOfQuestions())
                .sum();
    }

    public QuizQuestion getRandomQuestion(String category) {
        if (!quizHasCategory(category)) {
            throw new IllegalArgumentException("Category doesn't exist");
        }
        QuizCategory quizCategory = quiz.get(category);
        return quizCategory.getRandomQuestion();
    }
}
```

Here, our `Quiz` is made up of several `QuizCategory` objects, each containing 1 or more questions.

This is the overarching class for our quiz. Users can ask for a question from a particular category, and a random question from that category will be selected.

## QuizCategory

```java
public class QuizCategory {
    private String categoryName;
    private List<QuizQuestion> questions;

    public QuizCategory(String categoryName) {
        this.categoryName = categoryName;
        this.questions = new ArrayList<>();
    }

    public String getCategoryName() {
        return categoryName;
    }

    public void addQuestion(QuizQuestion question) {
        questions.add(question);
    }

    public int getNumberOfQuestions() {
        return questions.size();
    }

    public QuizQuestion getRandomQuestion() {
        int index = (int) (Math.random() * questions.size());
        return questions.get(index);
    }
}
```

A `QuizCategory` has a name (such as "sports" or "math"), and a list of `QuizQuestion`s. Questions can be added via `addQuestion`, and we can get a random question from the list using `getRandomQuestion`

## QuizQuestion

This represents on question on a quiz, including the answer:

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
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append(getQuestion()).append("\n");

        for (String option : getOptions()) {
            stringBuilder.append("\t").append(option).append("\n");
        }
        stringBuilder.append("Answer: ").append(getAnswer());
        return stringBuilder.toString();
    }
}
```

# Parsing a JSON String

Now we will show how to use the [org.json library](https://mvnrepository.com/artifact/org.json/json) to parse our JSON string into the objects above:

## Parsing QuizQuestion json

Now we are ready to start parsing our .json file above.

First, let's think about parsing an individual `QuizQuestion`. For instance, a single `QuizQuestion` can be represented by the following JSONObject:

```json
{
  "question": "Which one is a correct team name in NBA?", 
  "options": [
    "New York Bulls", 
    "Los Angeles Kings", 
    "Golden State Warriros", 
    "Houston Rockets"
  ], 
  "answer": "Houston Rockets"
}
```

Our JSONObject has three **keys**, "question", "options", and "answer". "question" and "answer" both indicate Strings, while "option" represents a **JSONArray** of Strings.

Looking at the **constructor** for `QuizQuestion`:

```java
    public QuizQuestion(String question, List<String> options, String answer) {
        if (!options.contains(answer)) {
            throw new IllegalArgumentException("Error: answer provided not in choices");
        }
        this.question = question;
        this.options = options;
        this.answer = answer;
    }
```

we need the "question" and "answer" as Strings (easy enough), and the "options" as a List<String>. To parse this JSON, let's consider the following method in `QuizParser.java`

```java
    private QuizQuestion jsonToQuizQuestion(JSONObject jsonObject) {
       String question = jsonObject.getString("question");
       String answer = jsonObject.getString("answer");

        JSONArray choiceArray = jsonObject.getJSONArray("options");
        List<String> options = new ArrayList<>();
        for (int i = 0; i < choiceArray.length(); i++) {
            options.add(choiceArray.getString(i));
        }

        return new QuizQuestion(answer, options, question);
    }
```

Looking at our question jsonObject (with linebreaks removed to fit into less space):

```json
{
  "question": "Which one is a correct team name in NBA?", 
  "options": ["New York Bulls", "Los Angeles Kings", "Golden State Warriros", "Houston Rockets"], 
  "answer": "Houston Rockets"
}
```

You can see we get "question" and "answer" using the appropriate key and "getString()" method:

```java
    String question = jsonObject.getString("question");
    String answer = jsonObject.getString("answer");
```

We can then get the JSONArray "options" using:

```java
    JSONArray choiceArray = jsonObject.getJSONArray("options");
```

From there, we can use a simple accumulator pattern to build up a List<String> of our options.

```java
    List<String> options = new ArrayList<>();
    for (int i = 0; i < choiceArray.length(); i++) {
        options.add(choiceArray.getString(i));
    }
```

Now, JSONArray does have a .toList() method, so you might **think** we could simply do:

```java
    List<String> options = choiceArray.toList();
```

However, this line won't compile. And the reason is that `JSONArray`'s `toList()` method returns `List<Object>`, not `List<String>`. Yes, we can look at the json string and know that all the contents of the array are Strings, but Java can't know that at compile time.

So, with our method in place, we can now convert a question object represented in JSON to a `QuizQuestion` object.

## Parsing QuizCategory

Now that we have a method that can parse a `QuizQuestion`, let's look at the structure of our categories. In the below, I replaced all questions with `{ question }`, since we can already parse that and don't need to focus on it.

```json
{
  "quiz": {
    "sports": [
      { question }
    ],
    "math": [
      { question },
      { question }
    ]
  }
}
```

Each `QuizCategory` is a `JSONArray` of `QuizQuestion` objects. And so, to parse a particular category, such as "math", we can use the following to get the category names:

```json
  var quizRoot = root.getJSONObject("quiz");
  var categorySet = quizRoot.keySet();
```

In the above `categorySet` is the `Set<String>` of keys in the quizRoot object (in our example a set of "sports" and "math"). Note that our "root" object is the overall JSONObject bounded by the starting and ending curly braces. This JSONObject only has **one** key, "quiz". So we "drill" down to the JSONObject mapped to "quiz" to get `quizRoot`, which is:

```json
{
  "sports": [
    { question }
  ],
  "math": [
    { question },
    { question }
  ]
}
```

Now that we have our `categorySet`, we can create each category by:

1) Access the JSONArray mapped to each categories **name** (example, "sports" and "math")  
2) Parse the JSONArray of questions into a `List<QuizQuestion>` for the category.

You can see that in the following code:

```java
private Quiz getQuizFromJSON(JSONObject root) {
        var quiz = new Quiz();

        var quizRoot = root.getJSONObject("quiz");
        var categorySet = quizRoot.keySet();

        for (String quizCategoryName : categorySet) {
            JSONArray questionArray = quizRoot.getJSONArray(quizCategoryName);
            QuizCategory quizCategory = getQuizCategory(quizCategoryName, questionArray);
            quiz.addCategory(quizCategory);
        }
        return quiz;
    }

    private QuizCategory getQuizCategory(String category, JSONArray questionArray) {
        QuizCategory quizCategory = new QuizCategory(category);
        for (Object questionObject : questionArray) {
            JSONObject questionJSON = (JSONObject) questionObject;
            QuizQuestion question = jsonToQuizQuestion(questionJSON);
            quizCategory.addQuestion(question);
        }
        return quizCategory;
    }
```

Here, we are using our previous made: `jsonToQuizQuestion(questionJSON);` above.

We now have a means or parsing a JSON Object into a `Quiz` object in our code. Using that, we can write a `main` method like:

```java
public class QuizParser {
    public static void main(String[] args) throws IOException{
        QuizParser parser = new QuizParser();
        JSONObject rootJSON = parser.getRootJsonObject("quiz.json");
        var quiz = parser.getQuizFromJSON(rootJSON);

        System.out.println("Quiz size: " + quiz.getNumQuestions());

        QuizQuestion quizQuestion = quiz.getRandomQuestion("math");
        System.out.println(quizQuestion);
    }

    private JSONObject getRootJsonObject(String quizFileName) throws IOException {
        var fileText = getFileContents(quizFileName);
        JSONObject root = new JSONObject(fileText);
        return root;
    }

    private String getFileContents(String filename) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        BufferedReader bufferedReader = new BufferedReader(new FileReader(filename));
        bufferedReader.lines().forEach(stringBuilder::append);
        return stringBuilder.toString();
    }
    
    // other methods we wrote here
}
```

Now, using the above, we may get the following printout:

```text
Quiz size: 3
Math Question!
12 - 8 = ?
	1
	2
	3
	4
Answer: 4
```

From here, we have the core structure of our `Quiz`, and a way to parse quizzes. So now, we can add questions to our quiz simply by adding questions to our JSON file!

## Final Quiz Parser code:

For convenience, the entirety of the QuizParse.java class is added below:

```java
public class QuizParser {
    public static void main(String[] args) throws IOException{
        QuizParser parser = new QuizParser();
        JSONObject rootJSON = parser.getRootJsonObject("quiz.json");
        var quiz = parser.getQuizFromJSON(rootJSON);

        System.out.println("Quiz size: " + quiz.getNumQuestions());

        QuizQuestion quizQuestion = quiz.getRandomQuestion("math");
        System.out.println(quizQuestion);
    }

    private JSONObject getRootJsonObject(String quizFileName) throws IOException {
        var fileText = getFileContents(quizFileName);
        JSONObject root = new JSONObject(fileText);
        return root;
    }

    private String getFileContents(String filename) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        BufferedReader bufferedReader = new BufferedReader(new FileReader(filename));
        bufferedReader.lines().forEach(stringBuilder::append);
        return stringBuilder.toString();
    }

    private Quiz getQuizFromJSON(JSONObject root) {
        var quiz = new Quiz();

        var quizRoot = root.getJSONObject("quiz");
        var categorySet = quizRoot.keySet();

        for (String quizCategoryName : categorySet) {
            JSONArray categoryJSON = quizRoot.getJSONArray(quizCategoryName);
            QuizCategory quizCategory = getQuizCategory(quizCategoryName, categoryJSON);
            quiz.addCategory(quizCategory);
        }
        return quiz;
    }

    private QuizCategory getQuizCategory(String category, JSONArray categoryJSON) {
        QuizCategory quizCategory = new QuizCategory(category);
        for (Object questionObject : categoryJSON) {
            JSONObject questionJSON = (JSONObject) questionObject;
            QuizQuestion question = jsonToQuizQuestion(questionJSON);
            quizCategory.addQuestion(question);
        }
        return quizCategory;
    }

    private QuizQuestion jsonToQuizQuestion(JSONObject jsonObject) {
       String question = jsonObject.getString("question");
       String answer = jsonObject.getString("answer");

        JSONArray choiceArray = jsonObject.getJSONArray("options");
        List<String> options = new ArrayList<>();
        for (int i = 0; i < choiceArray.length(); i++) {
            options.add(choiceArray.getString(i));
        }

        return new QuizQuestion(question, options, answer);
    }
}
```

