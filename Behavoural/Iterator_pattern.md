# Iterator Design Pattern — Java & Kotlin

An interview-focused guide to the Iterator Pattern, using a playlist example with collection traversal, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Iterator Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How the Iterator Pattern Works
6. Types and Common Use Cases
7. When to Use the Iterator Pattern
8. Iterator vs. Other Patterns
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Iterator Pattern?

The Iterator Pattern is a behavioral design pattern that provides a way to access elements of a collection sequentially without exposing the collection's internal structure.

In simple words, the Iterator Pattern allows you to traverse a collection one element at a time without worrying about how the collection stores its data.

### Real-world example: Music playlist

Imagine you have a music playlist containing three songs:

- Song A
- Song B
- Song C

You want to play the songs one by one.

You could access them using indexes, but what if the playlist uses a linked list or another data structure instead of an array?

An iterator provides a common way to access each song without needing to know how the playlist stores it.

What happened to Multi-select in Desktop - The Spotify Community

What's the Most Popular Music Streaming Service of 2018? | The Motley Fool

Adverteren via digital audio - FD Mediagroep - NL

For example, in Java:

java

```
Iterator<String> iterator = songs.iterator();

while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

Here:

- `hasNext()` checks whether another song is available.
- `next()` returns the next song and advances the iterator.

Memory trick: Iterator Pattern = Traverse a collection one element at a time without exposing its internal structure.

# 2. Why Do We Need It?

Suppose you have a playlist implemented using an array.

java

```
class Playlist {

    private String[] songs = {
        "Song A",
        "Song B",
        "Song C"
    };

    public String[] getSongs() {
        return songs;
    }
}
```

A client might access the songs using indexes:

java

```
Playlist playlist = new Playlist();

String[] songs = playlist.getSongs();

for (int i = 0; i < songs.length; i++) {
    System.out.println(songs[i]);
}
```

This works, but the client depends on the collection's representation.

Now imagine the playlist changes to use a linked list or a custom data structure. The traversal logic might need to change as well.

This can lead to:

- Clients depending on internal collection details.
- Different traversal logic for different data structures.
- Internal data structures being exposed unnecessarily.
- Difficulties supporting alternative traversal strategies.
- Repeated iteration code across the application.

## With the Iterator Pattern

The collection provides an iterator that knows how to traverse its elements.

text

```
             Client
                |
                v
             Iterator
                |
                v
             Playlist
                |
                v
        Song A → Song B → Song C
```

The client interacts with the iterator rather than directly managing the collection's internal structure.

The flow is:

1. The client requests an iterator from the collection.
2. The iterator tracks the current traversal position.
3. The client checks whether another element exists.
4. The client retrieves the next element.
5. The process continues until no elements remain.

Important: The client does not need to know whether the collection uses an array, linked list, tree, or another internal representation.

# 3. Java Implementation

Let's build a playlist using the Iterator Pattern.

## Step 1: Create the Iterator interface

For learning purposes, we'll define our own iterator interface.

java

```
interface Iterator<T> {

    boolean hasNext();

    T next();
}
```

### What does `Iterator` do?

- `hasNext()` checks whether another element is available.
- `next()` returns the next element.
- The interface provides a common traversal contract.

Java already provides `java.util.Iterator<T>`, which you would normally use in production code. Our custom interface makes the pattern easier to understand.

## Step 2: Create the Playlist collection

The playlist stores songs and provides an iterator.

java

```
class Playlist {

    private final String[] songs = {
        "Song A",
        "Song B",
        "Song C"
    };

    public Iterator<String> createIterator() {
        return new PlaylistIterator(songs);
    }
}
```

Notice that the playlist does not expose its internal array.

Instead, it creates and returns an iterator.

## Step 3: Create the Concrete Iterator

The concrete iterator tracks the current position in the playlist.

java

```
class PlaylistIterator implements Iterator<String> {

    private final String[] songs;
    private int position = 0;

    public PlaylistIterator(String[] songs) {
        this.songs = songs;
    }

    @Override
    public boolean hasNext() {
        return position < songs.length;
    }

    @Override
    public String next() {

        if (!hasNext()) {
            throw new java.util.NoSuchElementException();
        }

        return songs[position++];
    }
}
```

### How does it work?

Consider this field:

java

```
private int position = 0;
```

It tracks which song should be returned next.

When `next()` is called:

java

```
return songs[position++];
```

The iterator returns the song at the current position and then increments the position.

For example:

| Call | Returned song | Position afterward |
| --- | --- | --- |
| First `next()` | Song A | 1 |
| Second `next()` | Song B | 2 |
| Third `next()` | Song C | 3 |

Once the position reaches the length of the array, `hasNext()` returns `false`.

## Step 4: Create the Client

The client uses the iterator to traverse the playlist.

java

```
public class Main {

    public static void main(String[] args) {

        Playlist playlist = new Playlist();

        Iterator<String> iterator =
            playlist.createIterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

Output:

text

```
Song A
Song B
Song C
```

### What happens here?

- `Main` is the client.
- `Playlist` is the collection.
- `PlaylistIterator` is the concrete iterator.
- The iterator handles the traversal position.

The client does not need to access the array or manage the index itself.

Interview takeaway: The Iterator Pattern separates collection traversal from the collection's internal representation.

## Step 5: Use Java's built-in Iterator

In real Java applications, you will usually use `Iterable` and `java.util.Iterator`.

java

```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class Main {

    public static void main(String[] args) {

        List<String> songs = new ArrayList<>();

        songs.add("Song A");
        songs.add("Song B");
        songs.add("Song C");

        Iterator<String> iterator = songs.iterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

Output:

text

```
Song A
Song B
Song C
```

Java collections such as `ArrayList`, `LinkedList`, and `HashSet` provide iterators, allowing clients to traverse them through a common API.

# 4. Kotlin Implementation

Kotlin also supports the Iterator Pattern.

Let's implement the same playlist example.

## Complete implementation

kotlin

```
// Custom iterator interface
interface PlaylistIterator<T> {

    fun hasNext(): Boolean

    fun next(): T
}

// Collection
class Playlist {

    private val songs = listOf(
        "Song A",
        "Song B",
        "Song C"
    )

    fun createIterator(): PlaylistIterator<String> {
        return SongsIterator(songs)
    }
}

// Concrete iterator
class SongsIterator(
    private val songs: List<String>
) : PlaylistIterator<String> {

    private var position = 0

    override fun hasNext(): Boolean {
        return position < songs.size
    }

    override fun next(): String {

        if (!hasNext()) {
            throw NoSuchElementException()
        }

        return songs[position++]
    }
}

// Client
fun main() {

    val playlist = Playlist()

    val iterator = playlist.createIterator()

    while (iterator.hasNext()) {
        println(iterator.next())
    }
}
```

Output:

text

```
Song A
Song B
Song C
```

## Why is this the Iterator Pattern?

Consider:

kotlin

```
private var position = 0
```

The iterator maintains the current traversal position.

Then:

kotlin

```
return songs[position++]
```

returns the current song and advances to the next position.

The playlist is responsible for providing the iterator, while the iterator is responsible for traversing the elements.

## Kotlin's built-in iteration

Kotlin already supports iteration through `Iterable` and `Iterator`.

kotlin

```
fun main() {

    val songs = listOf(
        "Song A",
        "Song B",
        "Song C"
    )

    val iterator = songs.iterator()

    while (iterator.hasNext()) {
        println(iterator.next())
    }
}
```

You can also use a `for` loop:

kotlin

```
for (song in songs) {
    println(song)
}
```

The `for` loop provides convenient syntax for traversing an iterable collection.

# 5. How the Iterator Pattern Works

The pattern has four common roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Iterator | Defines traversal operations | `Iterator<T>` |
| Concrete Iterator | Tracks the current position and traverses elements | `PlaylistIterator` |
| Aggregate / Collection | Stores elements and provides an iterator | `Playlist` |
| Concrete Aggregate | Implements the collection | `Playlist` implementation |

## Class diagram

text

```
                 Client
                    |
                    v
                 Iterator
                    ^
                    |
             PlaylistIterator
                    ^
                    |
                 Playlist
             creates iterator
                    |
                    v
               Song storage
```

More precisely:

- The collection creates or supplies an iterator.
- The iterator keeps track of traversal state.
- The client uses the iterator interface to retrieve elements.
- The collection's internal representation remains hidden from the client.

## Execution flow

For our playlist:

text

```
Client creates Playlist
          |
          v
Client requests Iterator
          |
          v
Iterator position = 0
          |
          v
hasNext() → true
          |
          v
next() → Song A
          |
          v
hasNext() → true
          |
          v
next() → Song B
          |
          v
hasNext() → true
          |
          v
next() → Song C
          |
          v
hasNext() → false
          |
          v
Traversal ends
```

## Why does the iterator maintain its own position?

The iterator needs to remember where it is in the collection.

For example, two iterators can traverse the same playlist independently:

java

```
Iterator<String> first = playlist.createIterator();
Iterator<String> second = playlist.createIterator();
```

Both iterators can start from the beginning and advance independently, provided the collection and iterator implementation support that behavior.

This separates traversal state from the collection itself.

## Does the Iterator Pattern only work with arrays?

No. Iterators can traverse arrays, linked lists, trees, sets, and many other collection types.

The traversal logic depends on the underlying collection, but the client can use a consistent iterator interface.

# 6. Types and Common Use Cases

The Iterator Pattern is commonly used in several traversal styles.

## 1. Forward Iterator

Traverses elements from beginning to end.

Example: Reading songs from the first song to the last song.

Java's standard `Iterator` supports forward traversal.

## 2. Bidirectional Iterator

Allows traversal in both directions.

Example: Moving forward and backward through a music playlist.

Java's `ListIterator` supports bidirectional traversal of lists, including `previous()` and `next()`.

## 3. Internal Iterator

The collection or iteration mechanism controls traversal, often using a callback or higher-order function.

Example in Kotlin:

kotlin

```
songs.forEach { song ->
    println(song)
}
```

The caller provides the action, while the iteration mechanism controls the traversal.

## 4. External Iterator

The client explicitly controls the traversal by requesting elements.

java

```
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

This gives the client control over when to advance, stop, or perform additional work.

### Common real-world use cases

- Traversing music playlists.
- Iterating over lists, sets, and maps.
- Reading records from data sources.
- Traversing file and directory structures.
- Visiting nodes in trees.
- Processing paginated data.
- Reading items from custom collections.

Note that not every loop is a separate implementation of the Iterator Pattern. The pattern is about providing a consistent traversal mechanism while hiding the collection's internal representation.

# 7. When to Use the Iterator Pattern

Use this pattern when:

- You want to traverse a collection without exposing its internal structure.
- Different collections need a consistent traversal API.
- You want to maintain traversal state separately from the collection.
- You need multiple independent traversals over the same collection.
- You want to provide specialized traversal behavior, such as forward or reverse traversal.

### When should you avoid it?

Avoid implementing a custom iterator when:

- The language's built-in collection iterators already satisfy your needs.
- A custom iterator adds complexity without providing useful abstraction.
- The collection's traversal is trivial and no encapsulation benefit is needed.

Main benefit: Separates traversal logic from the collection's internal structure.

Main limitation: Custom iterators require careful handling of traversal state, exhaustion, and collection modifications.

# 8. Iterator vs. Other Patterns

| Feature | Iterator | Composite | Visitor | Factory Method | Command |
| --- | --- | --- | --- | --- | --- |
| Main purpose | Traverse a collection | Represent part-whole hierarchies | Add operations over object structures | Delegate object creation to subclasses | Encapsulate a request |
| Main focus | Accessing elements sequentially | Organizing objects | Performing operations on elements | Creating objects | Executing operations |
| Example | Traverse songs | Represent folders and files | Calculate file sizes | Create different document types | Turn on a light |

## Iterator vs. Composite

- Iterator traverses elements in a collection.
- Composite represents individual objects and groups using a common interface.

Example: Iterator traverses files in a directory structure. Composite represents files and directories through a common abstraction.

## Iterator vs. Visitor

- Iterator provides a way to traverse elements.
- Visitor defines operations to perform on elements of an object structure.

These patterns can be combined: an iterator can traverse a structure while a visitor performs operations on each element.

## Iterator vs. Factory Method

- Iterator focuses on traversal.
- Factory Method focuses on object creation.

A collection may create an iterator, but that does not make Iterator and Factory Method the same pattern.

## Iterator vs. Command

- Iterator provides sequential access to elements.
- Command encapsulates an operation as an object.

Example: An iterator retrieves songs from a playlist, while a command represents the action of playing a song.

# 9. Common Interview Questions

### Q1. What is the Iterator Pattern?

It is a behavioral design pattern that provides sequential access to the elements of a collection without exposing its internal representation.

### Q2. What are the main components?

- Iterator: Defines traversal operations.
- Concrete Iterator: Maintains traversal state and retrieves elements.
- Aggregate: Provides an iterator.
- Concrete Aggregate: Stores elements and creates the concrete iterator.
- Client: Uses the iterator to traverse elements.

### Q3. Why do we need the Iterator Pattern?

It separates traversal logic from the collection and provides a consistent way to access different collection types.

### Q4. Is Iterator a behavioral or structural pattern?

It is a behavioral design pattern because it focuses on how objects are accessed and traversed.

### Q5. What is the difference between `hasNext()` and `next()`?

- `hasNext()` checks whether another element is available without advancing the iterator.
- `next()` returns the next element and advances the iterator.

### Q6. What happens when there are no more elements?

In Java's standard `Iterator`, calling `next()` when no elements remain throws `NoSuchElementException`. Custom implementations should also define clear behavior for exhausted iterators.

### Q7. Can multiple iterators traverse the same collection?

Yes. Each iterator can maintain its own traversal position, allowing independent traversals. Behavior may depend on the collection and its modification rules.

### Q8. What is the difference between an internal and external iterator?

An external iterator allows the client to control traversal explicitly. An internal iterator controls traversal through callbacks or operations such as `forEach`.

### Q9. What are the advantages?

- Hides the collection's internal representation.
- Separates traversal logic from collection storage.
- Provides a consistent traversal interface.
- Supports independent traversal state.
- Makes custom traversal strategies possible.

### Q10. What are the disadvantages?

- Custom iterators introduce additional code.
- Collection modifications during traversal can cause problems.
- Iterator state must be managed correctly.
- Some iterators require extra memory or processing, depending on the data structure.

### Q11. Is Iterator used in Java collections?

Yes. Many Java collections implement `Iterable` and provide iterators through `iterator()`. Examples include `ArrayList`, `LinkedList`, and `HashSet`.

### Q12. What is the difference between `Iterable` and `Iterator` in Java?

- `Iterable` represents an object that can provide an iterator through `iterator()`.
- `Iterator` represents the mechanism that traverses the elements through methods such as `hasNext()` and `next()`.

### Q13. Can an iterator remove elements?

Java's `Iterator` provides an optional `remove()` operation. Whether it is supported depends on the implementation. Kotlin's standard read-only `Iterator` does not provide the same removal method.

### Q14. What is `ListIterator` in Java?

`ListIterator` is a specialized iterator for lists. It supports traversal in both directions and provides additional operations such as retrieving indexes and modifying list elements when supported.

### Q15. What happens if a collection is modified while iterating?

The behavior depends on the collection and iterator implementation. Many standard Java collections use fail-fast iterators that may throw `ConcurrentModificationException` after certain structural modifications. This is a best-effort bug-detection mechanism, not a thread-safety guarantee.

### Q16. Can Iterator be used with trees?

Yes. Iterators can implement tree traversal strategies such as preorder, inorder, postorder, or level-order traversal.

### Q17. How is Iterator different from a simple `for` loop?

A `for` loop is a language construct for repetition. The Iterator Pattern provides an abstraction for accessing collection elements without requiring the client to know how the collection is stored. Many `for-each` loops use iterators internally.

### Q18. Does Iterator improve performance?

Not necessarily. Its primary purpose is abstraction and encapsulation. Performance depends on the collection, iterator implementation, and traversal operation.

# 10. Quick Revision

- Category: Behavioral design pattern.
- Purpose: Traverse a collection without exposing its internal structure.
- Iterator: Defines traversal operations.
- Concrete Iterator: Tracks the current position.
- Aggregate: Provides an iterator.
- Client: Uses the iterator to retrieve elements.
- Java: Uses `Iterable`, `Iterator`, and `ListIterator`.
- Kotlin: Supports `Iterator`, `Iterable`, `for` loops, and collection operations such as `forEach`.
- External iteration: The client controls traversal.
- Internal iteration: The iteration mechanism controls traversal.
- Main benefit: Separates traversal from collection storage.
- Main limitation: Custom iterators need careful handling of exhaustion and collection modifications.

## One-Line Memory Trick

Iterator Pattern = Access collection elements one by one without knowing how the collection stores them.
