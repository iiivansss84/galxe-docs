---
sidebar_label: How to setup push-based API credential
sidebar_position: 1
slug: api-cred-items-update
---

# Use API to update credential Items

API credential allows you to push credential data to Galxe for your users.

## Prerequisite

1. First you need to create a credential that you want to use in your campaign. Please contact the galaxy BD team if you have no access/experience with galaxy dashboard before, they will help you to walk through the concepts of Galxe dashboard and give you access.
2. Then you will need an access token bound to your wallet address to use this API. Please go to the Galxe user setting page to generate an access token.

**NOTE**
Galxe backend has a rate litmit. If request volume is large, please contact us with your external IP so that we can put you on the whitelist. Otherwise, you will encounter 403 error when rate limit is reached.

## Endpoint

## Input

1. (header, string, mandatory) access-token: use to auth if you have access to update credential items, the user with this access token must be the credential curator
2. (int, mandatory) credId: the credential id you want to update
3. (enum string, mandatory) operation:
   1. APPEND, append items in the list.
   2. REPLACE (use only when there is very little addresses in total, e.g., less than 500), remove all items and replace them with items in the list. This operation is not transactional, so DO NOT use if you have a long list of addresses. It is recommended to call REMOVE first then use APPEND.
   3. REMOVE, remove items from the list.
4. (string array, mandatory) items: items list(address or email) to be modified, refer to operation. **The length of array must not exceed 500**.

NOTE:
Always retry when any error is returned. See below example of how to check both GraphQL and HTTP errors.
APPEND and REMOVE are re-entrant, so it is ok to just send the query again. REPLACE is not recommended to use.

### GraphQL

```graphql
mutation {
  credentialItems(
    input: {
      credId: "312"
      operation: APPEND
      items: [
        "0x111fd6240381af2c5f1a9e27f282bae8b92b257"
        "0x222dde76Cf5752f2bc1DC798BA1369dcA49d7c79"
        "0x333eC1a5d0BC3C4291aeb962CBda49677E9a9FcB"
        "0x444022af64bfc0f59ce1069e4ab51aa15148e60b"
        "0x55526ef96b12fba7a507afba39bdfc78e0039742"
        "0x6662c6b59e87b302b43400303427acd50f8071e6"
        "0x777742ee649ee36edcf5ac9a97df34333a97fd24"
        "0x8886b92fda46b8d9d33ca28d8837e1661edf8b97"
        "0x999886e265cf2ec39f8868d7b6c67ab78e027736"
      ]
    }
  ) {
    eligible(address:"0x999886e265cf2ec39f8868d7b6c67ab78e027736")
  }
}
```

## Examples

### Node.js

```typescript
const credId = "123";
const operation = "APPEND";
const items = ["0x123"];

// Nodejs using Axios lib
let result = await axios.post("https://graphigo.prd.galaxy.eco/query", {
  operationName: "credentialItems",
  query: `
    mutation credentialItems($credId: ID!, $operation: Operation!, $items: [String!]!) 
      { 
        credentialItems(input: { 
          credId: $credId 
          operation: $operation 
          items: $items 
        }) 
        { 
          name 
        } 
      }
    `,
    variables: {
      // Make sure this is string type as int might cause overflow
      credId: credId,
      operation: operation,
      items: items
    },
  },
  {
    headers: {
      "access-token": "access-token-of-yours",
    }
  }
);
if (result.status != 200) {
  throw new Error(result);
} else if (result.errors && result.errors.length > 0) {
  // NOTE: GraphQL returns 200 even if there's an error,
  // so must explicitly check result.errors.
  console.log(result.errors);
  throw new Error(result.errors);
}
```
