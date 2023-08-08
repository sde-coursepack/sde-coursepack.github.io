--- 
Title: Plaintext File Reading
---

* TOC
  {:toc}

This page will give a short example of writing a plaintext .csv file using `BufferedWriter`. Be aware that writing to a file is considered **pre-requisite knowledge**, but this gives an example of how to use a `BufferedWriter` in Java to perform this task. There are other means of writing to a file, and you are not required to use a `BufferedWriter`.


# Plaintext File Writing

Let's consider the following simple "pick a number" game. In addition to printing the output, let's say we wanted to generate a log for debugging purposes. The log will tell us what number was generated, what each user input was, and what they were shown. Specifically, we are going to write this log to a file called `log.txt`.

## `Log.java`

```java
public class Log {
    private final BufferedWriter log;

    public Log(String filename) throws IOException {
        log = new BufferedWriter(new FileWriter(filename));
    }

    public void writeLine(String s) throws IOException {
        log.write(s + "\n");
    }

    public void close() throws IOException {
        log.close();
    }
}
```

Here, we are setting up a simple log where, at construction time, we specify the file to be logged to. We specify two functions, `writeLine` and `close` which are fairly self-explanatory. `writeLine` appends the new line character so that each logged item appears on a new line.

## `GuessANumberGame.java`

The below class models the logic of a `GuessANumberGame`

```java
public class GuessANumberGame {
private static final int MIN = 1;
private static final int MAX = 100;

    private static final int INITIAL_GUESSES = 7;

    private final int answer;
    private int guessesRemaining;

    public GuessANumberGame() {
        Random random = new Random();
        answer = random.nextInt(MIN, MAX+1); // plus one to make 100 inclusive
        guessesRemaining = INITIAL_GUESSES;
    }

    public int getAnswer() {
        return answer;
    }

    public int getGuessesRemaining() {
        return guessesRemaining;
    }

    public GuessANumberResult submitGuess(int guess) {
        if (guessesRemaining <= 0) {
            throw new IllegalStateException("No more guesses remaining!");
        }
        guessesRemaining--;
        if (guess == answer) {
            return GuessANumberResult.CORRECT;
        } else if (guess > answer) {
            return GuessANumberResult.TOO_HIGH;
        } else {
            return GuessANumberResult.TOO_LOW;
        }
    }

}
```

`GuessANumberResult` is an enumerated type withg three values: 
* `CORRECT`
* `TOO_HIGH`
* `TOO_LOW`

Which is returned by `submitGuess` to state the result. From there, we handle the User interface of the game in `Main.java` below.

## `Main.java`

```java
public class Main {
    private static Log log;
    private static Scanner scanner;
    private static GuessANumberGame game;
    private static boolean hasWon = false;

    public static void main(String[] args) {
        try {
            initiateGame();
            while (gameNotOver()) {
                System.out.println("Guesses Left: " + game.getGuessesRemaining());
                int guess = getGuessFromUser();
                handleGuess(guess);
            }
            generateWinOrLossMessage();
            log.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private static boolean gameNotOver() {
        return game.getGuessesRemaining() > 0 && !hasWon;
    }

    private static void initiateGame() throws IOException {
        scanner = new Scanner(System.in);
        log = new Log("log.txt");
        game = new GuessANumberGame();
        log.writeLine("New Game generated: answer = " + game.getAnswer());
    }

    public static int getGuessFromUser() throws IOException {
        while (true) {
            System.out.println("Enter a number (1-100): ");
            String input = scanner.nextLine();
            log.writeLine("User Number Entry: " + input);
            try {
                return Integer.parseInt(input);
            } catch (NumberFormatException e) {
                System.out.println("Invalid Entry, not a number. Try again!");
            }
        }
    }

    private static void handleGuess(int guess) throws IOException {
        GuessANumberResult result = game.submitGuess(guess);
        log.writeLine("Result: " + result);
        switch (result) {
            case CORRECT -> hasWon = true;
            case TOO_LOW -> System.out.println("Guess is too low!");
            case TOO_HIGH -> System.out.println("Guess is too high!");
        }
    }

    private static void generateWinOrLossMessage() throws IOException {
        log.writeLine("Game Result: has won? " + hasWon);
        if (hasWon) {
            System.out.println("Congratulations! You won!");
        } else {
            System.out.println("Sorry, you lost! The answer was " + game.getAnswer());
        }
    }
}
```

You'll notice that many methods have `throws IOException` in the signature. The rationale for this is that anywhere logging occurs, we need to handle the IOException. However, rather than scattering try-catch blocks throughout our code, we can simply worry about the exception handling in the `main` function. The reason is that none of the functions have any meaningful way to handle an `IOException`, so there's no reason to catch the exception in those methods. Additionally, following the DRY principle (Don't Repeat Yourself), we want to handle the `IOException` that can come from using `Log` in one place unless we have a very good reason otherwise (which we don't!).

## BufferedWriter

Be aware that the use of BufferedWriter here **overwrites** the file that already exists. If you want to **append** to a file, one obvious option would be to read the existing contents to the file with a `BufferedReader`, then open that file with a `BufferedWriter` and re-write the previous contents. However, there is an easier way. Specifically, the `FileReader` constructor has an optional boolean argument:

```java
  new FileWriter(filename, true)
```

The `true` in this case means "I would like to append to an existing file if it exists", rather than deleting and overwriting the file. By default, when you simply call `new FileWriter(filename)`, that is the same as calling `new FileWriter(filename, false)`, meaning "do not append".

Thus, we can simply update our constructor to default to appending.

```java
public class Log {
    private final BufferedWriter log;

    public Log(String filename) throws IOException {
        log = new BufferedWriter(new FileWriter(filename, true));
    }
    ...
}
```