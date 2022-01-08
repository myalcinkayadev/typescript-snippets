# Typescript Snippets

### Playground
https://www.typescriptlang.org/play

### Table of contents
* [Interface declaration merging](#interface-declaration-merging)
* [Implements keyword](#implements-keyword)
* [Abstract class](#abstract-class)
* [Readonly arrays and tuples](#readonly-arrays-and-tuples)
* [This parameter](#this-parameter)
* [Generic constraints](#generic-constraints)
* [Typeof type operator](#typeof-type-operator)
* [Lookup types](#lookup-types)
* [Keyof type operator](#keyof-type-operator)

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
