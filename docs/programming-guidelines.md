# Programming Guidelines

These guidelines summarize the linked course programming guidance and are intended to support good software development habits for this lab and later assignments.

## Compilation and Execution

- Read the entire assignment carefully and complete all requested tasks.
- Make sure all files needed to evaluate your solution are present in the repo.
- Record deviations, design decisions, open questions, and implementation notes in `ANSWERS.md`.
- Test your program thoroughly, including edge cases and failure cases.
- Compile with strong warning flags such as `-Wall`, and fix warnings whenever possible.
- Return `0` from `main` on success and distinct non-zero values for different failure modes.
- Use clear usage messages and error messages when command-line arguments can be malformed.
- Do not let exceptions escape `main` uncaught.
- Check the success or failure of operations such as opening files.

## Design and Coding Practices

- Prefer clear modular designs; split large functions into smaller helper functions when appropriate.
- Prefer standard containers, [RAII](https://en.cppreference.com/cpp/language/raii), and smart pointers over manual memory management when possible.
- If dynamic allocation is necessary, know where the allocated memory is released.
- Avoid `exit()` in the middle of program logic; prefer returning appropriate values or using well-justified exceptions.
- Avoid unnecessary default arguments.
- Use `nullptr` rather than `NULL` in modern C++ code.
- Avoid fixed-length buffers when variable-length strings or sequences are more appropriate.

## Debugging and Reliability

- Use a debugger methodically: characterize behavior, form hypotheses, make predictions, and test them.
- Ask what the last known-correct point was and what the first known-incorrect point was.
- Be careful with pointer arithmetic, array indexing, and access violations.
- Ensure error paths and normal paths both release resources safely.

## Style Notes

- Use descriptive names instead of hard-coded magic numbers.
- Keep comments focused on design intent, assumptions, and non-obvious behavior.
- Document any warning suppressions or unusual implementation choices in `ANSWERS.md`.
