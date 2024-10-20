
TinyRV1 ISA
==========================================================================

Below is a minimal example using [Wavedrom](https://wavedrom.com/). 
Wavedrom is a nice tool that has fairly widespread usage/support, but
not as great documentation. You can find examples of what it can do
[here](https://pypi.org/project/wavedrom/), and specific examples
of the [bit-field](https://www.npmjs.com/package/bit-field) library
looking [here](https://observablehq.com/@drom/wavedrom-bit-field-guide?collection=@drom/bitfield)

This is an example encoding for the ADD instruction using the
[markdown-wavedrom](https://pypi.org/project/markdown-wavedrom/) extension.
The only issue I've found is that you can't specify that a name is
all 0's as a literal (unlike the actual editor), but must instead
have a 1 in the more significant bits that ends up not being
displayed.

wavedrom(
  {reg: [
    {bits: 7, name: 0b0110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1000},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: 0b10000000}
  ]}
)

To get this to work locally, you must also

```bash
pip install markdown-wavedrom
```

(I already updated the build workflow)