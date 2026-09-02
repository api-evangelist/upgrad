---
name: Read learner reading-time leaderboards
description: Read current-day reading-time leaderboards for a course or a learner, and read or record micro-interaction
  notifications.
api: openapi/upgrad-learner-analytics-openapi.yml
operations:
- getUserReadingTimeLeaderBoardForCurrentDay
- getUserReadingTimeLeaderBoardForCurrentDay_1
- getUserMicroInteractionNotification
- saveMicroInteractionNotification
---

# Read learner reading-time leaderboards

## When to use this
Use this skill to read engagement signal out of the upGrad learning platform — who is reading, how much, and what nudges have fired.

## Before you start
- Base URL: `https://learner-analytics-rest.upgrad.com`
- The contract declares an `Authorization: Bearer <JWT>` scheme but applies it to **no operation**, so the spec does not actually state what is protected. Send the bearer token; expect 401/403 without it.
- Leaderboards are **current-day only**. There is no historical or date-ranged variant in the contract.

## Steps
1. **Course leaderboard** — `GET /leaderboard/reading-time/daily/course/{courseId}` (`getUserReadingTimeLeaderBoardForCurrentDay`). Paginate with the optional `pageNumber` and `pageSize` query parameters.
2. **One learner's rank** — `GET /leaderboard/reading-time/daily/user/{userId}` (`getUserReadingTimeLeaderBoardForCurrentDay_1`), with the required `courseId` query parameter. Note the operationId carries a `_1` suffix — springdoc generated it from an overloaded Java method, and it is the id you must use.
3. **Read micro-interaction notifications** — `GET /micro-interaction-notification/{microInteractionType}` (`getUserMicroInteractionNotification`), with required `userId` and optional `notificationDate`.
4. **Record a micro-interaction** — `POST /micro-interaction-notification` (`saveMicroInteractionNotification`). This is the only write in this skill and it has no reversal operation.

## Errors
This service declares 400, 403, 404, 409, 422, 500 and returns `ErrorContext` (`status`, `messages`) under `*/*`. It declares **no 429**, so no throttling contract is stated — still back off on repeated failures.
