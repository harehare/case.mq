<h1 align="center">case.mq</h1>

String case conversion utilities implemented as an [mq](https://github.com/harehare/mq) module — camelCase, PascalCase, snake_case, kebab-case, CONSTANT_CASE, Title Case, and more.

## Features

- Converts between `camelCase`, `PascalCase`, `snake_case`, `kebab-case`, `CONSTANT_CASE`, `dot.case`, `path/case`, `Title Case`, and `Sentence case.`
- Correctly splits acronyms (`XMLHttpRequest` -> `xml`, `http`, `request`)
- Treats any run of non-alphanumeric characters as a word separator, so it accepts input in any of the above styles
- `is_*_case` predicates to check whether a string already matches a given convention
- Optional abbreviation dictionary (`API`, `OAuth`, ...) so public names stay consistent, plus `explain_case` to show which rule was applied to each word

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
| `camel_case(s, abbrs=[])` | Converts `s` to `camelCase` |
| `pascal_case(s, abbrs=[])` | Converts `s` to `PascalCase` |
| `snake_case(s, abbrs=[])` | Converts `s` to `snake_case` |
| `kebab_case(s, abbrs=[])` | Converts `s` to `kebab-case` |
| `constant_case(s, abbrs=[])` | Converts `s` to `CONSTANT_CASE` |
| `dot_case(s, abbrs=[])` | Converts `s` to `dot.case` |
| `path_case(s, abbrs=[])` | Converts `s` to `path/case` |
| `title_case(s, abbrs=[])` | Converts `s` to `Title Case` |
| `sentence_case(s, abbrs=[])` | Converts `s` to `Sentence case.` |
| `abbr_dict(abbrs)` | Builds a reusable abbreviation dictionary from an array such as `["API", "OAuth"]` |
| `explain_case(s, style, abbrs=[])` | Returns the rule applied to each word when converting `s` to `style` |
| `is_snake_case(s)` | Returns `true` if `s` is already valid `snake_case` |
| `is_kebab_case(s)` | Returns `true` if `s` is already valid `kebab-case` |
| `is_camel_case(s)` | Returns `true` if `s` is already valid `camelCase` |
| `is_pascal_case(s)` | Returns `true` if `s` is already valid `PascalCase` |

All conversion functions accept input in any style (or a mix), since they first
split the string into words on case boundaries, acronyms, and non-alphanumeric
separators before rejoining.

## Abbreviation dictionary

By default every acronym is treated as an ordinary word, so `OAuthToken` becomes
`o_auth_token` and `api_key` becomes `ApiKey`. Pass a dictionary of abbreviations
as the optional second argument to keep such names consistent:

```mq
import "case"
| let abbrs = ["API", "OAuth", "ID"]
| [
    case::pascal_case("oauth_api_key", abbrs),  # "OAuthAPIKey"
    case::snake_case("OAuthAPIKey", abbrs),     # "oauth_api_key"
    case::camel_case("user_id", abbrs)          # "userID"
  ]
```

Each entry is written with the spelling you give it. Adjacent words that together
spell an entry are merged into one word (`o_auth`, `oAuth` and `OAuth` all match
`OAuth`), so the input style does not matter. When entries overlap, the longest
one wins (`OAuth2` over `OAuth`).

| Style | Rule applied to each word |
|---|---|
| `snake_case`, `kebab-case`, `dot.case`, `path/case` | lowercase (`oauth_token`) |
| `CONSTANT_CASE` | uppercase (`OAUTH_TOKEN`) |
| `PascalCase`, `Title Case` | dictionary spelling, otherwise capitalized (`OAuthToken`) |
| `camelCase` | first word lowercase (`apiKey`, `oauthToken`), the rest as in `PascalCase` |
| `Sentence case.` | first word as in `PascalCase`, the rest lowercase unless in the dictionary (`API key of OAuth`) |

Without a dictionary, output is unchanged. `abbr_dict(abbrs)` pre-builds the lookup
when you convert many strings with the same dictionary, and can be passed wherever
`abbrs` is accepted.

Use `explain_case` to see which rule was adopted for each word. `style` is one of
`"camel"`, `"pascal"`, `"snake"`, `"kebab"`, `"constant"`, `"dot"`, `"path"`,
`"title"` or `"sentence"`:

```mq
import "case"
| case::explain_case("oauth_api_key", "pascal", ["API", "OAuth"])
# => [{"word": "oauth", "rule": "abbreviation", "output": "OAuth"},
#     {"word": "api",   "rule": "abbreviation", "output": "API"},
#     {"word": "key",   "rule": "capitalize",   "output": "Key"}]
```

`rule` is `abbreviation`, `capitalize`, `lowercase` or `uppercase`.

Notes:

- Entries are matched as whole words, so digits and plurals need their own entry:
  register `OAuth2` and `APIs` if you want `OAuth2Token` and `APIs` kept intact.
- No abbreviations are built in. Conventions differ (`UserID` vs `UserId`), so
  choose the dictionary that matches your public API.

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
