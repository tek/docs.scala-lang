# Introduction

An advanced mechanism for formatting type errors and inspecting missing
implicits has been introduced in Scala 2.15.
It is based on the compiler plugin [splain](https://github.com/tek/splain).

This tool abstracts several classses of compiler errors with simple data types
that can be processed by a few built-in routines as well as
[user-provided analyzer plugins](/overviews/plugins/index.html).

The most significant feature is the illustration of chains of implicit instances
that allows a user to determine the root cause of an implicit error:

![implicits](/resources/img/implicits-circe.jpg)

# Basic Configuration

To activate and configure advanced error formatting, use the following compiler
options:

* `-Vexplain-implicits:enable` Enable all standard features
* `-Vexplain-implicits:no-color` Disable colors in the output

## Additional Configuration

`-Vexplain-implicits:verbose-tree` shows the implicits between the error site and the
root cause, see [#implicit-resolution-chains].

`-Vexplain-implicits-trunc-refined` reduces the verbosity of refined types, see
[#truncating-refined-types].

# Features

The error formatting engine provides the following enhancements:

# Infix Types

Instead of `shapeless.::[A, HNil]`, prints `A :: HNil`.

# Found/Required Types

Rather than printing up to four types, only the dealiased types are shown as a colored diff:

![foundreq](/resources/img/foundreq.jpg)

<!-- special consideration for `shapeless.Record` (available at [link to analyzer plugin](https://github.com/???)): -->

<!-- ![foundreq_record](/resources/img/foundreq_record.jpg) -->

# Implicit Resolution Chains

When an implicit is not found, only the outermost error at the invocation point
is printed by the regular error reporter. This can be expanded with the
compiler flag `-Xlog-implicits`, but that also shows all invalid implicits for
parameters that have been resolved successfully.

This feature prints a compact list of all involved implicits:

![compact](/resources/img/implicits-compact.jpg)

Here, `!I` stands for *could not find implicit value*, the name of the implicit
parameter is in yellow, and its type in green.

If the parameter `verbose-tree` is set, all intermediate implicits will be
printed, potentially spanning tens of lines.
An example of this is the circe error at the top of the page.

For comparison, this is the regular compiler output for this case:

```
[error] /path/Example.scala:20:5: could not find implicit value for parameter a: io.circe.Decoder[A] 
[error]   A.fun
[error]     ^
```

# Infix Type and Type Argument Line Breaking

Types longer than 79 characters will be split into multiple lines:

```
implicit error;
!I e: String
f invalid because
!I impPar4: List[
  (
    VeryLongTypeName ::::
    VeryLongTypeName ::::
    VeryLongTypeName ::::
    VeryLongTypeName
  )
  ::::
  (Short :::: Short) ::::
  (
    VeryLongTypeName ::::
    VeryLongTypeName ::::
    VeryLongTypeName ::::
    VeryLongTypeName
  )
  ::::
  VeryLongTypeName ::::
  VeryLongTypeName ::::
  VeryLongTypeName ::::
  VeryLongTypeName
]
```

# Truncating Refined Types

A type of the shape `T { type A = X; type B = Y }` will be displayed as `T
{...}` if the option `-Vexplain-implicits-max-refined` is set to a value `/= 0` and
the refinement's length is greater than the value.
