# Typescript Snippets

### Table of contents

* [Lookup types](#lookup-types)

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
