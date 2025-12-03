# GitHub Copilot Instructions for CodeQL Action Report

## CodeQL Language Support

**IMPORTANT**: This action is designed to work with **ALL CodeQL-supported languages**, not just a specific language.

### Supported Languages
CodeQL supports the following languages ex:
- **Actions** (GitHub Actions workflows)
- **Cpp**
- **Csharp**
- **Go**
- **Java**
- **JavaScript**
- **Python**
- **Ruby**
- **Rust**
- **Swift**

### Guidelines for Development

1. **Never hardcode language-specific values**: Avoid hardcoding language names like `/actions`, `/javascript`, `/python`, etc. in code that processes SARIF files or CodeQL results.

2. **Use pattern matching**: When searching for CodeQL metadata in SARIF files, use pattern matching that works across all languages:
   - Use `contains()` or `startswith()` for descriptor IDs
   - Example: `contains("/diagnostics/successfully-extracted-files")` instead of `== "actions/diagnostics/successfully-extracted-files"`

3. **Language-agnostic queries**: When writing jq queries or processing SARIF data, ensure the logic works for any language:
   - Extracted files: Look for IDs containing `/diagnostics/successfully-extracted-files`
   - Expected files: Look for IDs starting with `cli/expected-extracted-files/`
   - Build mode: Always use `cli/build-mode` (language-independent)

4. **Test with multiple languages**: When making changes, consider how they will work with:
   - Interpreted languages (Python, JavaScript, Ruby)
   - Compiled languages (Java, C++, C#, Go)
   - Configuration languages (Actions/YAML)

5. **Documentation**: Always mention "all CodeQL languages" or "language-agnostic" when documenting features.

### Common Patterns

#### ✅ Correct (Language-Agnostic)
```bash
# Count files for any language
EXTRACTED_FILES=$(jq -r '
  [.runs[0].invocations[0].toolExecutionNotifications[] |
  select(.descriptor.id | contains("/diagnostics/successfully-extracted-files"))] | length
' "$SARIF_FILE")
```

#### ❌ Incorrect (Language-Specific)
```bash
# Only works for Actions
EXTRACTED_FILES=$(jq -r '
  [.runs[0].invocations[0].toolExecutionNotifications[] |
  select(.descriptor.id == "actions/diagnostics/successfully-extracted-files")] | length
' "$SARIF_FILE")
```

## Summary Generation

When generating summaries or reports, ensure they work for all languages without modification. The summary should accurately reflect the scanned language without requiring language-specific code paths.
