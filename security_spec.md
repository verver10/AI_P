# Security Specification: 프롬프트 창고 (Prompt Warehouse - Admin Auth)

## 1. Data Invariants
1. **Collection**: `/prompts/{promptId}`
2. **Schema**: Every document in `prompts` MUST have exactly the 4 fields `title`, `category`, `content`, `createdAt` (no extra/shadow fields).
3. **Field Bounds**:
   - `promptId`: `^[a-zA-Z0-9_-]+$`, max 128 chars.
   - `title`: non-empty string, 1–120 characters.
   - `category`: one of `["사진 합성", "스타일 변환", "캐릭터·포스터"]` (1–40 characters).
   - `content`: non-empty string, 1–20,000 characters (supports prompts well over 1,000 characters).
   - `createdAt`: server timestamp (`request.time`) on create, immutable on update.
4. **Access Control**:
   - Guests (unauthenticated) may only read (`get`, `list`) prompts.
   - Only the single verified Google admin (`request.auth != null && request.auth.token.email_verified == true && request.auth.token.email == 's01053522183@gmail.com'`) may `create`, `update`, or `delete` prompts.
   - Updates may only modify `title`, `category`, and `content`; `createdAt` is immortal.

## 2. Dirty Dozen Payloads (Must All Be Denied)
1. Unauthenticated `create` attempt
2. Unauthenticated `update` or `delete` attempt
3. Authenticated non-admin email `create`/`update`/`delete` attempt
4. Spoofed admin email with `email_verified: false`
5. Shadow field injection on create (`isAdmin: true`)
6. Missing required field (`content` omitted)
7. Empty or oversized `title` (0 or 121+ chars)
8. Invalid `category` (`"기타"`)
9. Empty or oversized `content` (0 or 20,001+ chars)
10. Spoofed client timestamp on create (`createdAt != request.time`)
11. Mutating immortal `createdAt` field during `update`
12. Invalid document ID (`../bad/id` or 129+ chars)
