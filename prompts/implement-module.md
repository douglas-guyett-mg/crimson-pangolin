# Prompt: Implement Module (Stage 3)

Objective
- Execute the `{module-name}` implementation per its checklist and verify completion via tests.

Inputs
- `implementation/{module-name}/overview.md`
- `implementation/{module-name}/implementation-checklist.md`
- `implementation/{module-name}/test-conditions.md`

Guardrails
- Follow the checklist strictly; update it as you work
- Use package managers for any dependency changes (never edit manifests directly)
- Safe-by-default runs only: tests/linters/builds are okay; deployments or installs require explicit permission
- Do not commit/push/merge without explicit approval

Steps
1) Read `overview.md`; confirm dependencies are satisfied
2) Work through `implementation-checklist.md` from top to bottom
   - For each file to create/modify, implement minimal code to satisfy tests
   - After each meaningful change, run targeted tests as specified in `test-conditions.md`
   - Check off items as you complete them
3) Keep `implementation-checklist.md` in sync (add sub-items if needed)
4) Run full module tests when the checklist is done
5) Record a Test Summary in the checklist file
6) Stop and hand off to Stage 4 review

Testing Guidance
- Prefer smallest scope runs: single test → test file → test package
- Consider success only if exit code is 0 and logs show no errors
- If a test fails, apply the minimal safe fix and re-run targeted tests

Definition of Done
- All checklist boxes are checked
- All tests in `test-conditions.md` pass cleanly
- Acceptance criteria from `overview.md` are met

Expected Outputs
- Updated source files per the checklist
- Updated `implementation-checklist.md` with all items checked and a test summary
- No unsolicited files created

