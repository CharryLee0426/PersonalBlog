# Typescript Crash Cource

> store my index.ts temporarily

```typescript
// Basic types
let id: number = 5
let company: string = "Apple Inc."
let isPublished: boolean = true
let x: any = "hello"                // when you use 'any' type, you actually not declare a type😅

// Array
let ids: number[] = [1, 2, 3, 4, 5]
let arr: any[] = [1, true, "hello"]

// Tuple
let person: [number, boolean, string] = [1, true, "Charry"]
// Tuple array
let employee: [number, string][]

employee = [
    [1, "Tom"],
    [2, "Jerry"],
]

// Union
let productID: string | number
productID = 22
productID = "iPhone📱"

// Enum
enum directions {       // By default
    Up,                 // 0
    Down,               // 1
    Left,               // 2
    Right,              // 3
}

// Object
// you can also define your own types.
type User = {
    id: number,
    name: string
}

const user: User = {
    id: 1,
    name: "John"
}

// Type assertion
let cid: any  = 1
// let customerID = <number>cid
let customerID = cid as number
customerID = 2

// Function
function addNum(x: number, y: number): number {
    return x + y
}
// void
function log(message: string | number): void {
    console.log(message)
}

// Interfaces
interface userInterface {
    readonly id: number,            // read only property just can be initialized but not be changed
    name: string,
    age: number
}

const newUser: userInterface = {
    id: 1,
    name: "John",
    age: 18
}

// newUser.id = 4                   // Error: because id is a read-only property

interface mathFunc {
    (x: number, y: number): number
}

// attention that they are not closures.
const add: mathFunc = (x: number, y: number) => x + y
const sub: mathFunc = (x: number, y: number) => x - y

// Class
// it's very similar with Java.
interface PersonInterface {
    id: number
    name: string
    register(): string
}

class Person implements PersonInterface {
    // private, protected, public(default)
    id: number
    name: string

    // constructor is responsible for initialization
    constructor(id: number, name: string) {
        this.id = id
        this.name = name
    }

    // method
    register() {
        return `${this.name} is registered now.`
    }
}

class Employee extends Person {
    // add more properties
    department: string

    constructor(id: number, name: string, department: string) {
        super(id, name)
        this.department = department
    }
}

const jackson = new Employee(1, "Jackson", "Dep  of Fin")
console.log(jackson)

// Generics "泛型" in Chinese
function getArray<T>(items: T[]): T[] {
    return new Array().concat(items)
}

let numArray = getArray<number>([1, 2, 3, 4])
let strArray = getArray<string>(["hello", "world"])
// strArray.push(1)                                 // error
```