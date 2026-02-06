# ABS Autonomy Protocol (Global Event Bus)
> **The Constitution of the Autonomous Agent**

This protocol defines the **MANDATORY** triggers that govern agent behavior.

## 1. Event: Start Task (`REHYDRATE`)
**Trigger**: When the user assigns a new objective or opens a workspace.
**Action Sequence**:
1.  **Read State**: Check `docs/_consolidated/STATE.md`.
2.  **Plan**: Create `task.md` and `implementation_plan.md`.
3.  **Consult**: Use `opencode` (if available) for architectural "Second Opinion".
    *   *Command*: `opencode consultant "Review this plan..."`

## 2. Event: Write Code (`EXECUTION`)
**Trigger**: Before any `write_to_file` or `replace_file_content`.
**Action Sequence**:
1.  **Policy Check**: Is this allowed by `abs scan`?
2.  **Security Check**: Does it violate `OWASP-TOP-10.md`?
3.  **Audit**: Log the intent to ABS WAL (`abs_wal_append`).

## 3. Event: Debug (`DEBUG-FIRST`)
**Trigger**: When an error occurs or a test fails.
**Action Sequence**:
1.  **Stop**: Do not randomly try fixes.
2.  **Systematic Workflow**: Execute `.agent/workflows/debuger.md`.
    *   *Rule*: "Hypothesis -> Evidence -> Fix".

## 4. Event: Finish Task (`HANDOFF`)
**Trigger**: Before calling `notify_user` to complete a task.
**Action Sequence**:
1.  **Verification**: Execute `.agent/workflows/verify.md`.
2.  **Documentation**: Update `WORKLOG.md` and `STATE.md`.
3.  **Handoff**: Generate a summary for the next session.

---
**Enforcement**:
This protocol is enforced by the Global Rules (`.cursorrules`). Deviations are considered **Governance Violations**.
