## Usage ##

```
readmeter [options] meter [meter ...]
   --help                      Print usage message
   --version                   Print version message
   -v, --verbose               Verbose output (more -v means more verbose)
   -d, --debug <level>         Print debug output at the given level
   -m, --minimum <reading>     Minimum level to report (floating-point number; default 0.1)
   -c, --config <meter>=<file> Use alternate configuration file for formatting readings
                               (can be specified multiple times)
   -o, --output <file>         Put output in <file>. Can be specified multiple times for
                               multiple files.

readmeter reads the named meter devices, writing the meter levels that exceed the minimum
level onto the standard output, or the output file(s) if specified.
```

## Option Specification ##
In order to provide the above interface, a possible `main/1` function might look like:

```
#!/usr/bin/env escript

-define(optspec,
        [
         {help, boolean},
         {version, boolean},
         {verbose, v, count},
         {debug, d, integer},
         {minimum, m, float},
         {config, c, {keylist, string}},
         {output, o, {list, string}}
        ]).

-define(defaults,
        [
         {verbose, 0},
         {minimum, 0.1},
         {debug, 0}
        ]).
         

main([Args]) ->
   {Opts, Meters} = getoptions:extract_options(Args, ?optspec, ?defaults),
   io:format("~p~n", [Opts]).

.
.
.
```

Note that by providing defaults for `verbose` and `debug` they can be more easily used in tests, for example:

```
   V = getoptions:get_opt(verbose, Opts),
   if
      V > 0 -> io:format("Reading meter...", []);
      true -> true
   end
   .
   .
   .
```