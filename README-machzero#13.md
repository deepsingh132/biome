## What is the Repo 

It is a open source toolchain for web development that is used for all in one code formatting (Automatically makes your code clean and consistent - big lines etc) and linter (Checks whether your code has syntax issues.)

Flow of Biome 

Parser (Reads and understands the Markdown structure.)→ 
Syntax Tree(Organises the document into structured elements, such as lists, paragraphs and blockquotes.) → 
Formatter(Produces properly formatted Markdown.)

## Why Choose 
Our Team has already work on prettier in past and as prettier is slow and Biome is faster by 50% as compared to prettier.

Reference Article:
https://medium.com/simform-engineering/biome-js-vs-eslint-and-prettier-what-it-replaces-what-it-doesnt-and-when-to-switch-7daf26d045cc

Responsive maintainers. PR was approved about 8 hours after opening.

One function to change, but the diagnosis crosses three layers 
Parser → Syntax Tree → Formatter.

## Issue 
A correct formatter must satisfy two important conditions: it must preserve the original meaning, and it must produce the same output every time it runs.
We found a bug in Biome's Markdown formatter. When a list item ended with a blockquote, Biome removed the blank line after it. In Markdown, that blank line separates the blockquote from the next normal paragraph. Without it, the next paragraph becomes part of the quote. A formatter should change how a document looks, not what the document means, this bug may lead to different contexual issues. 

## What was wrong with the old code?
The function list_ends_with_line_break is in:

markdown/lists/block_list.rs:782

This function checks the last block of the last list item.

Before our fix, the function did not correctly check blockquotes.

A blockquote is neither a leaf element nor a list item.

Because of this, the function returned false.

Biome then removed the blank line after the blockquote.

This caused the next paragraph to become part of the blockquote.
When we ran the formatter again, it added the > symbol before the paragraph. This meant the formatter changed the document's syntax tree and did not produce consistent output.
Our contribution was not simply fixing an empty line. We helped maintain the correctness of a software formatting system.

## Relevant Codebase

markdown/lists/block_List.ts - contains list layout logic.
function - list_ends_with_line_break - checks whether the last element of a list is already has a line ending.
markdown/auxilliary/newline.rs - decides whether to keep or remove a newline after a list.

## Fix

Before the fix:

list_ends_with_line_break
- Checked the last block of the last list item.
- Did not correctly handle blockquotes.
- Returned false for the affected blockquote structure.
After the fix:

list_ends_with_line_break
- Uses block_ends_with_line_break.
- Correctly checks blockquotes.
- Supports nested structures.

## Testing 

CI/CD tests of the repo itself. 

e2e tests of the browser. 

## Cross question 
"Why not a bigger feature?" A correctness fix in a formatter is high value per line: formatters run on every save and in CI for thousands of repos, and one that rewrites meaning is worse than none.
What is > it is a simbol that is used to markdown like the fololwing test is taken from the soming else said or wrote but it doesent always > is not just a symbol for decoration. It changes how Markdown interprets (understands or treats something in a particular way.) and displays the text.

## cascading effect
 The defect is a sequence of effects. One function gives an incorrect value. Four other parts of the formatter use this value. Each part is correct by itself. Thus the error is not visible until the last step.

## Correction
The correction. The PR adds a condition for a blockquote. The function now returns true for a blockquote that ends with a line break. The newline file keeps the Newline node. The output has the blank line. The paragraph stays outside the blockquote. The second output is the same as the first output.

## Environment

Fedora flavour of Linux (KDE Plasma)
