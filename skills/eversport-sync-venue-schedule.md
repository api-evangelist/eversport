---
name: Sync a venue's class and course schedule
description: Pull a venue's upcoming activities (classes, courses, workshops, events) from the Eversports Provider API and surface bookable checkout links.
api: graphql/eversport-provider-api.graphql
operations: [venues, venue, activities, activityGroups, activityGroup]
---

# Sync a venue's class and course schedule

The Eversports Provider API is a **read-only GraphQL** endpoint at
`https://provider-api.eversportsmanager.io/api/graphql`. All requests require
provider-issued credentials in a request header (see
`authentication/eversport-authentication.yml`). Bookings are **not** made through
this API — surface each activity's `bookable.checkoutURL` so the customer
completes the booking on the Eversports Checkout.

## Steps

1. **Find the venues you have access to.** Query `venues(first, after)` and read
   `nodes { id name location { city street } }`. Paginate with `pageInfo.hasNextPage`
   and `pageInfo.endCursor` (pass it back as `after`) — this is a Relay cursor
   connection (see `conventions/eversport-conventions.yml`).

2. **(Optional) Confirm a single venue.** Query `venue(id)` to read its
   `company`, `teachers`, and `images`.

3. **List scheduled activities for the venue over a time window.** Query
   `activities(venueIds: [ID], timeRange: { start, end }, first, after)`. Useful
   filters: `sportIds`, `teacherIds`, `roomIds`, `activityGroupTypes`,
   `isCancelled`, `hasOnlineStream`. Read
   `nodes { id name end isCancelled limited { freeSpots totalSpots } bookable { checkoutURL cancellationDeadline bookableWindowStart bookableWindowEnd } activityGroup { id name type } }`.

4. **(Optional) Group context.** To present series-level metadata (description,
   images, sport, publication state) query `activityGroups(venueIds, timeRange)`
   or `activityGroup(id)`.

5. **Present availability + checkout.** For each activity show name, start/end,
   `limited.freeSpots`, and link the customer to `bookable.checkoutURL`.

## Rules

- Read-only: there are no mutations. Never attempt to write bookings.
- Paginate every list with `first` + `after`; do not assume a single page.
- Respect fair-use — the API is for scheduled sync, not real-time polling
  (see `lifecycle/eversport-lifecycle.yml`).
- Handle errors from the top-level GraphQL `errors[]` array; check
  `extensions.code` (see `errors/eversport-problem-types.yml`).
