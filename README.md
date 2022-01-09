# Typescript Snippets

### Playground
https://www.typescriptlang.org/play

### Table of contents
* [Interface declaration merging](#interface-declaration-merging)
* [Implements keyword](#implements-keyword)
* [User defined type guards](#user-defined-type-guards)
* [Abstract class](#abstract-class)
* [Readonly arrays and tuples](#readonly-arrays-and-tuples)
* [This parameter](#this-parameter)
* [Generic constraints](#generic-constraints)
* [Typeof type operator](#typeof-type-operator)
* [Lookup types](#lookup-types)
* [Keyof type operator](#keyof-type-operator)
* [Mapped type modifiers](#mapped-type-modifiers)

#### Interface declaration merging
```typescript
export interface Request {
  body: unknown;
}

// Add additional properties to the request interface
export interface Request {
  json: unknown;
}

// Our Application
function handleRequest(req: Request) {
  req.body;
  req.json;
}
```

#### Implements keyword
```typescript
type Animal = {
    name: string,
    voice(): string,
}

function log(animal: Animal) {
    console.log(`Animal ${animal.name}: ${animal.voice()}`);
}

class Cat implements Animal {
    constructor(public name: string) {}
    voice() { return 'meow 🙀'; }
}

class Dog implements Animal {
    constructor(public name: string) { }
    voice() { return 'woof 🐶'; }
}

log(new Cat('Sisi'));
log(new Dog('Lassie'));
```

#### User defined type guards
```typescript
type Square = {
    size: number,
};

type Rectangle = {
    width: number,
    height: number,
};

type Shape = Square | Rectangle;

/*
    These conditions are not explicitly clear that we are checking if something is a square or a rectangle
    if('size' in shape)  -> Square
    if('width' in shape) -> Rectangle
    
    We can improve the situation by adding a common member between sqaure and rectangle of different 
    little types, thus forming a *discriminated union*.
    
    type Sqaure {
        kind: 'square',
        size: number
    }

    However, if we do not want to modify the types themselves, we can still get the readability benefit
    by creating user defined type guards.

    Simply a function that returns a boolean value and is annotated in the form (parameter is type)

    function isSquare(shape: Shape): boolean {}
    function isSquare(shape: Shape): shape is Square {}
*/

// User defined type guard
function isSquare(shape: Shape): shape is Square {
    return 'size' in shape;
}

function isRectangle(shape: Shape): shape is Rectangle {
    return 'width' in shape;
}

function area(shape: Shape) {
    if(isSquare(shape)) {
        return shape.size * shape.size;
    }
    if(isRectangle(shape)) {
        return shape.width * shape.height;
    }
    const _ensure: never = shape;
    return _ensure;
}

const sqaure: Square = {
    size: 10
};
console.log(`area of the sqaure: ${area(sqaure)}`);
```

#### Abstract class
```typescript
abstract class Command {
    abstract commandLine(): string;

    execute() {
        console.log('Executing: ', this.commandLine());
    }
}

class GitResetCommand extends Command {
    commandLine() {
        return 'git reset --hard';
    }
}

class GitFetchCommand extends Command {
    commandLine() {
        return 'git fetch --all';
    }
}

new GitResetCommand().execute();
new GitFetchCommand().execute();
```

#### Readonly arrays and tuples
```typescript
/*
    Instead of declaring the input as a number array, we can declare it as a reaonly number array.
    Now any mutating methods will no longer be avaible on input.

    So if you want to use the sort and the reverse methods, the first thing that we will need to do is
    to create a copy of the input aray using the built in slice method or rest parameters.

    This creates a new non-readonly array which can then be safely sorted and reversed and then returned.
    And now when we invoke this reverseSorted method on the start variable, the start variable is no longer 
    mutated.
*/
function reverseSorted(input: readonly number[]): number[] {
    // return [...input].sort().reverse();
    return input.slice().sort().reverse();
}

const start = [1, 2, 3, 5, 4];
const result = reverseSorted(start);

console.log(`result: [${result}]`);   // [5, 4, 3, 2, 1]
console.log(`start:  [${start}]`);    // [1, 2, 3, 5, 4] 

// type Neat = readonly number[];
// type Long = ReadonlyArray<number>;

/*
    Tuple example
*/

type Point = readonly [number, number];

function move(point: Point, x: number, y: number): Point {
    /*
        Now any attempts to modify any given point will result in a compiler error.
        Here you can see that we cannot mutate the value.
        
        point[0] += x;
        point[1] += y;
    */

    /*
        So now we can fix this error by replacing mutating implementation with an implementation 
        that returns a new tuple created from the existing tuple, plus the moves in the x and y dimensions.
    */

   return [point[0] + x, point[1] + y];
}

const point: Point = [0, 0];
const moved = move(point, 10, 10);

console.log(moved);
console.log(point);
```

#### This parameter
```typescript
function double(this: { value: number }) {
    this.value = this.value * 2; // calling context
}

const valid = {
    value: 10,
    double,
}

valid.double();

console.log(valid.value);
```

#### Generic constraints
```typescript
type NameFields = { firstName: string, lastName: string };

function addFullName<T extends NameFields>(obj: T): T & { fullName: string } {
    return {
        ...obj,
        fullName: `${obj.firstName} ${obj.lastName}`
    }
}

const claude = addFullName({
    email: 'i-would-like-to-paint-the-way-a-bird-sings@claudemonet.fr',
    firstName: 'Claude',
    lastName: 'Monet'
});

console.log(claude.email);      // i-would-like-to-paint-the-way-a-bird-sings@claudemonet.fr
console.log(claude.fullName);   // Claude Monet

// const jane = addFullName({ firstName: 'Jane' }); // Compile-time error, missing field
```

#### Typeof type operator
```typescript
const center = {
  x: 0,
  y: 0,
  z: 0,
};

type Point = typeof center;

const unit: Point = {
    x: center.x + 1,
    y: center.y + 1,
    z: center.z + 1,
};

// json usage
const personResponse = [{
    "id": 1,
    "first_name": "Rory",
    "last_name": "Simonot",
    "email": "rsimonot0@howstuffworks.com",
    "ip_address": "165.250.182.98"
}];

type PersonResponse = typeof personResponse[0];

function processResponse(person: PersonResponse) {
    console.log(`${person.first_name} ${person.last_name}`);
}
```

#### Lookup types
```typescript
export type SubmitRequest = {
    // ... complex type
    personal: {
        previousAliases: {
            firstName: string,
            middleName: string,
            lastName: string,
        }[],
    },
    consent: {
        additionalInformation: string,
    },
    payment: {
        creditCardToken: string
    }
};

// Lookup types
type PaymentRequest = SubmitRequest['payment'];
type PreviousAliasRequest = SubmitRequest['personal']['previousAliases'][0];

export function getPayment(): PaymentRequest {
    return {
        creditCardToken: '124q234n12l!@$234234>'
    };
}
```

#### Keyof type operator
```typescript
type Person = {
    name: string,
    age: number,
    location: string,
};

const mimo: Person = {
    name: 'Mimo',
    age: 28,
    location: 'In the scandinavian folklore'
};

type PersonKeys = keyof Person;

/*
    We are restricting the keys to be from the person type, will annotate the obj to be of type person 
    as well.
*/
function logGet(obj: Person, key: keyof Person) {
    const value = obj[key];
    console.log('Getting:', key, value);
    return value;
}

logGet(mimo, 'location');
//logGet(mimo, 'what_the_hell_is_this'); // compile-time error

/*
    Notice that our logGet function doesn't have any code specific to the person type to make this 
    function more general.
    
    We can use generics using a *generic type* for the obj and a generic type for the key.
    And now the requirement that key must be something that is in the key of obj can be enforced 
    by a *generic constraint* when we define the *generic argument*.

    Now, with this *generic constraint* typescript knows that key will be something that is in 
    the key of obj.
    
    So when we try to index obj with key in our JavaScript code, TypeScript correctly infers the 
    type of the return value to be an index *lookup type* obj key.

    And since value is what we return from the logGet2 function, the return type of the function 
    is also inferred to be the *lookup type* object key.
*/

function logGet2<Obj, Key extends keyof Obj>(obj: Obj, key: Key) {
    const value = obj[key];
    console.log('Getting:', key, value);
    return value;
}

/*
    Now with this function signature in place when obj is of type person and key is of type age, 
    the return type will be the lookup person age and we can see that it needs to be a number.

    And indeed that is what is automatically inferred when we try to use it to get mimo's age.
*/
const age = logGet2(mimo, 'age');   // 28;
//logGet2(mimo, 'not_now_im_too_drunk'); // compile-time error

/*
    Now, similar to how we have the logGet function, we can easily apply the same knowledge to build 
    a log set function.
    
    The body of the function is pretty simple in terms of the function signature. It is very similar 
    to the signature of the logGet function.
    
    The only difference is that the log set function takes an additional third argument, which has been 
    annotated with a lookup type of key, and we've seen this type before as well.
    
    This is exactly the return type of the log function.
*/
function logSet<Obj, Key extends keyof Obj>(obj: Obj, key: Key, value: Obj[Key]) {
    console.log('Setting: ', key, value);
    obj[key] = value;

}

logSet(mimo, 'age', 30);
logGet2(mimo, 'age');

//logSet(mimo, 'ages', 36); // typo error
//logSet(mimo, 'age', '36'); // error: type string is not assignable to type number
```

#### Mapped type modifiers
```typescript
export type Point = {
    x: number,
    y: number,
    z: number,
};
/*
    We want this to be immutable.
*/
// const center: Point = {
//     x: 0,
//     y: 0,
//     z: 0,
// };

// center.x = 100; // should error

/*
    However, notice the code duplication between point and readonly point.
    We can get rid of this duplication by using mapped types.
*/
// type ReadonlyPoint = {
//     readonly x: number,
//     readonly y: number,
//     readonly z: number,
// };

// const center: ReadonlyPoint = {
//     x: 0,
//     y: 0,
//     z: 0,
// };

// center.x = 100; // should error

// type ReadonlyPoint = {
//     [Item in Union]: Output
// };

// type ReadonlyPoint = {
//     [Item in 'x' | 'y' | 'z']: number
// };

/*
    The great thing about mapped types is that you can add modifiers 
    that will apply to all the loop items.
*/

// type ReadonlyPoint = {
//     readonly [Item in 'x' | 'y' | 'z']: number
// };

// const center: ReadonlyPoint = {
//     x: 0,
//     y: 0,
//     z: 0,
// };

/*
    Now let's apply other features from TypeScript to clean.

    Instead of hand coding of this union of strings we can generated by applying 
    the key of operator to the pointer type.
*/

// Usage
// type ReadonlyPoint = {
//     readonly [Item in keyof Point]: number
// };

/*
    Additionally, instead of assuming that all of these members need to be of type number, 
    we can read the type of the individual members by using the loop variable to look up 
    the type of the member from point.
*/

// type ReadonlyPoint = {
//     readonly [Item in keyof Point]: Point[Item]
// };

/*
    As a final step
    We can create a utility type function by taking a generic argument of type T and using
    that instead of the hardcoded point type.
*/

// type Readonly<T> = {
//     readonly [Item in keyof T]: T[Item]
// };

/*
    And now we can use it to create read only version of any input type.
    And for our particular case, our input type will be the data point.
*/

const center: Readonly<Point> = {
    x: 0,
    y: 0,
    z: 0,
};

/*
    And at this juncture, I'm actually happy to inform you that you don't need to create this particular
    read on the utility type.

    It comes baked in as a part of the typescript of compiler, and if you delete a custom implementation, 
    it falls back to that one.

    And the internal implementation has been coded up exactly how we did it right now.
*/

// center.x = 100; // should error
```
