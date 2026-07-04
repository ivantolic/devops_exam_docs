# Final expanded LO4 and LO5 documents

Use these instead of the older `LO4_QA_FINAL_SAFE.md` and `LO5_QA_FINAL_SAFE.md`.

What changed:

- Questions are now written fully, closer to the professor's original practice questions.
- Every answer includes a simple English typed answer.
- Commands are not just dumped anymore: each section explains what is being diagnosed/fixed, what to run, how to verify success, and why the fix works.
- LO5 troubleshooting tasks now include confirmation commands after the fix, especially for cases like OOMKilled, readiness probe failures, Service endpoint problems, ImagePullBackOff and CrashLoopBackOff.
- The commented `yaml/` folder is included.

Recommended GitHub structure:

```text
devops-exam/
  LO4_QA_FINAL_EXPANDED.md
  LO5_QA_FINAL_EXPANDED.md
  LO6_FULL_MARKS_SIMPLE_ENGLISH_REVIEWED.md
  LO6_PRAVILA_ZA_PISANJE_ODGOVORA_HR.md
  EXAM_TRAPS_AND_ALTERNATIVE_QUESTIONS.md
  SAFE_COPY_PASTE_RULES.md
  FINAL_CORRECTIONS_LOG.md
  LO6_REVIEW_NOTES.md
  yaml/
```

Additional final pass:

- Added command-by-command explanations under the command blocks, so you can see what each command is checking or fixing.
- LO5 examples now include verification steps after fixes, not only the diagnostic command.
