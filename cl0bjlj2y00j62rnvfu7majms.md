## How to design better APIs

APIs are awesome, but they're also extremely hard to design. When creating an API from scratch, you need to get many details right. From basic security considerations to using the right HTTP methods, implementing authentication, deciding which requests and responses you should accept and return, ... the list goes on.

In this post, I'm trying my best to compress everything I know about what makes a good API. An API, that your consumers will enjoy using. All tips are language-agnostic, so they apply to any framework or technology.

### 1. Be consistent

I know it sounds logical, but it's hard to get this one right. The best APIs I know are *predictable*. When a consumer uses and understands one endpoint, they can expect that another endpoint works the same way. This is important for the entire API and one of the key indicators of whether or not an API is well-designed and great to use.

* Use the same casing for fields, resources, and parameters (I prefer `snake_case`)
* Use either plural or singular resource names (I prefer plural)
  * `/users/{id}`, `/orders/{id}` or `/user/{id}`, `/order/{id}`
* Use the same authentication and authorization methods for all endpoints
* Use the same HTTP headers across the API
  * For example `Api-Key` for passing an API key
* Use the same HTTP status codes based on the type of response
  * For example `404` when a resource can not be found
* Use the same HTTP methods for the same kind of actions
  * For example `DELETE` when deleting a resource

### 2. Use [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) UTC dates

When dealing with date and time, APIs should always return ISO 8601-formatted strings. Displaying dates in a specific time zone is generally a concern of client applications.

```JSON
{
    "published_at": "2022-03-03T21:59:08Z"
}
```

### 3. Make an exception for public endpoints

Every endpoint should require authorization *by default*. Most endpoints require an authenticated user to be called, so making this the default makes sense. If an endpoint needs to be called publicly, explicitly set this endpoint to allow unauthorized requests.

### 4. Provide a health check endpoint

Provide an endpoint (for example `GET /health`) that determines whether or not a service is healthy. This endpoint can be called by other applications such as load balancers to act in case of a service outage.

### 5. Version the API

Make sure you version your API and pass the version on each request so that consumers aren't affected by any changes to another version. API versions can be passed using HTTP headers or query/path parameters. Even the first version of the API (1.0) should be explicitly versioned.

Some examples:

* `https://api.averagecompany.com/v1/health`
* `https://api.averagecompany.com/health?api_version=1.0`

### 6. Accept API key authentication

If an API needs to be called by a third party, it makes sense to allow authentication via API keys. API keys should be passed using a custom HTTP header (such as `Api-Key`). They should have an expiration date, and it must be possible to revoke active keys so that they can be invalidated in case they get compromised. Avoid checking API keys into source control (use environment variables instead).

### 7. Use reasonable HTTP status codes

Use conventional HTTP status codes to indicate the success or failure of a request. Don't use too many, and use the same status codes for the same outcomes across the API. Some examples:

* `200` for general success
* `201` for successful creation
* `400` for bad requests from the client
* `401` for unauthorized requests
* `403` for missing permissions
* `404` for missing resources
* `429` for too many requests
* `5xx` for internal errors (these should be avoided at all costs) 

### 8. Use reasonable HTTP methods

There are many HTTP methods, but the most important ones are:

* `POST` for creating resources
  * `POST /users`
* `GET` for reading resources (both single resources and collections)
  * `GET /users`
  * `GET /users/{id}`
* `PATCH` for applying partial updates to a resource
  * `PATCH /users/{id}`
* `PUT` for applying full updates to a resource (replaces the current resource)
  * `PUT /users/{id}`
* `DELETE` for deleting resources
  * `DELETE /users/{id}`

### 9. Use self-explanatory, simple names

Most endpoints are resource-oriented and should be named that way. Don't add unnecessary information that can be inferred from elsewhere. This also applies to field names.

✅ **GOOD**
* `GET /users` => Retrieve users
* `DELETE /users/{id}` => Delete a user
* `POST /users/{id}/notifications` => Create a notification for a specific user
* `user.first_name`
* `order.number`

❌ **BAD**

* `GET /getUser`
* `POST /updateUser`
* `POST /notification/user`
* `order.ordernumber`
* `user.firstName`

### 10. Use standardized error responses

Aside from using HTTP status codes that indicate the outcome of the request (success or error), when returning errors, always use a *standardized* error response that includes more detailed information on what went wrong. Consumers can always expect the same structure and act accordingly.

```JSON
// Request => GET /users/4TL011ax

// Response <= 404 Not Found
{
	"code": "user/not_found",
	"message": "A user with the ID 4TL011ax could not be found."
}
``` 

```JSON
// Request => POST /users
{
    "name": "John Doe"
}

// Response <= 400 Bad Request
{
	"code": "user/email_required",
	"message": "The parameter [email] is required."
}
``` 

### 11. Return created resources upon `POST`

It's a good idea to return the created resource after creating it with a `POST` request. That's mostly important because the returned, created resource will reflect the current state of the underlying data source and will contain more recent information (such as a generated ID).

```JSON
// Request: POST /users
{
	"email": "jdoe@averagecompany.com",
	"name": "John Doe"
}

// Response
{
	"id": "T9hoBuuTL4",
	"email": "jdoe@averagecompany.com",
	"name": "John Doe"
}
```

### 12. Prefer `PATCH` over `PUT`

As mentioned earlier, `PATCH` requests should apply partial updates to a resource, whereas `PUT` replaces an existing resource entirely. It's usually a good idea to design updates around PATCH requests, because:

* When using `PUT` to update only a subset of fields for a resource, the entire resource still needs to be passed, which makes it more network-intensive and error-prone
* It's also quite dangerous to allow any field to be updated without any restrictions
* From my experience, there barely exist any use cases in practice where a full update on a resource would make sense
* Imagine an `order` resource that has an `id` and a `state`
* It would be very dangerous to allow consumers to update the `state` of an `order`
* A state change would much more likely be triggered by another endpoint (such as `/orders/{id}/fulfill`)

### 13. Be as specific as possible

As outlined in the previous section, it's generally a good idea to be *as specific as possible* when designing endpoints, naming fields, and deciding which requests and responses to accept. If a `PATCH` request only accepts two fields (`name` and `description`), there is no danger of using it incorrectly and corrupting data.

### 14. Use pagination

Paginate all requests that return a collection of resources and use the same response structure. Use `page_number` and `page_size` (or similar) to control which chunk you want to retrieve. 

```JSON
// Request => GET /users?page_number=1&page_size=15

// Response <= 200 OK
{
	"page_number": 1,
	"page_size": 15,
	"count": 378,
	"data": [
        // resources
    ],
	"total_pages": 26,
	"has_previous_page": true,
	"has_next_page": true
}
```

### 15. Allow expanding resources

Allow consumers to load related data using a query parameter called `expand` (or similar). This is especially useful to avoid round-trips and load all data that is necessary for a specific action in one go.

```JSON
// Request => GET /users/T9hoBuuTL4?expand=orders&expand=orders.items

// Response <= 200 OK
{
  "id": "T9hoBuuTL4",
  "email": "jdoe@averagecompany.com",
  "name": "John Doe",
  "orders": [
    {
      "id": "Hy3SSXU1PF",
      "items": [
        {
          "name": "API course"
        },
        {
          "name": "iPhone 13"
        }
      ]
    },
    {
      "id": "bx1zKmJLI6",
      "items": [
        {
          "name": "SaaS subscription"
        }
      ]
    }
  ]
}
```

If you enjoyed this post, feel free to [follow me on Twitter](https://twitter.com/rbluethl).