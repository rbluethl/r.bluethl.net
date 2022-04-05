## How to implement better APIs

### TL;DR

I created "reference implementation" based on my recent post on [How to design better APIs](https://r.bluethl.net/how-to-design-better-apis). I tried my best to practically implement all tips mentioned in the article. It's built with Next.js. [Check it out here](https://github.com/rbluethl/better-apis).


### Walk-through Video

%[https://www.youtube.com/watch?v=H3YLiCtGits]

### Preface

On March 3rd 2022, I published an article on [How to design better APIs](https://r.bluethl.net/how-to-design-better-apis) on my [Hashnode blog](https://r.bluethl.net). For some reason, it got quite popular — by the time of writing this, it has over 325 reactions, 34 comments (including my answers) and 112k+ views. It even made it to the [front page of Hacker News](https://news.ycombinator.com/item?id=30647784) on March 12th, which is kind of a big deal for me.

What I’m even more happy about, however, is that so many people reached out to me, said that the post was helpful and that it really provided value to them. I appreciate that a lot and I’ll do my best to continue creating useful content in the future, so thank you for reading this. 🙂

Since the tips mentioned in the post were meant to be „language-agnostic“, a couple of folks also asked if there is a project where they can look up an actual real-world implementation. That’s when I thought it would be a nice idea to complement the blog post with an actual project where most of the tips are applied in practice. So here it is! The API is, of course, overly simplistic (no database, no real authentication), but you should get an idea of all the concepts.

### How does it work?

The project is built with [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction) and TypeScript. I created one `/users` resource and tried to compress all 15 tips (including pagination, error handling, ...) into a few routes. You can have a look at the repository and the entire source code on my GitHub account. [Clone it here](https://github.com/rbluethl/better-apis). If you have any questions, just [shoot me a DM on Twitter](https://twitter.com/rbluethl). I'm always happy to help and discuss.

The project is basically structured like this:

- `pages/api/v1*` — The API
- `models/*` — Request and response types
- `services/*` — Fake data services

In order to keep everything focused on the API itself, I didn't use a real database. Instead, I "faked" the data access logic and used services that return test data.

Following, I will repeat the original blog post's tips and add detailed information on how they're implemented. You can click most of the headers and jump right into the code on GitHub.

There is also a [Postman collection](https://github.com/rbluethl/better-apis/blob/main/Better%20APIs.postman_collection.json) checked into the project, where you can play with all API routes.

### 1. Be consistent

I use the same naming conventions, resource names, HTTP methods and status codes across the entire project.

- `snake_case` naming in requests and responses
- `POST` for creating
- `GET` for reading
- `PATCH` for updating
- `DELETE` for deleting
- `200` for success
- `201` for creation
- `400` for client errors
- `401` for unauthorized
- `404` for not found
- `405` for method not allowed

### [2. Use ISO 8601 UTC dates](https://github.com/rbluethl/better-apis/blob/main/services/users.ts#L31)

I use ISO 8601 UTC dates (more precisely, the specific [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) format, providing date and time). 

```javascript
  const now = new Date()
  return {
    id,
    name,
    email,
    created_at: now.toISOString(),
    modified_at: now.toISOString()
  }
```

### [3. Make an exception for public endpoints](https://github.com/rbluethl/better-apis/blob/main/pages/_middleware.tsx)

I added a middleware that requires an API key to be present for each API route.

```javascript
import { NextResponse, NextRequest } from 'next/server'
import { unauthorized } from 'models/error'

export async function middleware(req: NextRequest, res: NextResponse) {
  const { pathname } = req.nextUrl

  if (pathname.startsWith('/api') && !req.headers.get('Api-Key')) {
    return new Response(JSON.stringify(unauthorized), {
      status: 401, headers: {
        'Content-Type': 'application/json'
      }
    })
  }

  return NextResponse.next()
}
```

### [4. Provide a health check endpoint](https://github.com/rbluethl/better-apis/blob/main/pages/api/v1/status/index.ts#L6)

I added a `GET /api/v1/status` endpoint that always returns `200`. 

```javascript
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  // GET /api/v1/status
  if (req.method === 'GET') {
    res.status(200).end()
  }

  res.status(405).end()
}
```

### [5. Version the API](https://github.com/rbluethl/better-apis/tree/main/pages/api/v1)

All API routes live inside the v1 folder, which automatically adds the `/v1` version to every resource within that folder. I also added a `/v2` version of the status endpoint to give you an idea on how another version would look like.

### [6. Accept API key authentication](https://github.com/rbluethl/better-apis/blob/main/pages/_middleware.tsx)

The middleware is used to determine the presence of an `Api-Key` HTTP header which could then be used to fetch an account with the given API key from the database. In our case, we only check whether this header is provided, otherwise the API will return a 401.

### 7. Use reasonable HTTP status codes

I'm using a very small set of HTTP status codes (200, 201, 400, 401, 404, 405). See #1 for details.

### 8. Use reasonable HTTP methods

I'm using a very small set of HTTP methods (`POST`, `GET`, `PATCH`, `DELETE`). See #1 for details.

### 9. Use self-explanatory, simple names

All endpoints and names are self-explanatory.

- `POST /users`
- `GET /users`
- `GET /users/{id}`
- `PATCH /users/{id}`
- `DELETE /users/{id}`

### [10. Use standardized error responses](https://github.com/rbluethl/better-apis/blob/main/models/error.ts)

I added an error response type that is always used when something wrong happens (4xx).

```javascript
export type Error = {
  code: string
  message: string
}
```

### [11. Return created resources upon POST](https://github.com/rbluethl/better-apis/blob/main/pages/api/v1/users/index.ts#L30)

After a resource is created, the most recent "snapshot" of that object is returned from the API.

### [12. Prefer PATCH over PUT](https://github.com/rbluethl/better-apis/blob/main/pages/api/v1/users/%5Bid%5D.ts#L23)

I don't use any `PUT` requests. Updates are always designed around `PATCH`.

### 13. Be as specific as possible

All endpoints are limited to the essentials and don't do anything unexpected. 

### [14. Use pagination](https://github.com/rbluethl/better-apis/blob/main/pages/api/v1/users/index.ts#L35)

I added a `Paged` type that wraps GET requests into a standardized paginated response.

### [15. Allow expanding resources](https://github.com/rbluethl/better-apis/blob/main/pages/api/v1/users/index.ts#L38)

It's possible to load `orders` when fetching users using the `expand` query parameter.

