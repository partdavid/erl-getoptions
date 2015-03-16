I made certain design decisions when thinking about this module, so I wanted to document them here. Some kinds of option syntax are supported, and some aren't. There is a wide variety of ways to parse command-line options and I wanted to note why I decided on the subset that I did.

## Run-Together Single-Letter Options ##

When an argument like `-cone` is encountered, it could be interpreted one of three ways:

  * `-c -o -n -e` - individual options glommed together
  * `-c one` - the `-c` option has an option argument of `one`
  * `-cone` - long option introduced by `--`

I decided against supporting long options not distinguishable from single-letter options.

One way of deciding between the remaining two ways is whether the `-c` option accepts an argument. If it does, you can decide on the latter interpretation, otherwise the former. That's what `getoptions` does. The other way is to interpret them "tar-style", and line up subsequent arguments in order as option arguments:

  * `-c -o -n -e Arg1 Arg2 Arg3` Since `-c` accepts an argument, its option argument is Arg1; the next option letter with an argument letter would take Arg2, and so on.

This is sort of idiosyncratically `tar`, which (in my opinion) doesn't really even have option letters so much as action letters. The traditional way to write `tar` commands is without the `-` anyway: the first argument is a sort of compound command made up of action letters.

## Optional Option Arguments ##

Some option parsing libraries support optional arguments, where an option can be present, but without an argument, and optionally can pass an argument. This is not supported, because it makes it easy for the resulting interface to be ambiguous, and it requires that you drop support for positional option arguments.

Conceptually, this:

  * `--with-lib` or `--with-lib=library-location`

is the same as:

  * `--has-lib`
  * `--lib-location=library-location`

and the resulting interface is unambiguous. I'm not a fan of forcing people to do their operation the "right" way, but something that makes it very easy to make mistakes should be avoided.

## Variable Options ##

Some unix commands like `kill`, `head` and `tail` accept a number as an option, and other variations on this scheme are possible. This isn't supported by `erl-getoptions` (except as far as you can enumerate the options) because it't difficult to combine it with other standard schemes (like running together option letters). And it's easily expressed as a regular option as well (as `tail` and `head` do with `-n` anyway).

## Options Introduced by Other Characters ##

Some commands accept options introduced by a plus (`+`) character or another character (for example, DOS-style commands and their `/` options). I chose not to bother implementing this, because I don't think it expresses anything not expressable with regular options, and leads to ambiguous interfaces where the meaning of `-` is overloaded: is it an option introducer or an option-turner-offer?

## Unsupported Option Schemes ##

The result of these various decisions is that `getoptions` does not support some noteworthy option processing schemes:

  * `tar` uses single-letter options in a block followed by positionally aligned option arguments
  * The `erl` interpreter uses `+` options, long options introduced by a single hyphen and multi-argument options
  * GNU autoconf `configure` scripts use options with optional arguments
  * `head` and `tail` use variable options
  * Many unix commands stop option processing at the first non-option argument, many do not. `erl-getoptions` assumes they can be intermixed: option processing stops with `--`.

## Other Notes ##

If underscore (`_`) could be a valid option argument, that option should not have a type of `atom`. If it does, you will not be able to distinguish between options that are not set (`getoptions:get_opt(Opt, OptList)` returns `'_'`) and options that are set to the underscore atom. A workaround would be to use a type of `string` instead, and handle atom creation yourself.