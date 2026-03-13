# Open Source Contributions Summary

> **Account:** [@saiprasanth-git](https://github.com/saiprasanth-git)  
> **Period:** May 2025  
> **Total PRs Submitted:** 3
> **Profile Optimized:** Yes

---

## 1. Google ADK Python — PR #4800

**Repository:** [google/adk-python](https://github.com/google/adk-python)  
**PR:** [fix: extract URL fragment key=value pairs as query params in _prepare_request_params](https://github.com/google/adk-python/pull/4800)  
**Status:** 🟢 Open — Awaiting maintainer review approval  
**Assignee:** @rohityan (Google maintainer)

### Problem Fixed
The `_prepare_request_params` function in `rest_api_tool.py` used `urlparse` to split URLs but **completely ignored the fragment portion** of URLs. This caused silent data loss when `ApplicationIntegrationToolset` generated endpoint URLs like:
```
https://integrations.googleapis.com/v2/.../execute?triggerId=api_trigger/Foo#httpMethod=POST
```
The `#httpMethod=POST` fragment was dropped, causing downstream HTTP 400 errors.

### Fix Applied
- Refactored parameter extraction loop to process **both** `parsed_url.query` and `parsed_url.fragment`
- Used `setdefault` so explicitly passed query params always take priority
- URL is rebuilt with both query and fragment cleared after extraction

### Files Changed
- `contributing/samples/tools_and_toolsets/rest_api_tool/rest_api_tool.py` — core fix
- Added test `test_prepare_request_params_fragment_params_become_query_params`

### CI Status
- ✅ CLA signed and passing
- ✅ Header check passing  
- ✅ Mypy syntax error fixed (removed redundant outer `if` guard)
- ⏳ 5 CI workflows awaiting maintainer approval to run

### Fixes Issue
[ApplicationIntegrationToolset ExecuteConnection requests return HTTP 400 (triggerId fragment not sent) #4598](https://github.com/google/adk-python/issues/4598)

---

## 2. LangChain — PR #35794

**Repository:** [langchain-ai/langchain](https://github.com/langchain-ai/langchain)  
**PR:** [fix(langchain-openai): use per-event-loop cache for async httpx client to prevent cross-loop errors](https://github.com/langchain-ai/langchain/pull/35794)  
**Status:** 🔴 Closed (auto-closed — assignment required before PR)  
**Fork PR:** [saiprasanth-git/langchain#1](https://github.com/saiprasanth-git/langchain/pull/1) (Open)

### Problem Fixed
The `_cached_async_httpx_client` function in `_client_utils.py` was decorated with `@lru_cache`, making the `httpx.AsyncClient` **process-global and shared across all event loops**. This caused `RuntimeError: Event loop is closed` errors when `ainvoke()` was called from:
- Multiple threads each running `asyncio.run()` (each creates their own event loop)
- Sequential `asyncio.run()` calls
- Frameworks like Celery that spin up multiple event loops

### Fix Applied
- Replaced process-global `@lru_cache` with a `WeakValueDictionary` keyed by `(base_url, timeout, loop_id)`
- Each event loop gets its own `AsyncClient` instance — no more cross-loop sharing
- `WeakValueDictionary` automatically cleans up entries when a client is garbage collected (no memory leaks)
- Sync clients (`_cached_sync_httpx_client`) were left unchanged as they don't have this problem

### Files Changed
- `libs/partners/openai/langchain_openai/chat_models/_client_utils.py` — core fix (+44/-7 lines)

### CI Results
- 76/86 checks passing
- CodSpeed flagged init-time performance regression (WeakValueDictionary lookup overhead vs lru_cache)
- Auto-closed because external contributors must be assigned to issues first

### Current Action Needed
- Awaiting assignment on [issue #35783](https://github.com/langchain-ai/langchain/issues/35783) from a maintainer
- Assignment request comment posted on the issue
- Once assigned, PR description edit will trigger automatic reopen

### Fixes Issue
[Bug: @lru_cache-ed async httpx client causes APIConnectionError across event loops #35783](https://github.com/langchain-ai/langchain/issues/35783)

---

## Profile Optimizations

| Task | Status |
|------|--------|
| Renamed profile README repo to match username exactly | ✅ Done |
| README now visible on GitHub profile overview page | ✅ Done |
| Pinned 6 repositories on profile (including forks with active PRs) | ✅ Done |

---

## Summary Table

| # | Repository | PR | Type | Status |
|---|-----------|-----|------|--------|
| 1 | google/adk-python | [#4800](https://github.com/google/adk-python/pull/4800) | Bug fix | 🟢 Open, awaiting review |
| 2 | langchain-ai/langchain | [#35794](https://github.com/langchain-ai/langchain/pull/35794) | Bug fix | 🔴 Closed (need assignment) |

| 3 | google/adk-python | [#4826](https://github.com/google/adk-python/pull/4826) | Feature | 🟢 Open, awaiting review |

## 3. Google ADK Python — PR #4826

**Repository:** [google/adk-python](https://github.com/google/adk-python)
**PR:** [feat(tools): add tuple type support in function parameter schema parsing](https://github.com/google/adk-python/pull/4826)

**Status:** 🟢 Open — Awaiting maintainer review
**Fixes Issue:** [Support Tuple for function tool signature #3575](https://github.com/google/adk-python/issues/3575)

### Problem Fixed
Python's `tuple` type (e.g. `tuple[int, str]`, `tuple[float, ...]`) was completely unsupported in automatic function calling schema generation. When a developer tried to define a tool function with a `tuple` parameter, the ADK raised a `ValueError` and refused to register the function as a tool.

### Fix Applied
- Added `tuple` to `_py_builtin_type_to_schema_type` mapping as `types.Type.ARRAY`
- Added `if origin is tuple:` handler in `_parse_schema_from_parameter` that:
  - For `tuple[T, ...]` (variable-length homogeneous): sets `schema.items` to the element type schema
  - For fixed-length tuples like `tuple[int, str]`: returns generic `ARRAY` (since `types.Schema` doesn't support `prefixItems`)
- Proper default value validation and `_raise_if_schema_unsupported` checks included

### Files Changed
- `src/google/adk/tools/_function_parameter_parse_util.py`

---

## Next Steps

1. **ADK PR #4800** — Respond to any further reviewer feedback from @rohityan; approve CI workflow runs
2. **LangChain** — Watch issue #35783 for maintainer assignment; once assigned, edit PR description to reopen
3. Consider addressing the CodSpeed performance regression in the LangChain fix before reopening
4. **ADK PR #4826** — Await review; address any feedback on tuple type implementation
