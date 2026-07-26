---
name: Build a directory of venues and teachers
description: Enumerate the companies, venues, and teachers available to an Eversports Provider API integration to build a directory or profile pages.
api: graphql/eversport-provider-api.graphql
operations: [companies, venues, venue, teachers, teacher]
---

# Build a directory of venues and teachers

Uses the read-only Eversports Provider API GraphQL endpoint at
`https://provider-api.eversportsmanager.io/api/graphql` with provider credentials
in a request header (see `authentication/eversport-authentication.yml`).

## Steps

1. **List companies (top-level accounts).** Query `companies(first, after)` and
   read `nodes { id name venues { id name } }`. Paginate with `pageInfo`.

2. **List venues.** Query `venues(first, after)` and read
   `nodes { id name company { id name } location { name street zip city country type coordinates { latitude longitude } } images { url mimeType } }`.

3. **Fetch a single venue's teachers.** Query `venue(id)` and read
   `teachers { id name description education gender image { url } }`, or query
   `teachers(venueIds: [ID], first, after)` to page across venues.

4. **(Optional) Fetch a single teacher.** Query `teacher(id)` for
   `description`, `education`, `gender`, and `image`.

5. **Assemble the directory.** Join companies -> venues -> teachers into profile
   records; use `location.coordinates` for maps and `images` for cards.

## Rules

- Read-only; no mutations exist.
- Every list is a Relay cursor connection — always paginate with `first`/`after`.
- Do not fabricate teacher or venue fields; only the fields above exist in the
  schema (`graphql/eversport-provider-api.graphql`).
- Inspect `errors[].extensions.code` for failures
  (see `errors/eversport-problem-types.yml`).
