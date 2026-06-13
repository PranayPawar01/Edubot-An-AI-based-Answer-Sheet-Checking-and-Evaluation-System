# Edubot

AI-powered answer sheet evaluation platform — teachers create question papers with model answers; students upload answer sheets; GPT Vision extracts text via OCR; GPT evaluates answers with strict per-question scoring, feedback, strengths/weaknesses/suggestions.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/edubot run dev` — run the React frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 (`artifacts/api-server`) on port 8080
- Frontend: React + Vite (`artifacts/edubot`) on dynamic port
- DB: PostgreSQL + Drizzle ORM (`lib/db`)
- Auth: Clerk (managed Replit tenant) — `@clerk/express` server-side, `@clerk/react` client-side
- AI: OpenAI `gpt-5.1` for OCR (Vision) + evaluation via `artifacts/api-server/src/lib/ai.ts`
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from `lib/api-spec/openapi.yaml`)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — source of truth for all API contracts
- `lib/api-client-react/src/generated/` — generated React Query hooks + Zod schemas (do not edit)
- `lib/db/src/schema/` — Drizzle schema (users, subjects, question_papers, model_answers, uploads, evaluations)
- `artifacts/api-server/src/routes/` — Express route handlers per resource
- `artifacts/api-server/src/lib/ai.ts` — OCR (GPT Vision) + evaluation logic
- `artifacts/api-server/src/lib/auth.ts` — Clerk auth helpers, `requireAuth` middleware
- `artifacts/edubot/src/pages/` — React pages (sign-in, role-select, dashboard, subjects, questions, uploads, evaluations)
- `artifacts/edubot/src/components/layout.tsx` — shared sidebar layout

## Architecture decisions

- Contract-first API: OpenAPI spec → Orval codegen → typed React Query hooks used in all pages.
- OCR runs async after upload (background promise); teacher manually triggers AI evaluation after OCR completes.
- Role selected at first login (stored in Clerk `unsafeMetadata` + DB `role` column); teachers and students see different nav/pages.
- Multipart file upload via `multer`; files stored at `artifacts/api-server/uploads/` on disk.
- `req.params["id"] as string` cast required because Express 5 types params as `string | string[]`.

## Product

- **Teachers**: create subjects → create question papers with model answers per question → view all student submissions → trigger AI evaluation → override AI scores with notes
- **Students**: upload answer sheet images/PDFs → OCR extracts text → AI evaluates against model answers → view question-wise marks, feedback, strengths/weaknesses/suggestions
- **Analytics**: teacher dashboard shows class-wide stats, top performers, subject breakdown; student dashboard shows personal progress

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Run `pnpm run typecheck:libs` before frontend typecheck if lib types are stale.
- `req.params.id` from Express 5 must be cast: `req.params["id"] as string`.
- Generated hook names differ from API path names (e.g. `useListSubjects` not `useGetSubjects`, `useTriggerEvaluation` not `useEvaluateUpload`).
- File uploads use `multipart/form-data`; the `createUpload` mutation receives a raw `FormData` object.
- OCR model `gpt-5.1` supports vision; evaluation also uses `gpt-5.1` with `response_format: json_object`.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
