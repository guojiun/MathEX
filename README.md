# Introduction

MathEX is a mathematical expression evaluator written in C. It takes string as
input and returns floating-point number as a result.

## Features

* Supports arithmetic, bitwise and logical operators
* Supports variables
* Can be extended with custom functions
* Simple evaluation takes ~50 nanoseconds on an average PC
* Low memory usage makes it suitable for embedded systems
* Pure C99 with no external dependencies
* Good test coverage
* Easy to understand

## Example

```c
#include "expression.h"

/* Custom function that returns the sum of its two arguments */
static float add(struct expr_func *f, vec_expr_t args, void *c) {
    float a = expr_eval(&vec_nth(&args, 0));
    float b = expr_eval(&vec_nth(&args, 1));
    return a + b;
}

static struct expr_func user_funcs[] = {
    {"add", add, 0}, {NULL, NULL, 0},
};

int main() {
    const char *s = "x = 40, add(2, x)";
    struct expr_var_list vars = {0};
    struct expr *e = expr_create(s, strlen(s), &vars, user_funcs);
    if (!e) { printf("Syntax error"); return 1; }

    printf("result: %f\n", expr_eval(e));

    expr_destroy(e, &vars);
    return 0;
}
```

Output: `result: 42.000000`

## API

`struct expr *expr_create(const char *s, size_t len, struct expr_var_list
*vars, struct expr_func *funcs)` - returns compiled expression from the given
string. If expression uses variables - they are bound to `vars`, so you can
modify values before evaluation or check the results after the evaluation.

`float expr_eval(struct expr *e)` - evaluates compiled expression.

`void expr_destroy(struct expr *e, struct expr_var_list *vars)` - cleans up
memory. Parameters can be NULL (e.g. if you want to clean up expression, but
reuse variables for another expression).

`struct expr_var *expr_var(struct expr_var *vars, const char *s, size_t len)` -
returns/creates variable of the given name in the given list. This can be used
to get variable references to get/set them manually.

## Supported operators

* Arithmetics: `+`, `-`, `*`, `/`, `%` (remainder), `**` (power)
* Bitwise: `<<`, `>>`, `&`, `|`, `^` (xor or unary bitwise negation)
* Logical: `<`, `>`, `==`, `!=`, `<=`, `>=`, `&&`, `||`, `!` (unary not)
* Other: `=` (assignment, e.g. `x=y=5`), `,` (separates expressions or function parameters)

Only the following functions from libc are used to reduce the footprint and
make it easier to use:

* calloc, realloc and free - memory management
* isnan, isinf, fmodf, powf - math operations
* strlen, strncmp, strncpy, strtof - tokenizing and parsing

## Running tests

To run all the tests and benchmarks, do `make check`.

## License

Code is distributed under MIT X License that can be found in the `LICENSE` file.
