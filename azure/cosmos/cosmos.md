# Cosmos learnings

## Performance

Best practices says that you should make `CosmosClient` a singleton but it was unclear to me on what level the tcp connection pool was handled. `HttpClientFactory` has the pattern where the factory should be a singleton and then you call `.GetClient("ClientName")` to get a client each time you want to make a http request.

So should you keep a reference to the singleton instance of `CosmosClient` and call `.GetContainer("DbName", "ContainerName")`? No, not according to [the answer from this MSFT authority](https://stackoverflow.com/questions/62383219/can-cosmos-sdk3-container-be-a-singleton):

> Yes, Container instance is just a proxy, nothing more. There is no overhead or network calls when calling GetContainer, so you can keep a single instance and it will be thread-safe
> ~ Matias Quaranta

## PATCH

You can't add more than 1 level when PATCHing a document. Let's say you have this document in the db:

```jsonc
{
  "id": 123,
  "contactDetails": null
}
```

And then you add an endpoint for `PUT /user/{id}/phone` and try to update the Cosmos document with this `PATCH` payload:

```json
[
  { "op": "add", "path": "/contactDetails/phone", "value": "555-123" }
]
```

This will result in an error since the `phone` property is not already present. In other words Cosmos will not create the path for you in the same way that `Azure AI Search` does.