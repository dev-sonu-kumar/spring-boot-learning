# Java Fundamentals Refresher - Complete Guide for Spring Framework

## Overview

This document is a comprehensive refresher on Java fundamentals that you need to master before learning Spring Framework. It covers all core concepts from basics to advanced features.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Variables and Data Types](#variables-and-data-types)
3. [Operators](#operators)
4. [Control Flow](#control-flow)
5. [Arrays](#arrays)
6. [Strings](#strings)
7. [Object-Oriented Programming (OOP)](#object-oriented-programming-oop)
8. [Classes and Objects](#classes-and-objects)
9. [Inheritance](#inheritance)
10. [Polymorphism](#polymorphism)
11. [Encapsulation](#encapsulation)
12. [Abstraction](#abstraction)
13. [Interfaces](#interfaces)
14. [Collections Framework](#collections-framework)
15. [Exception Handling](#exception-handling)
16. [Generics](#generics)
17. [Lambda Expressions and Streams](#lambda-expressions-and-streams)
18. [Java 8+ Features](#java-8-features)
19. [File I/O](#file-io)
20. [Annotations](#annotations)

---

## Getting Started

### Java Installation and Setup

```bash
# Check Java version
java -version

# Check javac version
javac -version

# Compile Java file
javac HelloWorld.java

# Run Java program
java HelloWorld
```

### Basic Program Structure

```java
public class HelloWorld {
  // Entry point of every Java application
  public static void main(String[] args) {
    System.out.println("Hello, World!");
  }
}
```

**Explanation**:

- `public`: Accessible from anywhere
- `class`: Blueprint for objects
- `static`: Belongs to class, not instance
- `void`: No return value
- `String[] args`: Command-line arguments

---

## Variables and Data Types

### Primitive Data Types

```java
// Integer Types
byte age = 25;              // 8 bits (-128 to 127)
short year = 2024;          // 16 bits
int salary = 50000;         // 32 bits (most common)
long population = 8000000L; // 64 bits (append 'L')

// Floating-point Types
float price = 19.99f;       // 32 bits (append 'f')
double gpa = 3.75;          // 64 bits (default for decimals)

// Boolean
boolean isActive = true;
boolean isDeleted = false;

// Character
char grade = 'A';
char unicode = '\u0041'; // A
```

### Reference Data Types

```java
// Strings
String name = "John Doe";
String message = new String("Hello");

// Objects
Person person = new Person();

// Arrays
int[] numbers = {1, 2, 3};

// Collections
List<String> list = new ArrayList<>();
Map<String, Integer> map = new HashMap<>();
```

### Type Conversion

```java
// Implicit conversion (smaller to larger)
int number = 42;
long largeNumber = number; // Automatically converted

// Explicit conversion (larger to smaller)
double decimal = 9.99;
int integer = (int) decimal; // Casting required, loses decimal

// String to primitive
String text = "123";
int value = Integer.parseInt(text); // "123" → 123
double price = Double.parseDouble("19.99");

// Primitive to String
int num = 42;
String str = String.valueOf(num); // 42 → "42"
String str2 = "" + num;           // 42 → "42"
```

---

## Operators

### Arithmetic Operators

```java
int a = 10, b = 3;

int sum = a + b;        // 13
int diff = a - b;       // 7
int product = a * b;    // 30
int quotient = a / b;   // 3 (integer division)
int remainder = a % b;  // 1

double division = (double) a / b; // 3.333... (float division)

// Compound assignments
a += 5;  // a = a + 5
b *= 2;  // b = b * 2
```

### Increment and Decrement

```java
int count = 5;

// Pre-increment (increment first, then use)
int result1 = ++count; // count becomes 6, result1 = 6

count = 5;
// Post-increment (use first, then increment)
int result2 = count++; // result2 = 5, count becomes 6

count = 5;
int result3 = --count; // Pre-decrement
count = 5;
int result4 = count--; // Post-decrement
```

### Comparison Operators

```java
int a = 10, b = 5;

boolean isEqual = (a == b);      // false
boolean isNotEqual = (a != b);   // true
boolean isGreater = (a > b);     // true
boolean isGreaterOrEqual = (a >= b); // true
boolean isLess = (a < b);        // false
boolean isLessOrEqual = (a <= b); // false
```

### Logical Operators

```java
boolean x = true, y = false;

// AND operator
boolean andResult = (x && y);  // false
boolean andShort = (x & y);   // false (no short-circuit)

// OR operator
boolean orResult = (x || y);   // true
boolean orShort = (x | y);    // true (no short-circuit)

// NOT operator
boolean notResult = !x;        // false

// XOR operator
boolean xorResult = (x ^ y);   // true
```

### Ternary Operator

```java
int age = 20;
String status = (age >= 18) ? "Adult" : "Minor";

// Nested ternary
int marks = 75;
String grade = (marks >= 80) ? "A" : (marks >= 70) ? "B" : "C";
```

---

## Control Flow

### If-Else Statements

```java
int age = 20;

// Simple if
if (age >= 18) {
  System.out.println("You are an adult");
}

// If-else
if (age >= 18) {
  System.out.println("You are an adult");
} else {
  System.out.println("You are a minor");
}

// If-else-if chain
int marks = 75;
if (marks >= 80) {
  System.out.println("Grade A");
} else if (marks >= 70) {
  System.out.println("Grade B");
} else if (marks >= 60) {
  System.out.println("Grade C");
} else {
  System.out.println("Grade F");
}
```

### Switch Statement

```java
int day = 3;
String dayName;

switch (day) {
  case 1:
    dayName = "Monday";
    break;
  case 2:
    dayName = "Tuesday";
    break;
  case 3:
    dayName = "Wednesday";
    break;
  default:
    dayName = "Unknown";
}

// Switch with String (Java 7+)
String fruit = "apple";
switch (fruit) {
  case "apple":
    System.out.println("Red fruit");
    break;
  case "banana":
    System.out.println("Yellow fruit");
    break;
  default:
    System.out.println("Unknown fruit");
}

// Switch expression (Java 12+)
String color = switch (day) {
  case 1, 2, 3, 4, 5 -> "Weekday";
  case 6, 7 -> "Weekend";
  default -> "Unknown";
};
```

### While Loop

```java
int count = 1;
while (count <= 5) {
  System.out.println(count);
  count++;
}

// Do-while loop (executes at least once)
int i = 1;
do {
  System.out.println(i);
  i++;
} while (i <= 5);
```

### For Loop

```java
// Traditional for loop
for (int i = 1; i <= 5; i++) {
  System.out.println(i);
}

// For-each loop (enhanced for)
String[] names = {"Alice", "Bob", "Charlie"};
for (String name : names) {
  System.out.println(name);
}

// With collections
List<Integer> numbers = Arrays.asList(10, 20, 30);
for (Integer num : numbers) {
  System.out.println(num);
}
```

### Break and Continue

```java
// Break: Exit loop
for (int i = 1; i <= 10; i++) {
  if (i == 5) {
    break; // Exit loop
  }
  System.out.println(i); // 1, 2, 3, 4
}

// Continue: Skip current iteration
for (int i = 1; i <= 5; i++) {
  if (i == 3) {
    continue; // Skip this iteration
  }
  System.out.println(i); // 1, 2, 4, 5
}

// Labeled break
outer: for (int i = 1; i <= 3; i++) {
  for (int j = 1; j <= 3; j++) {
    if (i == 2 && j == 2) {
      break outer; // Exit outer loop
    }
  }
}
```

---

## Arrays

### Single Dimensional Arrays

```java
// Declaration and initialization
int[] numbers = new int[5];           // Array of size 5
int[] values = {10, 20, 30, 40, 50}; // Initialize with values
String[] names = new String[3];       // Array of Strings

// Accessing elements
System.out.println(values[0]); // 10
values[1] = 25;                 // Modify element

// Array length
System.out.println(values.length); // 5

// Iterate through array
for (int i = 0; i < values.length; i++) {
  System.out.println(values[i]);
}

for (int value : values) {
  System.out.println(value);
}
```

### Multi-Dimensional Arrays

```java
// 2D Array
int[][] matrix = new int[3][3];
int[][] grid = {
  {1, 2, 3},
  {4, 5, 6},
  {7, 8, 9}
};

// Access 2D array
System.out.println(grid[0][1]); // 2

// Iterate 2D array
for (int i = 0; i < grid.length; i++) {
  for (int j = 0; j < grid[i].length; j++) {
    System.out.println(grid[i][j]);
  }
}

// 3D Array
int[][][] cube = new int[3][3][3];
```

### Useful Array Methods

```java
int[] arr = {5, 2, 8, 1, 9};

// Sort array
Arrays.sort(arr);
System.out.println(Arrays.toString(arr)); // [1, 2, 5, 8, 9]

// Search in sorted array
int index = Arrays.binarySearch(arr, 5); // Returns index

// Fill array
Arrays.fill(arr, 0); // Fill with 0

// Copy array
int[] copy = Arrays.copyOf(arr, arr.length);
int[] partial = Arrays.copyOfRange(arr, 0, 3);

// Check equality
int[] arr2 = {1, 2, 5, 8, 9};
boolean equal = Arrays.equals(arr, arr2);

// Convert to List
List<Integer> list = Arrays.asList(1, 2, 3);
```

---

## Strings

### String Basics

```java
// String creation
String str1 = "Hello";           // String literal
String str2 = new String("Hello"); // String object

// String is immutable
String original = "Java";
String modified = original + " Programming"; // New object created

// Length
int length = str1.length(); // 5

// Character at index
char ch = str1.charAt(0); // 'H'

// Substring
String substr = str1.substring(1);    // "ello"
String substr2 = str1.substring(0, 3); // "Hel"
```

### String Methods

```java
String text = "Hello World";

// Case conversion
String upper = text.toUpperCase();   // "HELLO WORLD"
String lower = text.toLowerCase();   // "hello world"

// Find and replace
boolean contains = text.contains("World");     // true
int index = text.indexOf("o");                 // 4
int lastIndex = text.lastIndexOf("o");         // 7
String replaced = text.replace("World", "Java"); // "Hello Java"

// Trim
String padded = "  Hello  ";
String trimmed = padded.trim();     // "Hello"

// Split
String[] words = text.split(" ");   // ["Hello", "World"]

// Starts and ends with
boolean startsWith = text.startsWith("Hello");  // true
boolean endsWith = text.endsWith("World");      // true

// Compare
int compare = "abc".compareTo("abc"); // 0 (equal)
boolean equals = "Hello".equals("Hello");        // true
boolean equalsIgnoreCase = "Hello".equalsIgnoreCase("hello"); // true
```

### StringBuilder and StringBuffer

```java
// StringBuilder (mutable, not synchronized)
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" ");
sb.append("World");
String result = sb.toString(); // "Hello World"

// StringBuffer (mutable, synchronized, slower)
StringBuffer sbf = new StringBuffer();
sbf.append("Hello");

// StringBuilder methods
StringBuilder sb2 = new StringBuilder("Hello");
sb2.insert(5, " World");    // "Hello World"
sb2.delete(5, 11);          // "Hello"
sb2.reverse();              // "olleH"
sb2.replace(0, 2, "HI");    // "HIllo"

// Efficient string concatenation in loops
StringBuilder sb3 = new StringBuilder();
for (int i = 0; i < 1000; i++) {
  sb3.append(i).append(",");
}
```

---

## Object-Oriented Programming (OOP)

### Pillars of OOP

1. **Encapsulation**: Data hiding
2. **Inheritance**: Code reuse
3. **Polymorphism**: Many forms
4. **Abstraction**: Hide complexity

---

## Classes and Objects

### Creating a Class

```java
public class Person {
  // Attributes (Member Variables)
  private String name;
  private int age;
  private String email;

  // Constructor (Parameterless)
  public Person() {
    this.name = "Unknown";
    this.age = 0;
  }

  // Constructor with parameters
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }

  // Constructor with all parameters
  public Person(String name, int age, String email) {
    this.name = name;
    this.age = age;
    this.email = email;
  }

  // Getter methods
  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }

  // Setter methods
  public void setName(String name) {
    this.name = name;
  }

  public void setAge(int age) {
    if (age > 0) {
      this.age = age;
    }
  }

  // Business logic method
  public void printInfo() {
    System.out.println("Name: " + name + ", Age: " + age);
  }

  // toString method
  @Override
  public String toString() {
    return "Person{" +
      "name='" + name + '\'' +
      ", age=" + age +
      '}';
  }
}
```

### Creating and Using Objects

```java
// Create objects
Person person1 = new Person("Alice", 25);
Person person2 = new Person("Bob", 30, "bob@example.com");
Person person3 = new Person(); // Default values

// Access methods
person1.printInfo();
System.out.println(person1.getName());

// Modify using setter
person1.setAge(26);

// toString is called implicitly
System.out.println(person1); // Person{name='Alice', age=26}
```

### This Keyword

```java
public class Car {
  private String brand;

  public Car(String brand) {
    this.brand = brand; // 'this' refers to current object
  }

  public void compare(Car other) {
    if (this.brand.equals(other.brand)) {
      System.out.println("Same brand");
    }
  }

  public void setBrand(String brand) {
    this.brand = brand;
  }

  // Call another constructor
  public Car() {
    this("Unknown"); // Calls parameterized constructor
  }
}
```

### Static Members

```java
public class Student {
  private String name;
  static int totalStudents = 0; // Shared by all instances

  public Student(String name) {
    this.name = name;
    totalStudents++; // Increment when student created
  }

  // Static method
  static void printTotalStudents() {
    System.out.println("Total: " + totalStudents);
  }

  // Static block (executed when class loads)
  static {
    System.out.println("Student class loaded");
  }
}

// Usage
Student s1 = new Student("Alice");
Student s2 = new Student("Bob");
Student.printTotalStudents(); // Total: 2
System.out.println(Student.totalStudents); // 2
```

### Final Keyword

```java
// Final class - cannot be extended
public final class ImmutableClass {
  // Implementation
}

// Final method - cannot be overridden
public class Parent {
  public final void criticalMethod() {
    // Cannot be overridden in child class
  }
}

// Final variable - cannot be reassigned
public class Config {
  public static final double PI = 3.14159;

  public void method() {
    final int value = 10;
    // value = 20; // Compilation error
  }
}
```

---

## Inheritance

### Basic Inheritance

```java
// Parent class (Superclass)
public class Animal {
  protected String name;

  public Animal(String name) {
    this.name = name;
  }

  public void eat() {
    System.out.println(name + " is eating");
  }

  public void sleep() {
    System.out.println(name + " is sleeping");
  }
}

// Child class (Subclass)
public class Dog extends Animal {
  private String breed;

  public Dog(String name, String breed) {
    super(name); // Call parent constructor
    this.breed = breed;
  }

  // Override parent method
  @Override
  public void eat() {
    System.out.println(name + " is eating dog food");
  }

  // New method specific to Dog
  public void bark() {
    System.out.println(name + " is barking");
  }
}
```

### Using Inherited Classes

```java
Dog dog = new Dog("Buddy", "Golden Retriever");
dog.eat();      // Buddy is eating dog food (overridden)
dog.sleep();    // Buddy is sleeping (inherited)
dog.bark();     // Buddy is barking (new method)
```

### Super Keyword

```java
public class Vehicle {
  protected String color;

  public void display() {
    System.out.println("Vehicle color: " + color);
  }
}

public class Car extends Vehicle {
  private String model;

  public Car(String color, String model) {
    this.color = color;
    this.model = model;
  }

  @Override
  public void display() {
    super.display();  // Call parent method
    System.out.println("Car model: " + model);
  }
}
```

### Types of Inheritance

```java
// Single Inheritance
public class Parent { }
public class Child extends Parent { }

// Multilevel Inheritance
public class GrandParent { }
public class Parent extends GrandParent { }
public class Child extends Parent { }

// Hierarchical Inheritance
public class Parent { }
public class Child1 extends Parent { }
public class Child2 extends Parent { }

// Note: Java does NOT support multiple inheritance with classes
// Use interfaces instead
public interface Interface1 { }
public interface Interface2 { }
public class Child implements Interface1, Interface2 { }
```

---

## Polymorphism

### Method Overriding (Runtime Polymorphism)

```java
public class Shape {
  public void draw() {
    System.out.println("Drawing a shape");
  }
}

public class Circle extends Shape {
  @Override
  public void draw() {
    System.out.println("Drawing a circle");
  }
}

public class Rectangle extends Shape {
  @Override
  public void draw() {
    System.out.println("Drawing a rectangle");
  }
}

// Polymorphic behavior
Shape shape1 = new Circle();
Shape shape2 = new Rectangle();
Shape shape3 = new Shape();

shape1.draw(); // Drawing a circle
shape2.draw(); // Drawing a rectangle
shape3.draw(); // Drawing a shape
```

### Method Overloading (Compile-time Polymorphism)

```java
public class Calculator {
  // Same method name, different parameters

  public int add(int a, int b) {
    return a + b;
  }

  public double add(double a, double b) {
    return a + b;
  }

  public int add(int a, int b, int c) {
    return a + b + c;
  }

  public String add(String a, String b) {
    return a + b;
  }
}

// Usage
Calculator calc = new Calculator();
System.out.println(calc.add(5, 10));           // 15
System.out.println(calc.add(5.5, 10.5));       // 16.0
System.out.println(calc.add(5, 10, 15));       // 30
System.out.println(calc.add("Hello", "World")); // HelloWorld
```

---

## Encapsulation

### Hiding Internal Details

```java
public class BankAccount {
  // Private attributes
  private String accountNumber;
  private double balance;
  private String accountHolder;

  // Public constructor
  public BankAccount(String accountNumber, String accountHolder, double initialBalance) {
    this.accountNumber = accountNumber;
    this.accountHolder = accountHolder;
    this.balance = initialBalance;
  }

  // Controlled access through public methods
  public void deposit(double amount) {
    if (amount > 0) {
      balance += amount;
      System.out.println("Deposited: " + amount);
    }
  }

  public void withdraw(double amount) {
    if (amount > 0 && amount <= balance) {
      balance -= amount;
      System.out.println("Withdrawn: " + amount);
    } else {
      System.out.println("Invalid withdrawal amount");
    }
  }

  public double getBalance() {
    return balance;
  }

  public String getAccountHolder() {
    return accountHolder;
  }
}

// Usage
BankAccount account = new BankAccount("12345", "John", 1000);
account.deposit(500);
account.withdraw(200);
System.out.println(account.getBalance()); // 1300
```

---

## Abstraction

### Abstract Classes

```java
// Abstract class
public abstract class DatabaseConnection {
  // Abstract method (no implementation)
  public abstract void connect();

  public abstract void disconnect();

  // Concrete method
  public void executeQuery(String query) {
    System.out.println("Executing: " + query);
  }
}

// Concrete class must implement all abstract methods
public class MySQLConnection extends DatabaseConnection {
  @Override
  public void connect() {
    System.out.println("Connected to MySQL");
  }

  @Override
  public void disconnect() {
    System.out.println("Disconnected from MySQL");
  }
}

public class OracleConnection extends DatabaseConnection {
  @Override
  public void connect() {
    System.out.println("Connected to Oracle");
  }

  @Override
  public void disconnect() {
    System.out.println("Disconnected from Oracle");
  }
}

// Usage
DatabaseConnection conn = new MySQLConnection();
conn.connect();        // Connected to MySQL
conn.executeQuery("SELECT * FROM users");
conn.disconnect();     // Disconnected from MySQL
```

---

## Interfaces

### Interface Basics

```java
// Interface definition
public interface Animal {
  // Abstract method (no implementation)
  void eat();
  void sleep();

  // Default method (Java 8+)
  default void move() {
    System.out.println("Moving");
  }

  // Static method (Java 8+)
  static void info() {
    System.out.println("This is an animal");
  }

  // Constant
  int MAX_SPEED = 100;
}

// Implementation
public class Dog implements Animal {
  @Override
  public void eat() {
    System.out.println("Dog eats meat");
  }

  @Override
  public void sleep() {
    System.out.println("Dog sleeps");
  }

  // Can override default method
  @Override
  public void move() {
    System.out.println("Dog runs");
  }
}

// Usage
Animal dog = new Dog();
dog.eat();      // Dog eats meat
dog.sleep();    // Dog sleeps
dog.move();     // Dog runs
Animal.info();  // This is an animal
```

### Interface vs Abstract Class

| Feature     | Interface                                 | Abstract Class      |
| ----------- | ----------------------------------------- | ------------------- |
| Methods     | Abstract (+ default, static from Java 8+) | Abstract + concrete |
| Variables   | public static final                       | Any access modifier |
| Constructor | No                                        | Yes                 |
| Multiple    | Yes (implements multiple)                 | No (extends one)    |
| Use Case    | Contract                                  | Shared code         |

### Multiple Interface Implementation

```java
public interface Swimmer {
  void swim();
}

public interface Runner {
  void run();
}

public class Athlete implements Swimmer, Runner {
  @Override
  public void swim() {
    System.out.println("Swimming");
  }

  @Override
  public void run() {
    System.out.println("Running");
  }
}

// Usage
Athlete athlete = new Athlete();
athlete.swim();
athlete.run();
```

---

## Collections Framework

### List Interface

```java
// ArrayList - Dynamic array, mutable
List<String> list = new ArrayList<>();
list.add("Alice");
list.add("Bob");
list.add("Charlie");

System.out.println(list.get(0));        // Alice
list.set(1, "David");                   // Replace element
list.remove(2);                         // Remove by index
System.out.println(list.size());        // Size

// LinkedList - Doubly linked list
LinkedList<Integer> numbers = new LinkedList<>();
numbers.add(10);
numbers.add(20);
numbers.addFirst(5);  // Add at beginning
numbers.addLast(30);  // Add at end
numbers.removeFirst();
numbers.removeLast();

// CopyOnWriteArrayList - Thread-safe variant
List<String> threadSafeList = new CopyOnWriteArrayList<>();
```

### Set Interface

```java
// HashSet - No duplicates, unordered
Set<String> set = new HashSet<>();
set.add("Apple");
set.add("Banana");
set.add("Apple"); // Duplicate, not added

System.out.println(set.size());    // 2
System.out.println(set.contains("Banana")); // true

// TreeSet - Sorted, no duplicates
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(30);
treeSet.add(10);
treeSet.add(20);
System.out.println(treeSet); // [10, 20, 30] - sorted

// LinkedHashSet - Maintains insertion order
Set<String> linkedSet = new LinkedHashSet<>();
linkedSet.add("First");
linkedSet.add("Second");
linkedSet.add("Third");
// Iterates in insertion order
```

### Map Interface

```java
// HashMap - Key-value pairs, unordered
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 25);
map.put("Bob", 30);
map.put("Charlie", 28);

System.out.println(map.get("Alice"));      // 25
System.out.println(map.containsKey("Bob")); // true
map.remove("Charlie");
System.out.println(map.size());            // 2

// Iterate
for (String key : map.keySet()) {
  System.out.println(key + ": " + map.get(key));
}

for (Integer value : map.values()) {
  System.out.println(value);
}

for (Map.Entry<String, Integer> entry : map.entrySet()) {
  System.out.println(entry.getKey() + ": " + entry.getValue());
}

// TreeMap - Sorted by key
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Zebra", 1);
treeMap.put("Apple", 2);
treeMap.put("Mango", 3);
// Prints in alphabetical order

// LinkedHashMap - Maintains insertion order
Map<String, Integer> linkedMap = new LinkedHashMap<>();

// ConcurrentHashMap - Thread-safe
Map<String, String> concurrentMap = new ConcurrentHashMap<>();
```

### Common Collection Methods

```java
List<Integer> list = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));

// Sorting
Collections.sort(list);  // [1, 2, 5, 8, 9]

// Reverse
Collections.reverse(list);  // [9, 8, 5, 2, 1]

// Binary search (on sorted list)
int index = Collections.binarySearch(list, 5);

// Max and Min
int max = Collections.max(list);
int min = Collections.min(list);

// Shuffle
Collections.shuffle(list);

// Frequency
int count = Collections.frequency(list, 5);

// Replace all
Collections.replaceAll(list, 5, 100);

// Copy
List<Integer> copy = new ArrayList<>(list);
```

---

## Exception Handling

### Try-Catch-Finally

```java
try {
  String text = "123abc";
  int number = Integer.parseInt(text); // Throws NumberFormatException
  System.out.println(number);
} catch (NumberFormatException e) {
  System.out.println("Invalid number format");
  e.printStackTrace();
} catch (Exception e) { // Catch generic Exception (must be last)
  System.out.println("An error occurred");
} finally {
  // Always executes, even if exception occurs
  System.out.println("Cleanup operations");
}
```

### Try with Resources

```java
// Automatically closes resources (FileReader, BufferedReader, etc.)
try (FileReader file = new FileReader("file.txt");
     BufferedReader reader = new BufferedReader(file)) {
  String line;
  while ((line = reader.readLine()) != null) {
    System.out.println(line);
  }
} catch (IOException e) {
  System.out.println("File not found or error reading");
}
```

### Throwing Exceptions

```java
public class AgeValidator {
  public void validateAge(int age) throws IllegalArgumentException {
    if (age < 0 || age > 150) {
      throw new IllegalArgumentException("Invalid age: " + age);
    }
  }

  public void riskyOperation() throws IOException {
    // Throws IOException
    FileReader reader = new FileReader("file.txt");
  }
}

// Usage
try {
  AgeValidator validator = new AgeValidator();
  validator.validateAge(25); // OK
  validator.validateAge(-5); // Throws exception
} catch (IllegalArgumentException e) {
  System.out.println("Error: " + e.getMessage());
}
```

### Custom Exceptions

```java
// Custom exception class
public class InvalidEmailException extends Exception {
  public InvalidEmailException(String message) {
    super(message);
  }
}

public class EmailValidator {
  public void validateEmail(String email) throws InvalidEmailException {
    if (!email.contains("@")) {
      throw new InvalidEmailException("Invalid email format");
    }
  }
}

// Usage
try {
  EmailValidator validator = new EmailValidator();
  validator.validateEmail("invalid-email");
} catch (InvalidEmailException e) {
  System.out.println(e.getMessage());
}
```

### Common Exception Types

```java
// Checked Exceptions (must be caught or declared)
IOException
SQLException
ClassNotFoundException

// Unchecked Exceptions (can be ignored)
NullPointerException
ArithmeticException
ArrayIndexOutOfBoundsException
ClassCastException
IllegalArgumentException
NumberFormatException
```

---

## Generics

### Generic Classes

```java
// Generic class with type parameter T
public class Container<T> {
  private T value;

  public void set(T value) {
    this.value = value;
  }

  public T get() {
    return value;
  }
}

// Usage
Container<String> stringContainer = new Container<>();
stringContainer.set("Hello");
String value = stringContainer.get(); // No casting needed

Container<Integer> intContainer = new Container<>();
intContainer.set(42);
Integer number = intContainer.get();
```

### Generic Methods

```java
public class Utils {
  // Generic method
  public static <T> void printArray(T[] array) {
    for (T element : array) {
      System.out.println(element);
    }
  }

  // Multiple type parameters
  public static <T, U> void print(T first, U second) {
    System.out.println(first + ", " + second);
  }
}

// Usage
Integer[] numbers = {1, 2, 3};
Utils.printArray(numbers);

String[] strings = {"A", "B", "C"};
Utils.printArray(strings);

Utils.print("Hello", 42);
```

### Bounded Type Parameters

```java
public class NumberContainer<T extends Number> {
  private T value;

  public void set(T value) {
    this.value = value;
  }

  public double doubleValue() {
    return value.doubleValue();
  }
}

// Usage
NumberContainer<Integer> intContainer = new NumberContainer<>();
intContainer.set(42);
System.out.println(intContainer.doubleValue()); // 42.0

// NumberContainer<String> invalid; // Compilation error
```

### Wildcards

```java
// Unbounded wildcard
public void printList(List<?> list) {
  for (Object item : list) {
    System.out.println(item);
  }
}

// Upper bounded wildcard
public void processNumbers(List<? extends Number> list) {
  for (Number num : list) {
    System.out.println(num);
  }
}

// Lower bounded wildcard
public void addIntegers(List<? super Integer> list) {
  list.add(42);
}

// Usage
List<String> strings = Arrays.asList("A", "B");
printList(strings);

List<Integer> integers = Arrays.asList(1, 2, 3);
processNumbers(integers);

List<Number> numbers = new ArrayList<>();
addIntegers(numbers);
```

---

## Lambda Expressions and Streams

### Lambda Expressions

```java
// Traditional anonymous class
Runnable task = new Runnable() {
  @Override
  public void run() {
    System.out.println("Running");
  }
};

// Lambda expression (Java 8+)
Runnable task2 = () -> System.out.println("Running");

// Lambda with parameter
Function<Integer, Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 25

// Lambda with multiple parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 8

// Lambda with multiple statements
BiFunction<Integer, Integer, Integer> calculate = (a, b) -> {
  int result = a + b;
  return result * 2;
};

// Comparator using lambda
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.sort((a, b) -> a.compareTo(b));
```

### Functional Interfaces

```java
// Built-in functional interfaces
@FunctionalInterface
interface Printer {
  void print(String message);
}

@FunctionalInterface
interface Converter<T, U> {
  U convert(T value);
}

// Using functional interfaces
Printer printer = (msg) -> System.out.println(msg);
printer.print("Hello");

Converter<String, Integer> converter = (str) -> Integer.parseInt(str);
System.out.println(converter.convert("42"));

// Common built-in functional interfaces
Function<Integer, Integer> function = x -> x * 2;
Predicate<Integer> predicate = x -> x > 10;
Consumer<String> consumer = System.out::println;
Supplier<String> supplier = () -> "Hello";
```

### Streams API

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

// Filter
List<Integer> evenNumbers = numbers.stream()
  .filter(n -> n % 2 == 0)
  .collect(Collectors.toList()); // [2, 4, 6]

// Map
List<Integer> squared = numbers.stream()
  .map(n -> n * n)
  .collect(Collectors.toList()); // [1, 4, 9, 16, 25, 36]

// Reduce
int sum = numbers.stream()
  .reduce(0, Integer::sum); // 21

// Sorted
List<Integer> sorted = numbers.stream()
  .sorted()
  .collect(Collectors.toList()); // [1, 2, 3, 4, 5, 6]

// Sorted in reverse
List<Integer> sortedDesc = numbers.stream()
  .sorted((a, b) -> b - a)
  .collect(Collectors.toList()); // [6, 5, 4, 3, 2, 1]

// Limit
List<Integer> firstThree = numbers.stream()
  .limit(3)
  .collect(Collectors.toList()); // [1, 2, 3]

// Skip
List<Integer> skipTwo = numbers.stream()
  .skip(2)
  .collect(Collectors.toList()); // [3, 4, 5, 6]

// FlatMap
List<List<Integer>> nestedList = Arrays.asList(
  Arrays.asList(1, 2),
  Arrays.asList(3, 4),
  Arrays.asList(5, 6)
);
List<Integer> flattened = nestedList.stream()
  .flatMap(List::stream)
  .collect(Collectors.toList()); // [1, 2, 3, 4, 5, 6]

// ForEach
numbers.stream()
  .forEach(System.out::println);

// Count
long count = numbers.stream()
  .filter(n -> n > 3)
  .count(); // 3

// Find first matching
numbers.stream()
  .filter(n -> n > 3)
  .findFirst()
  .ifPresent(System.out::println); // 4

// Any match
boolean hasEven = numbers.stream()
  .anyMatch(n -> n % 2 == 0); // true

// All match
boolean allPositive = numbers.stream()
  .allMatch(n -> n > 0); // true

// None match
boolean noNegative = numbers.stream()
  .noneMatch(n -> n < 0); // true
```

### Collectors

```java
List<String> fruits = Arrays.asList("apple", "banana", "apple", "cherry");

// To List
List<String> list = fruits.stream()
  .filter(f -> f.startsWith("a"))
  .collect(Collectors.toList());

// To Set
Set<String> set = fruits.stream()
  .collect(Collectors.toSet());

// To Map
Map<String, Integer> fruitLengths = fruits.stream()
  .collect(Collectors.toMap(
    fruit -> fruit,
    String::length
  ));

// GroupBy
Map<Integer, List<String>> groupedByLength = fruits.stream()
  .collect(Collectors.groupingBy(String::length));

// Joining
String joined = fruits.stream()
  .collect(Collectors.joining(", "));

// Counting
long count = fruits.stream()
  .collect(Collectors.counting());

// Partitioning
Map<Boolean, List<String>> partitioned = fruits.stream()
  .collect(Collectors.partitioningBy(f -> f.length() > 5));
```

---

## Java 8+ Features

### Default Methods in Interfaces

```java
public interface Animal {
  void eat();

  // Default method (Java 8+)
  default void sleep() {
    System.out.println("Sleeping");
  }
}

public class Dog implements Animal {
  @Override
  public void eat() {
    System.out.println("Eating");
  }
  // Can override or inherit sleep()
}
```

### Static Methods in Interfaces

```java
public interface Calculator {
  static int add(int a, int b) {
    return a + b;
  }

  static int subtract(int a, int b) {
    return a - b;
  }
}

// Usage
Calculator.add(5, 3); // 8
Calculator.subtract(10, 4); // 6
```

### Method References

```java
// Reference to static method
Function<Integer, String> toString = String::valueOf;

// Reference to instance method
String text = "Hello";
Supplier<Integer> length = text::length;

// Reference to constructor
Function<String, ArrayList> listCreator = ArrayList::new;

// Use in streams
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(System.out::println);

List<String> upperNames = names.stream()
  .map(String::toUpperCase)
  .collect(Collectors.toList());
```

### Optional (Java 8+)

```java
// Creating Optional
Optional<String> value = Optional.of("Hello");
Optional<String> empty = Optional.empty();
Optional<String> nullable = Optional.ofNullable(null);

// Check if present
if (value.isPresent()) {
  System.out.println(value.get());
}

// Get with default
String result = empty.orElse("Default");
String result2 = empty.orElseGet(() -> "Default");

// Throw if empty
String result3 = empty.orElseThrow(() -> new RuntimeException("Empty!"));

// Map and filter
Optional<String> upper = value.map(String::toUpperCase);
Optional<String> filtered = value.filter(v -> v.startsWith("H"));

// Chain operations
value
  .filter(v -> v.length() > 2)
  .map(String::toUpperCase)
  .ifPresent(System.out::println);
```

### LocalDate and LocalTime (Java 8+)

```java
import java.time.*;

// Current date and time
LocalDate today = LocalDate.now();
LocalTime currentTime = LocalTime.now();
LocalDateTime now = LocalDateTime.now();

// Create specific date/time
LocalDate date = LocalDate.of(2024, 11, 17);
LocalTime time = LocalTime.of(14, 30, 0);
LocalDateTime dateTime = LocalDateTime.of(date, time);

// Manipulation
LocalDate nextMonth = today.plusMonths(1);
LocalDate nextYear = today.plusYears(1);
LocalDate tomorrow = today.plusDays(1);

// Comparison
boolean isBefore = today.isBefore(date);
boolean isAfter = today.isAfter(date);

// Format
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = today.format(formatter);

LocalDate parsed = LocalDate.parse("17-11-2024", formatter);
```

---

## File I/O

### Reading Files

```java
import java.io.*;
import java.nio.file.*;

// Method 1: FileReader with BufferedReader
try (FileReader fileReader = new FileReader("file.txt");
     BufferedReader reader = new BufferedReader(fileReader)) {
  String line;
  while ((line = reader.readLine()) != null) {
    System.out.println(line);
  }
} catch (IOException e) {
  e.printStackTrace();
}

// Method 2: Files utility (Java 7+)
List<String> lines = Files.readAllLines(Paths.get("file.txt"));
lines.forEach(System.out::println);

// Method 3: Scanner
try (Scanner scanner = new Scanner(new File("file.txt"))) {
  while (scanner.hasNextLine()) {
    System.out.println(scanner.nextLine());
  }
} catch (FileNotFoundException e) {
  e.printStackTrace();
}
```

### Writing Files

```java
import java.io.*;
import java.nio.file.*;

// Method 1: FileWriter with BufferedWriter
try (FileWriter fileWriter = new FileWriter("output.txt");
     BufferedWriter writer = new BufferedWriter(fileWriter)) {
  writer.write("Hello, World!");
  writer.newLine();
  writer.write("Line 2");
} catch (IOException e) {
  e.printStackTrace();
}

// Method 2: Files utility
List<String> lines = Arrays.asList("Line 1", "Line 2", "Line 3");
Files.write(Paths.get("output.txt"), lines);

// Method 3: PrintWriter
try (PrintWriter writer = new PrintWriter("output.txt")) {
  writer.println("Hello");
  writer.println("World");
} catch (FileNotFoundException e) {
  e.printStackTrace();
}
```

### Working with Directories

```java
import java.nio.file.*;

// List files
Files.list(Paths.get("."))
  .forEach(System.out::println);

// List files recursively
Files.walk(Paths.get("."))
  .forEach(System.out::println);

// Create directory
Files.createDirectory(Paths.get("newDir"));
Files.createDirectories(Paths.get("path/to/nested/dir"));

// Check if file exists
boolean exists = Files.exists(Paths.get("file.txt"));

// Delete file
Files.delete(Paths.get("file.txt"));

// Copy file
Files.copy(Paths.get("source.txt"), Paths.get("dest.txt"));

// Move file
Files.move(Paths.get("old.txt"), Paths.get("new.txt"));
```

---

## Annotations

### Using Built-in Annotations

```java
public class Parent {
  public void method() {
    System.out.println("Parent method");
  }
}

public class Child extends Parent {
  // Deprecated annotation
  @Deprecated
  public void oldMethod() {
    System.out.println("This method is deprecated");
  }

  // Override annotation
  @Override
  public void method() {
    System.out.println("Child method");
  }

  // Suppress warnings
  @SuppressWarnings("unchecked")
  public void unsafeMethod() {
    List list = new ArrayList();
    list.add("String");
  }
}
```

### Creating Custom Annotations

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
  String value() default "default value";
  int count() default 0;
}

public class AnnotatedClass {
  @MyAnnotation(value = "test", count = 5)
  public void myMethod() {
    System.out.println("Annotated method");
  }
}

// Reading annotations at runtime
Method method = AnnotatedClass.class.getMethod("myMethod");
MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
System.out.println(annotation.value());
System.out.println(annotation.count());
```

---

## Summary: Key Concepts to Remember

### Do's ✅

- Use meaningful variable names
- Initialize variables before use
- Use try-with-resources for file operations
- Override `equals()` and `hashCode()` for custom objects
- Use immutable objects when possible
- Leverage streams for functional operations
- Use Optional instead of null checks
- Implement interfaces for contracts
- Use generics for type safety
- Follow naming conventions (camelCase for variables, PascalCase for classes)

### Don'ts ❌

- Don't use raw types with collections
- Don't ignore exceptions silently
- Don't create large objects in loops unnecessarily
- Don't use mutable static collections
- Don't mix checked and unchecked exceptions carelessly
- Don't violate DRY (Don't Repeat Yourself) principle
- Don't use `==` for string comparison
- Don't forget to close resources
- Don't create unnecessary objects
- Don't use public static mutable variables

---

## Quick Reference: Common Patterns

### Reading User Input

```java
// Using Scanner
Scanner scanner = new Scanner(System.in);
System.out.print("Enter your name: ");
String name = scanner.nextLine();

System.out.print("Enter your age: ");
int age = scanner.nextInt();
```

### Sorting Custom Objects

```java
class Student implements Comparable<Student> {
  private String name;
  private int age;

  @Override
  public int compareTo(Student other) {
    return Integer.compare(this.age, other.age);
  }
}

List<Student> students = new ArrayList<>();
Collections.sort(students); // Uses compareTo()

// Or with lambda
students.sort((s1, s2) -> Integer.compare(s1.getAge(), s2.getAge()));
```

### String Formatting

```java
// Old way
String formatted = "Name: " + name + ", Age: " + age;

// String.format()
String formatted2 = String.format("Name: %s, Age: %d", name, age);

// StringBuilder
StringBuilder sb = new StringBuilder();
sb.append("Name: ").append(name).append(", Age: ").append(age);
String formatted3 = sb.toString();
```

---

## Resources for Further Learning

1. **Official Documentation**

   - docs.oracle.com/en/java/
   - Java API Documentation

2. **Books**

   - "Effective Java" by Joshua Bloch
   - "Clean Code" by Robert C. Martin

3. **Online Platforms**

   - LeetCode
   - HackerRank
   - Codewars

4. **Courses**
   - Udemy Java courses
   - Pluralsight Java paths
   - YouTube Java tutorials

---

## Practice Exercises

1. Create a `Person` class with proper encapsulation
2. Implement a custom `List` using arrays
3. Create a `Calculator` interface with multiple implementations
4. Use streams to process a large dataset
5. Write a program that reads and writes files
6. Implement exception handling for various scenarios
7. Create a generic `Stack` class
8. Use annotations to create a custom logger

---

Good luck with your Java learning journey! Master these fundamentals thoroughly before moving to Spring Framework. The concepts here form the foundation for understanding Spring's powerful features. 🚀
