# GPT Markdown Test Framework

This directory contains the widget test framework for the `gpt_markdown` package. The framework uses a custom serializer to produce stable, comparable string representations of the rendered markdown output.

## Overview

### Design Philosophy

The test framework is designed around these principles:

1. **Stable Output**: Tests compare serialized string representations rather than widget instances, avoiding issues with theme-dependent styles, memory addresses, and Flutter version changes.

2. **Semantic Testing**: The serializer captures the semantic meaning (bold, italic, list items, etc.) rather than visual details (colors, font sizes).

3. **Granular Organization**: Each markdown feature has its own test file for easy navigation and focused testing.

4. **Regression Testing**: A dedicated folder for bug reproduction tests ensures fixed bugs stay fixed.

## Directory Structure

```
test/
├── README.md                    # This file
├── utils/
│   ├── serializer.dart          # Custom stable serializer
│   └── test_helpers.dart        # Shared test utilities
│
├── inline/                      # Inline element tests
│   ├── bold_test.dart
│   ├── italic_test.dart
│   ├── strikethrough_test.dart
│   ├── underline_test.dart
│   ├── highlight_test.dart
│   └── links_test.dart
│
├── block/                       # Block element tests
│   ├── headings_test.dart
│   ├── code_block_test.dart
│   ├── unordered_list_test.dart
│   ├── ordered_list_test.dart
│   ├── checkbox_test.dart
│   ├── radio_button_test.dart
│   ├── table_test.dart
│   ├── blockquote_test.dart
│   ├── horizontal_rule_test.dart
│   └── indent_test.dart
│
├── latex/                       # LaTeX tests
│   ├── inline_latex_test.dart
│   └── block_latex_test.dart
│
├── images/                      # Image tests
│   └── image_test.dart
│
└── regression/                  # Bug reproduction tests
    └── issue_<number>_<description>_test.dart
```

## Serializer Output Format Reference

The serializer transforms the widget tree into a stable string format. Here's the complete reference:

### Text Elements

| Markdown | Serialized Output |
|----------|-------------------|
| `plain text` | `TEXT("plain text")` |
| `**bold**` | `TEXT("bold")[bold]` |
| `*italic*` | `TEXT("italic")[italic]` |
| `***bold italic***` | `TEXT("bold italic")[bold,italic]` |
| `~~striked~~` | `TEXT("striked")[strike]` |
| `<u>underline</u>` | `TEXT("underline")[underline]` |
| `` `code` `` | `TEXT("code")[highlight]` |

### Links and Images

| Markdown | Serialized Output |
|----------|-------------------|
| `[text](url)` | `LINK("text", url="url")` |
| `![alt](img.png)` | `IMAGE(url="img.png")` |
| `![100x50](img.png)` | `IMAGE(url="img.png", w=100, h=50)` |

### Headings

| Markdown | Serialized Output |
|----------|-------------------|
| `# H1` | `H1("H1")` |
| `## H2` | `H2("H2")` |
| `### H3` | `H3("H3")` |
| `#### H4` | `H4("H4")` |
| `##### H5` | `H5("H5")` |
| `###### H6` | `H6("H6")` |

### Lists

| Markdown | Serialized Output |
|----------|-------------------|
| `- item` | `UL_ITEM(TEXT("item"))` |
| `1. item` | `OL_ITEM(1, TEXT("item"))` |

### Form Elements

| Markdown | Serialized Output |
|----------|-------------------|
| `[ ] unchecked` | `CHECKBOX(checked=false, TEXT("unchecked"))` |
| `[x] checked` | `CHECKBOX(checked=true, TEXT("checked"))` |
| `( ) unchecked` | `RADIO(checked=false, TEXT("unchecked"))` |
| `(x) checked` | `RADIO(checked=true, TEXT("checked"))` |

### Code Blocks

````markdown
```dart
void main() {}
```
````

Serialized: `CODE_BLOCK(lang="dart", "void main() {}")`

### LaTeX

| Markdown | Serialized Output |
|----------|-------------------|
| `\(x^2\)` | `LATEX_INLINE("x^2")` |
| `\[x^2 + y^2\]` | `LATEX_BLOCK("x^2 + y^2")` |

### Other Elements

| Markdown | Serialized Output |
|----------|-------------------|
| `---` | `HR` |
| `> quote` | `BLOCKQUOTE(TEXT("quote"))` |
| (paragraph break) | `NEWLINE` |

### Tables

```markdown
| A | B |
|---|---|
| 1 | 2 |
```

Serialized:
```
TABLE(
  HEADER("A", "B")
  ROW("1", "2")
)
```

## How to Write Tests

### Basic Test Pattern

```dart
import 'package:flutter_test/flutter_test.dart';
import '../utils/test_helpers.dart';

void main() {
  testWidgets('descriptive test name', (tester) async {
    await expectMarkdown(
      tester,
      '**bold text**',           // Markdown input
      'TEXT("bold text")[bold]', // Expected serialized output
    );
  });
}
```

### Available Helpers

#### `expectMarkdown`
The primary helper for exact output matching.

```dart
await expectMarkdown(tester, '**bold**', 'TEXT("bold")[bold]');
```

#### `expectMarkdownContains`
For partial matching when exact output is complex.

```dart
await expectMarkdownContains(tester, 'complex **markdown**', 'TEXT("markdown")[bold]');
```

#### `expectMarkdownMatches`
For regex-based matching when content varies.

```dart
await expectMarkdownMatches(tester, 'text', RegExp(r'TEXT\(".*"\)'));
```

#### `debugMarkdownOutput`
For discovering the expected output when writing new tests.

```dart
await debugMarkdownOutput(tester, '**bold** and *italic*');
// Prints: TEXT("bold")[bold] TEXT(" and ") TEXT("italic")[italic]
```

### Testing with Custom Styles

```dart
await expectMarkdown(
  tester,
  '**bold**',
  'TEXT("bold")[bold]',
  style: TextStyle(fontSize: 16),
);
```

## Adding Regression Tests

Regression tests reproduce specific bugs from GitHub issues. Follow this naming convention:

**Filename**: `issue_<number>_<brief_description>_test.dart`

**Example**: `issue_42_nested_bold_italic_test.dart`

### Template

```dart
// Regression test for: https://github.com/Infinitix-LLC/gpt_markdown/issues/42
//
// Bug: Nested bold and italic text was not rendering correctly
// when bold was the outer wrapper.
//
// Input: ***bold italic***
// Expected: Bold and italic styling applied
// Actual (before fix): Only bold was applied

import 'package:flutter_test/flutter_test.dart';
import '../utils/test_helpers.dart';

void main() {
  testWidgets('issue #42: nested bold italic renders correctly', (tester) async {
    await expectMarkdown(
      tester,
      '***bold italic***',
      'TEXT("bold italic")[bold,italic]',
    );
  });
}
```

## Running Tests

### Run All Tests

```bash
flutter test
```

### Run Tests in a Specific Directory

```bash
flutter test test/inline/
flutter test test/block/
```

### Run a Specific Test File

```bash
flutter test test/inline/bold_test.dart
```

### Run with Verbose Output

```bash
flutter test --reporter expanded
```

### Run with Coverage

```bash
flutter test --coverage
```

## Tips

1. **Discovering Output Format**: Use `debugMarkdownOutput` to see what the serializer produces for any input.

2. **Nested Content**: The serializer handles nesting automatically. `UL_ITEM(TEXT("bold")[bold])` represents a list item containing bold text.

3. **Whitespace**: Leading/trailing whitespace in text is preserved. Use exact matching.

4. **Multiple Elements**: Multiple elements are space-separated in the output.

5. **Complex Markdown**: For complex inputs, use `expectMarkdownContains` to test specific parts rather than the entire output.
