# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

- Presentation tier decision: Use the shared frontend API client for relative `/api` requests (honoring `VITE_API_BASE_URL` when set). React pages own presentation, including accessible loading, empty, and error states; they do not contain persistence or business rules.
- Logic tier decision: Keep HTTP handling and request validation in Express routes/services. Validate payloads and references before persistence, and respond with deliberate 4xx statuses for invalid input or missing resources rather than allowing cast errors to become 500s.
- Data tier decision: Use Mongoose models for required fields, defaults, and user/team references. Keep demo seeding idempotent and non-destructive; do not clear unrelated application data.
- Activity tracking approach: Accept `userId`, `teamId`, `type`, and positive `minutes`; validate ID shape and confirm both records exist. Persist the references as `user` and `team`, set `occurredAt` on creation, and calculate points deterministically at one point per minute, consistent with the seed records.
- Team management approach: Create and list teams while preserving `name`, `mascot`, and member IDs. Default omitted members to an empty array and reject a missing name with `400`.
- Leaderboard approach: Sum all activity points for each team, sort by total descending, and return each team name, total, and 1-based rank.
- Workout suggestions approach: Look up a user by email and recommend workouts matching their fitness goal, using recent activity to guide the focus when appropriate. Include a human-readable reason for each suggestion and return a deliberate 4xx when the user is not found.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
- API command: `npm run test:api --prefix octofit-tracker/backend`
- Frontend command: `npm test --prefix octofit-tracker/frontend`
- Backend build command: `npm run build --prefix octofit-tracker/backend`
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
