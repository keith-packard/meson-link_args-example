Meson's executable stanza may include a 'link_args' parameter which
offers a way to pass data to the linker which is target specific.

The values passed to link_args are added to the link command after any
local libraries and object files. There doesn't appear to be any way
to specify link options to be placed before the local libraries.

This example shows the limitations of this current design by creating
a local library which defines a function, `toy_puts`, a
a program which uses a function `defsym_puts`, with the goal of using
the linker `--defsym` command line parameter to map `defsym_puts` to
`toy_puts`.

The resulting ninja file executes this command:

	$ cc  -o toy-example 'toy-example@exe/toy-example.c.o' \
	  -Wl,--as-needed -Wl,--no-undefined \
	  -Wl,--start-group libtoy-library.a -Wl,--end-group \
	  -Wl,--defsym=defsym_puts=toy_puts \
	  '-Wl,-rpath,$ORIGIN/' \
	  -Wl,-rpath-link,/home/keithp/src/toy-example/build/

To make this work, we need to do this instead:

	$ cc  -o toy-example 'toy-example@exe/toy-example.c.o' \
	  -Wl,--defsym=defsym_puts=toy_puts \
	  -Wl,--as-needed -Wl,--no-undefined \
	  -Wl,--start-group libtoy-library.a -Wl,--end-group \
	  '-Wl,-rpath,$ORIGIN/' \
	  -Wl,-rpath-link,/home/keithp/src/toy-example/build/

The --defsym option now comes before the library that defines the
target symbol.
