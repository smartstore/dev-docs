# 🥚 Search filter expressions

A search filter expression is a set of criteria used to define a subset of data to be retrieved from the database. In an expression, you can specify various conditions that the data must meet in order to be included in the search results. You can also use Boolean operators like `and` and `or` to combine multiple conditions and create more complex expressions.

In Smartstore, you can specify such expressions on data grid field level. Any textbox in a data grid filter form which contains a question mark icon can process expressions.

## Examples

| Example                                 | Result                                                                           |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| banana joe                              | Contains "banana" or contains "joe"                                              |
| banana and !\*.joe                      | Contains "banana" but does not match "\*.joe"                                    |
| banana and (!\~"hello world" or !\*jim) | Contains "banana", but does not contain "hello world" or does not end with "jim" |
| \*Leather and !(Sofa Jacket\*)          | Ends with "Leather", but does not starts with "Sofa" or "Jacket"                 |
| (>=10 and <=100) or 1 or >1000          | Is between 10 and 100, or equals 1, or is greater than 1000                      |

## Terms

Quoted search term (double or single), or unquoted search term without whitespaces. For example, unquoted _banana joe_ is the equivalent of _\~banana or \~joe_, whereas quoted _"banana joe"_ performs an exact term match. Supports wildcards (\* or ?). If wildcards are present, default operator is switched to "Equals" (=). Use "NotEquals" (!) to negate pattern.

## Groups

Multiple search words or phrases may be grouped in a fielded query by enclosing them in parenthesis. Any group can be negated with a preceding ! (exclamation mark), e.g. _!(banana and joe)_. Groups can be nested as often as required.

## Operators

| = _or_ == | Equals (default when omitted on numeric terms)                           |
| --------- | ------------------------------------------------------------------------ |
| ! _or_ != | Not equals                                                               |
| >         | Greater than                                                             |
| >=        | Greater than or equal                                                    |
| <         | Less than                                                                |
| <=        | Less than or equal                                                       |
| \~        | Contains (default when omitted on string terms)                          |
| !\~       | Does not contain                                                         |
| and, or   | Logical term combinators (case-insensitive). If omitted, "or" is default |
