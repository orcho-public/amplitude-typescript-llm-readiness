# Documentation–Implementation Mismatches and LLM API Hallucination Risk

**Repository analyzed:** `Amplitude-TypeScript`  
**Commit:** `6ff4832a88f61126d9c0edb25bef5a2378e88356` (short: `6ff4832`)  
**Commit date:** 2026-01-26  
**Commit message:** `chore(release): publish`  
**Package version (analytics-browser):** `2.34.0`

---

## Executive summary

- **13 public-facing APIs** were reviewed against the specified commit.
- **7 findings are fully verified, high-confidence contradictions** where comments or JSDoc clearly misstate runtime behavior.
- **6 additional findings identify ambiguous API contracts**, where documentation is incomplete or underspecified but not strictly false.
- The dominant issue is **API hallucination risk**:
  - callers infer return values, sync/async behavior, or nullability that the code does not guarantee.
- These issues do not indicate broken functionality, but they **increase the likelihood of plausible-but-wrong code**, especially in LLM-assisted workflows.

---

## What “API hallucination” means in this context

API hallucination occurs when:
- documentation asserts a contract the runtime does not implement, or
- documentation omits critical constraints (e.g. `undefined`, async behavior),
- forcing callers to *invent* behavior that “seems reasonable.”

In AI-assisted coding:
- comments are weighted more heavily than implementation,
- contradictions force models to guess intent,
- guesses optimize for plausibility, not correctness.

---

## Fully verified contradictions (high hallucination risk)

These findings are clear doc–code mismatches. The documented behavior is **false** relative to runtime behavior in the analyzed commit.

### 1. `translateRemoteConfigToLocal`

- **Location:** `packages/analytics-browser/src/config/joined-config.ts`
- **Documented contract:** Returns a transformed config object.
- **Runtime behavior:** Returns `void`; performs side effects only.
- **Hallucination risk:** Callers attempt to capture or chain a nonexistent return value.

---

### 2. `AmplitudeBrowser.init`

- **Location:** `packages/analytics-browser/src/browser-client.ts`
- **Documented contract:** Returns `void`.
- **Runtime behavior:** Returns an `AmplitudeReturn<T>` (thenable with `.promise`).
- **Hallucination risk:** Missing `await`, race conditions during initialization.

---

### 3. `RemoteConfigClient.updateConfigs`

- **Location:** `packages/analytics-core/src/remote-config/remote-config.ts`
- **Documented contract:** Returns `void`.
- **Runtime behavior:** `async` function returning `Promise<void>`.
- **Hallucination risk:** Fire-and-forget usage where sequencing matters.

---

### 4. `parseHeaderCaptureRule`

- **Location:** `packages/plugin-network-capture-browser/src/track-network-event.ts`
- **Documented contract:** Returns a `HeaderCaptureRule` object.
- **Runtime behavior:** Returns `string[] | undefined`.
- **Hallucination risk:** Property access on non-object values.

---

### 5. `updateBrowserConfigWithRemoteConfig`

- **Location:** `packages/analytics-browser/src/config/joined-config.ts`
- **Documented contract:** Implies a returned updated config.
- **Runtime behavior:** Returns `void`; mutates input object.
- **Hallucination risk:** Incorrect chaining and silent misconfiguration.

---

### 6. `parseCdnUrl`

- **Location:** `test-server/mock-api.js`
- **Documented contract:** Always returns a parsed object.
- **Runtime behavior:** Returns `null` when input does not match.
- **Hallucination risk:** Missing null checks leading to crashes in tests or mocks.

---

### 7. `resolveSourcePath`

- **Location:** `test-server/mock-api.js`
- **Documented contract:** Returns a `string`.
- **Runtime behavior:** Returns a structured object.
- **Hallucination risk:** String operations applied to objects.

---

## Ambiguous API contracts (moderate hallucination risk)

These findings are not outright false documentation, but **leave critical aspects of the API contract implicit**.

### 8. `asyncMap`

- **Reality:** Returns `ZenObservable<R>`.
- **Ambiguity:** Observable/streaming semantics are not emphasized.
- **Hallucination risk:** Promise-like usage inferred.

---

### 9. `uploadFile`

- **Reality:** `async` function returning `Promise<void>`.
- **Ambiguity:** No documented return semantics.
- **Hallucination risk:** Invented boolean success/failure logic.

---

### 10. `getEventsArraySize`

- **Reality:** Returns size plus serialization overhead.
- **Ambiguity:** Exact return semantics not fully specified.
- **Hallucination risk:** Incorrect size thresholds and buffering logic.

---

### 11. `RequestWrapperFetch.headers`

- **Reality:** Returns `Record<string, string> | undefined`.
- **Ambiguity:** `undefined` not clearly documented.
- **Hallucination risk:** Assumption that headers are always present.

---

### 12. `nativeConfig`

- **Reality:** Synchronous function returning a config object.
- **Ambiguity:** No JSDoc defining the contract.
- **Hallucination risk:** Async usage inferred from surrounding patterns.

---

### 13. `ResponseWrapperFetch.headers`

- **Reality:** Returns `Record<string, string> | undefined`.
- **Ambiguity:** Documentation vague about shape and nullability.
- **Hallucination risk:** Overconfident destructuring and silent failures.

---

## Why this matters

- **Hallucinated API contracts are costly**:
  - they compile,
  - they look correct,
  - they fail late and inconsistently.
- **LLMs amplify documentation drift**:
  - comments outweigh implementation,
  - ambiguity forces guesswork,
  - guesswork increases retries and correction loops.
- **Reducing ambiguity lowers cost and risk**:
  - fewer runtime errors,
  - fewer corrective iterations,
  - lower token usage in AI-assisted workflows.

Improving documentation accuracy directly reduces **API hallucination surface area**.

---

## Research attribution

This analysis aligns with prior research on hallucination and ambiguity in LLM-assisted code generation:

- Ji et al., 2023 — *A Survey of Hallucination in Large Language Models*  
  https://arxiv.org/abs/2311.05232

- Liu et al., 2024 — *CodeMirage: Hallucinations in Code Generation*  
  https://arxiv.org/abs/2408.08333
