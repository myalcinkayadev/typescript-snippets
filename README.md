# Typescript Snippets

### Table of contents
* [Interface declaration merging](#interface-declaration-merging)
* [Generic constraints](#generic-constraints)
* [Typeof type operator](#typeof-type-operator)
* [Lookup types](#lookup-types)

#### Interface declaration merging
```
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

#### Generic constraints
```
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
```
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
```
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
