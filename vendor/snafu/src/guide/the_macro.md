# Details about the output of the `Snafu` macro

This procedural macro:

- produces the corresponding context selectors
- implements the [`Error`][Error] trait
- implements the [`Display`][Display] trait
- implements the [`ErrorCompat`][ErrorCompat] trait

## Detailed example

```rust
use snafu::{prelude::*, Backtrace};
use std::path::PathBuf;

#[derive(Debug, Snafu)]
enum Error {
    #[snafu(display("Could not open config at {}", filename.display()))]
    OpenConfig {
        filename: PathBuf,
        source: std::io::Error,
    },

    #[snafu(display("Could not open config"))]
    SaveConfig { source: std::io::Error },

    #[snafu(display("The user id {user_id} is invalid"))]
    UserIdInvalid { user_id: i32, backtrace: Backtrace },

    #[snafu(display("Could not validate config with key {key}: checksum was {checksum}"))]
    ConfigValidationFailed {
        checksum: u64,
        key: String,
        source: crypto::Error,
    },
}
# mod crypto { #[derive(Debug, snafu::Snafu)] pub struct Error; }
```

### Generated code

<div style="color: #000; background: #ffffd0; padding: 0.6em; margin-bottom: 0.6em;">

**Note** — The actual generated code may differ in exact names and
details. This section is only intended to provide general
guidance. Use a tool like [cargo-expand][] to see the exact generated
code for your situation.

</div>

[cargo-expand]: https://crates.io/crates/cargo-expand

#### Context selectors

Each enum variant will generate an additional type called a *context
selector*:

```rust,ignore
struct OpenConfigSnafu<P> {
    filename: P,
}

struct SaveConfigSnafu<P>;

struct UserIdInvalidSnafu<I> {
    user_id: I,
}

struct ConfigValidationFailedSnafu {
    checksum: u64,
    key: String,
    source: crypto::Error,
}
```

Notably:

1. The name of the selector is the enum variant's name with the suffix
   `Snafu` added. If the name originally ended in `Error`, that is
   removed.
1. The `source` and `backtrace` fields have been removed; the
   library will automatically handle these for you.
1. Each remaining field's type has been replaced with a generic
   type.
1. If there are no fields remaining for the user to specify, the
   selector will not require curly braces.

If the original variant had a `source` field, its context selector
will have an implementation of [`IntoError`][IntoError]:

```rust,ignore
impl<P> IntoError<Error> for OpenConfigSnafu<P>
where
    P: Into<PathBuf>,
```

Otherwise, the context selector will have the inherent methods `build`
and `fail` and can be used with the [`ensure`](ensure) macro:

```rust,ignore
impl<I> UserIdInvalidSnafu<I>
where
    I: Into<i32>,
{
    fn build(self) -> Error { /* ... */ }

    fn fail<T>(self) -> Result<T, Error> { /* ... */ }
}
```

If the original variant had a `backtrace` field, the backtrace
will be automatically constructed when either `IntoError` or
`build`/`fail` are called.

#### `Error`

[`Error::source`][source] will return the underlying error, if
there is one:

```rust,ignore
impl std::error::Error for Error {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            Error::OpenConfig { source, .. } => Some(source),
            Error::SaveConfig { source, .. } => Some(source),
            Error::UserIdInvalid { .. } => None,
            Error::ConfigValidationFailed { source, .. } => Some(source),
        }
    }
}
```

[`Error::cause`][cause] will return the same as `source`. As
[`Error::description`][description] is soft-deprecated, it will
return a string matching the name of the variant.

#### `Display`

Every field of the enum variant is made available to the format
string, even if they are not used:

```rust,ignore
impl std::fmt::Display for Error {
    fn fmt(&self, f: &mut fmt::Formatter ) -> fmt::Result {
        match self {
            Error::OpenConfig { filename, source } =>
                write!(f, "Could not open config at {}", filename.display()),
            Error::SaveConfig { source } =>
                write!(f, "Could not open config"),
            Error::UserIdInvalid { user_id, backtrace } =>
                write!(f, "The user id {} is invalid", user_id = user_id),
            Error::ConfigValidationFailed { key, checksum, source } =>
                write!(f, "Could not validate config with key {}: checksum was {}", key = key, checksum = checksum),
        }
    }
}
```

If no display format is specified, the variant's name will be used
by default. If the field is an underlying error, that error's
`Display` implementation will also be included.

#### `ErrorCompat`

Every variant that carries a backtrace will return a reference to
that backtrace.

```rust,ignore
impl snafu::ErrorCompat for Error {
    fn backtrace(&self) -> Option<&Backtrace> {
        match self {
            Error::OpenConfig { .. } => None,
            Error::SaveConfig { .. } => None,
            Error::UserIdInvalid { backtrace, .. } => Some(backtrace),
            Error::ConfigValidationFailed { .. } => None,
        }
    }
}
```

[Display]: std::fmt::Display
[ErrorCompat]: crate::ErrorCompat
[Error]: std::error::Error
[IntoError]: crate::IntoError
[cause]: std::error::Error::cause
[description]: std::error::Error::description
[source]: std::error::Error::source