<h1 align="center">case.mq</h1>

String case conversion utilities implemented as an [mq](https://github.com/harehare/mq) module — camelCase, PascalCase, snake_case, kebab-case, CONSTANT_CASE, Title Case, and more.

## Features

- Converts between `camelCase`, `PascalCase`, `snake_case`, `kebab-case`, `CONSTANT_CASE`, `dot.case`, `path/case`, `Title Case`, and `Sentence case.`
- Correctly splits acronyms (`XMLHttpRequest` -> `xml`, `http`, `request`)
- Treats any run of non-alphanumeric characters as a word separator, so it accepts input in any of the above styles
- `is_*_case` predicates to check whether a string already matches a given convention

## Installation

Copy `case.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp case.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub without any local setup:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/case.mq" | case::snake_case(.)' input.txt
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/case.mq@v0.1.0" | ...'
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "case" | case::snake_case(.)' input.txt
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "case" | case::snake_case(.)' input.txt
```

## API

| Function | Description |
|---|---|
| `camel_case(s)` | Converts `s` to `camelCase` |
| `pascal_case(s)` | Converts `s` to `PascalCase` |
| `snake_case(s)` | Converts `s` to `snake_case` |
| `kebab_case(s)` | Converts `s` to `kebab-case` |
| `constant_case(s)` | Converts `s` to `CONSTANT_CASE` |
| `dot_case(s)` | Converts `s` to `dot.case` |
| `path_case(s)` | Converts `s` to `path/case` |
| `title_case(s)` | Converts `s` to `Title Case` |
| `sentence_case(s)` | Converts `s` to `Sentence case.` |
| `is_snake_case(s)` | Returns `true` if `s` is already valid `snake_case` |
| `is_kebab_case(s)` | Returns `true` if `s` is already valid `kebab-case` |
| `is_camel_case(s)` | Returns `true` if `s` is already valid `camelCase` |
| `is_pascal_case(s)` | Returns `true` if `s` is already valid `PascalCase` |

All conversion functions accept input in any style (or a mix), since they first
split the string into words on case boundaries, acronyms, and non-alphanumeric
separators before rejoining.

## Example

```sh
mq -L . -I raw 'import "case" | case::camel_case(.)' <<< "my_http-Server Name"
# => "myHttpServerName"

mq -L . -I raw 'import "case" | case::snake_case(.)' <<< "XMLHttpRequest"
# => "xml_http_request"
```

Rename a list of identifiers in bulk:

```mq
import "case"
| map(["userId", "user-name", "user_email"], case::snake_case)
# => ["user_id", "user_name", "user_email"]
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.6 or later (uses regex-based `gsub`/`split`).

## License

MIT
