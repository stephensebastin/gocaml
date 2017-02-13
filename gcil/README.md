Language Spec of GoCaml Intermediate Language
=============================================

GCIL (GoCaml Intermediate Language) is an IL for GoCaml. It is SSA and K-normalized form, low-layer representation of GoCaml code converted from AST. It maintains high level type information for high-level optimizations.

You can see the string representation of GCIL from command line.

```
gocaml -gcil test.ml
```

GCIL consists of basic blocks. Since MinCaml program has only one expression as root of AST, GCIL has one root basic block as program.

```
BEGIN: program

...

END: program
```

Basic block has a list of instructions to execute. Instructions are run sequentially.

GCIL is an SSA and K-normalized form. So one instruction contains one assignment and one operation.

```
var = operation
```

`var` is assigned only once and never be changed (SSA). Operation does only one thing (e.g. make a constant, add two integer, access to array element, ...) (K-normalization).


Below is a small example of `let foo = 1 + 2 in -foo` expression.

```
$k1 = int 1 ; type=int
$k2 = int 2 ; type=int
foo$t1 = binary + $k1 $k2 ; type=int
$k4 = ref foo$t1 ; type=int
$k5 = unary - $k4 ; type=int
```

Variable names follow below naming convention.

- `$k{number}`: Variables inserted automatically by K-normalization.
- `name$t{number}`: Alpha-transformed variables to identify variables which has the same name, but actually different because of shadowing. Original variable name is `name`.
- `name`: External symbols which will be linked by linker. It's not modified because external symbol names can't be changed.

## Instructions

| Instruction               | Description                                                                                     |
|---------------------------|-------------------------------------------------------------------------------------------------|
| `unit`                    | Create a `()` value                                                                             |
| `bool {constant}`         | Create a boolean value (`true` or `false`)                                                      |
| `int {constant}`          | Create an integer value                                                                         |
| `float {constant}`        | Create an floating point number value                                                           |
| `unary {op} {id}`         | Apply unary operator to `{id}`. `{op}` is `-` or `not` or `-.`.                                 |
| `binary {op} {id} {id}`   | Apply binary operator                                                                           |
| `ref {id}`                | Reference to `{id}` variable.                                                                   |
| `if {id} {block} {block}` | When `{id}` is true, then enter to first `{block}` . Otherwise inter to second `{block}`.       |
| `fun {ids...} {block}`    | Function. {ids...} are comma separated parameter IDs. `{block}` is its body.                    |
| `app {id} {ids...}`       | Apply function. First `{id}` is called function. Following comma separated IDs are arguments.   |
| `appcls {id} {ids...}`    | Apply function. First `{id}` is called closure. Following comma separated IDs are arguments.    |
| `appx {id} {ids...}`      | Apply function. First `{id}` is external symbol. Following comma separated IDs are arguments.   |
| `tuple {ids...}`          | Tuple value.                                                                                    |
| `array {id} {id}`         | Array value. First `{id}` is index and second `{id}` is element value.                          |
| `tplload {constant} {id}` | Load element value of tuple. Index must be constant.                                            |
| `arrload {id} {id}`       | Load element value of array. First `{id}` is index value.                                       |
| `arrstore {id} {id} {id}` | Store value to array. First `{id}` is index, second `{id}` is array, third `{id}` is set value. |
| `xref {id}`               | Reference to external symbol. `{id}` represents the symbol.                                     |
