# Typescript Snippets

### Table of contents

* [Generic constraints](#generic-constraints)
* [Lookup types](#lookup-types)

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
