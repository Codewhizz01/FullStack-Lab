-----------------------------Day-9 TASK Objects & Array Methods-----------------------------------------



## What Was Covered

### 1. Objects
- An object is a collection of related data stored as key-value pairs.
- Example: a student's details (name, age, city, marks) grouped in one place instead of separate variables.
- Objects are used because they keep code readable (keys explain the data), reusable, and maintainable.
- Used everywhere in real apps: users, products, orders, employees.

### 2. Accessing Properties
- **Dot notation:** `student.name` — the everyday, default choice.
- **Bracket notation:** `student["age"]` — required when the key is stored in a variable, or contains spaces/special characters.

### 3. Add, Update, Delete Properties
- **Add:** `student.school = "ABC School";`
- **Update:** `student.marks = 95;`
- **Delete:** `delete student.city;`

### 4. Methods and `this`
- A method is a function stored inside an object.
- Inside a method, `this` refers to the object itself, allowing it to access its own properties.
- Example: `greet() { console.log("Hello " + this.name); }`

### 5. Looping Through an Object
- `for...in` loop visits every key in the object.
- Three built-in helpers convert an object into arrays:
  | Method | Returns |
  |---|---|
  | `Object.keys(obj)` | array of keys |
  | `Object.values(obj)` | array of values |
  | `Object.entries(obj)` | array of [key, value] pairs |

### 6. Nested Objects
- An object can contain another object inside it.
- Accessed by chaining dots: `student.address.city`

### 7. Arrays of Objects
- An array stores multiple values in order.
- An array of objects is the shape most real app data takes — a list of products, users, or orders.
- Confirmed by observing a live network call on shorterloop.com in DevTools: the API response came back exactly as an array of objects.

### 8. Array Methods

| Method | Returns | Use Case |
|---|---|---|
| `forEach` | nothing | run code on each item (e.g. print every name) |
| `map` | new array | transform every item (e.g. extract just names) |
| `filter` | new array | keep items matching a condition (e.g. price > 10000) |
| `find` | one item | first match only (e.g. find product by name) |
| `some` | true/false | does at least one item match? |
| `every` | true/false | do all items match? |
| `sort` | sorted array | reorder items (e.g. cheapest first) |
| `reduce` | one value | combine all items into a single total |

- **Key distinction:** `map`/`filter` return new arrays. `find` returns a single item. `some`/`every` return booleans. `reduce` returns one combined value. `forEach` returns nothing — it just executes.




## Interview Question Answers

**1. What is an object?**
A collection of related data stored as key-value pairs, grouped under one variable (e.g. a student's name, age, and marks kept together instead of separate variables).

**2. Difference between an object and an array?**
An object stores data as key-value pairs, accessed by key name (`student.name`). An array stores data as an ordered list, accessed by numeric index (`arr[0]`). Objects describe "things with properties"; arrays describe "ordered collections."

**3. Difference between dot and bracket notation?**
Dot notation (`student.name`) is the everyday choice — clean and readable. Bracket notation (`student["age"]`) is required when the key is stored in a variable, or contains spaces/special characters/starts with a number.

**4. What is the `this` keyword?**
Inside an object method, `this` refers to the object the method was called on, letting the method access that object's own properties (e.g. `this.name` inside `greet()`).

**5. Difference between `map()` and `forEach()`?**
`map()` returns a new array built from the transformed items. `forEach()` returns nothing — it just runs a function on each item (used for side effects like logging).

**6. Difference between `filter()` and `find()`?**
`filter()` returns a new array of *all* items matching a condition. `find()` returns only the *first* matching item (not an array), or `undefined` if none match.

**7. What does `reduce()` do?**
Boils an array down to a single value (sum, count, object, etc.) by running an accumulator function over each item, starting from an initial value.

**8. When do you use `some()` and `every()`?**
`some()` checks if *at least one* item satisfies a condition (returns true/false). `every()` checks if *all* items satisfy a condition (returns true/false). Used for validation checks — e.g. "is anything in stock" vs "is everything in stock."

// Day 9 - Practice Answers

// 1 & 2. Create Employee, Mobile, Car objects + add/update/delete
```
const employee = { name: "Aman", role: "Developer", salary: 50000 };
employee.dept = "Engineering";      // add
employee.salary = 55000;            // update
delete employee.role;               // delete
```
```

const mobile = { brand: "Samsung", model: "S23", price: 70000 };
mobile.color = "Black";             // add
mobile.price = 68000;               // update
delete mobile.model;                // delete
```
```
const car = { brand: "Honda", model: "City", year: 2022 };
car.fuel = "Petrol";                // add
car.year = 2023;                    // update
delete car.model;                   // delete
```

// 3. Loop through an object's properties
```
for (const key in employee) {
  console.log(key, employee[key]);
}
```

// 4. Array of 5 students
```
const students = [
  { id: 1, name: "Rahul", marks: 92 },
  { id: 2, name: "Aman", marks: 81 },
  { id: 3, name: "Riya", marks: 96 },
  { id: 4, name: "Neha", marks: 74 },
  { id: 5, name: "Karan", marks: 88 }
];
```

// 5. map -> all names
```
const names = students.map(s => s.name);
```

// 6. filter -> above 80 marks
```
const above80 = students.filter(s => s.marks > 80);
```
// 7. find -> search one student
```
const found = students.find(s => s.id === 2);
```

// 8. reduce -> total marks
```
const total = students.reduce((sum, s) => sum + s.marks, 0);
```

// 9. sort by marks
```
const sorted = [...students].sort((a, b) => a.marks - b.marks);
```

  ## Takeaways
- An object groups related data; an array is an ordered list of values.
- `map` and `filter` produce new arrays; `find` returns one item; `reduce` returns one value; `forEach` returns nothing.
- Real-world API data almost always arrives as an array of objects — these methods are the standard toolkit for working with it.

---