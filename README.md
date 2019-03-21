# Styleguides

## TL;DR - Absolute Minimum

- Use the `.sv` extension for SystemVerilog files and the `.svh` extension for SystemVerilog files that are `` `include``d.
- Use only ASCII characters with Unix-style line endings (i.e., LF aka `\n`).
- Wrap code at 100 characters per line.
- Indent with two spaces per level. Do not use tabs.
- Delete trailing whitespace at the end of every line. Delete trailing newlines at the end of every file. Every file ends with *exactly* one LF.
- Use `begin` and `end` unless a statement fits on a single line. Put the `begin` on the same line as the condition and the `end` on its own line (where it may be followed by an `else`, see below).
- Prefix module instances with `i_`, e.g.: `i_prefetcher`.
- Append the direction to port names: `_o`, `_i`, `_io`.
- For active-low signals put an additional `n`: `n` for internal signals, `_no`, `_ni`, `_nio` for ports.
- Denote output of a register with `_q` and the input with `_d`. When pipelining a signal, append a number starting at 2, e.g., `_q2`.
- Avoid `defines` and `ifdefs` **as much as possible**. Use parameters and packages instead.
- If one file deviates from this style but consistently uses a different style, stick to the other style unless you are changing almost all code of that file.

## Naming

|         Construct          |       Style            |
|----------------------------|------------------------|
| Declaration                | `lower_snake_case`     |
| Instance names             | `i_lower_snake_case`   |
| Signals                    | `lower_snake_case`     |
| Variable, functions, tasks | `lower_snake_case`     |
| `` `define``               | `ALL_CAPS`             |
| Tunable Parameters         | `CamelCase`            |
| Constants                  | `ALL_CAPS`             |
| Enumerated types           | `lower_snake_case_e`   |
| Other `typedef`s           | `lower_snake_case_t`   |
| Enumeration value names    | `UpperCamelCase`       |
| Generate blocks            | `gen_lower_snake_case` |

## Files and Project Structure

- Each repository must contain a License file indicating the License at use.
- Please link to this style guide if your project adheres to it.

## Coding Style

- Keep the files tidy. No superfluous line breaks, align ports on a common boundary.
- Name dedicated signals wiring `module foo` (output) with `module bar` (input) `signal_foo_bar`
- Use interfaces to connect component instances and *only* to connect instances. Do not perform logic operations on interface signals; various tools have bugs with it.
- Use a flat port list (i.e., no interfaces) for modules. When a module is a reusable unit and has ports that can be represented as interfaces, provide a wrapper named `module_name_wrap` that exposes interfaces and does *nothing* else than connecting interfaces to the corresponding ports.
- Do not put overly large comment headers. Nevertheless, try to structure your HDL code, e.g.:

    ```
    // ------------------------------------
    // CSR - Control and Status Registers
    // ------------------------------------
    ```

<!-- - Specify memory map and integration rules while coding, using the `crazy88` (TODO: Link to Documentation) syntax. -->
- Put `begin` statements on the same level as the block qualifier (K&R style), for example:

    ```verilog
    module A (
      input  logic flush_i,
      output logic stall_o
    );

      logic whatever_signal;

      always_comb begin
        stall_o = 1'b0;
        if (flush_i) begin
          // do some stuff here
          stall_o = 1'b1;
        end else if (whatever_signal) begin
          // do some other stuff
        end
      end
    endmodule
    ```
    > The rationale is that extra lines for `begin/else/end` carry no information at all. They even may prevent parts of the code to not have enough space on some screens. Process blocks on the other hand are more self-contained and multiple process blocks are not required to be visible at the same time.
    > The intention behind this is to keep code which is closely related together (like the code in an `always` block). It then should easily fit on a single screen.

- This also applies to `case` statements.  `begin` and `end` may be omitted iff the entire case item (i.e., the case expression and the statement) fits on a single line.  Use a consistent style within one `case` statement.

    ```verilog
    unique case (state_q)
      Idle: begin
        state_d = Something;
        valid_o = 1'b1;
      end
      Something: begin
        // Multiple lines of other assignments
        if (ready_i) begin
          state_d = Idle;
        end
      end
      default: begin
        state_d = Idle;
      end
    endcase
    ```

    ```verilog
    unique case (stride_i)
      2'd0: state_d = Idle;
      2'd1: state_d = Bubble;
      2'd2: state_d = Forward;
      2'd3: state_d = Hold;
    endcase
    ```

- Give generics a meaningful type e.g.: `parameter int unsigned AsidWidth = 1`. The default type is a signed integer which in most of the time does not make an awful lot of sense for hardware.

- Name `generate` blocks with `begin : gen_name`. Do not put the name of the generate into a comment after the end; that's redundant. Do not use the `generate` keyword; it's redundant. For example:

    ```verilog
    for (genvar i=0; i<10; i++) begin : gen_ten_times
      // something to generate 10x
    end

    if (PARAM == 0) begin : gen_no_param
      // something
    end else begin: gen_param
      // something else
    end // param_gen
    ```
    > This simplifies synthesis and back-end scripts significantly and make them more vendor independent.

-  Name enumerate types with a `_e` suffix and `structs` that are used as types with a `_t` suffix:

    ```verilog
    typedef enum logic [1:0] { PrivUser, PrivSupervisor, PrivMachine } priv_lvl_e;
    typedef struct packed {
      logic [1:0]  rw;
      priv_lvl_e   priv_lvl;
      logic  [7:0] address;
    } csr_addr_t;
    ```
    ```verilog
    module A (
      input logic [11:0] address_i
    );

      csr_addr_t csr_addr;

      assign csr_addr = csr_addr_t'(address_i);

      always_comb begin
        if (csr_addr.priv_lvl == UMode) begin
          // do something fancy with this signal
        end
      end
    endmodule
    ```

- Use [EditorConfig](http://editorconfig.org/) to make your editor consistently apply a style:

    ```
    # top-most EditorConfig file
    root = true

    # Unix-style newlines with a newline ending every file
    [*]
    end_of_line = lf
    insert_final_newline = true
    trim_trailing_whitespace = true
    max_line_length = 100
    # 2 space indentation
    [*.{sv, svh, v, vhd}]
    indent_style = space
    indent_size = 2
    ```

    There are plug-ins for almost any sane editor. The same example `.editorconfig` can also be found in this repository.

## Git

- Do not push to master unless you are the owner (as in responsible) of a repository. If you want to contribute, create a branch and open a Pull Request.
- Separate subject from body with a blank line.
- Limit the subject line to 50 characters.
- Capitalize the subject line.
- Do not end the subject line with a period.
- Use the imperative mood in the subject line.
- Use the present tense ("Add feature" not "Added feature").
- Wrap the body at 72 characters.
- Use the body to explain what and why vs. how.
- Consider starting the commit message with an applicable emoji:
    * :sparkles: `:sparkles:` When introducing a new feature
    * :art: `:art:` Improving the format/structure of the code
    * :zap: `:zap:` When improving performance
    * :fire: `:fire` Removing code or files.
    * :memo: `:memo:` When writing docs
    * :bug: `:bug:` When fixing a bug
    * :wastebasket: `:wastebasket:` When removing code or files
    * :green_heart: `:green_heart:` When fixing the CI build
    * :construction_worker: `:construction_worker:` Adding CI build system
    * :white_check_mark: `:white_check_mark:` When adding tests
    * :lock: `:lock:` When dealing with security
    * :arrow_up: `:arrow_up:` When upgrading dependencies
    * :arrow_down: `:arrow_down:` When downgrading dependencies
    * :rotating_light: `:rotating_light:` When removing linter warnings
    * :pencil2: `:pencil2:` Fixing typos
    * :recycle: `:recycle:` Refactoring code.
    * :boom: `:boom:` Introducing breaking changes
    * :truck: `:truck:` Moving or renaming files.
    * :space_invader: `:space_invader:` When fixing something synthesis related
    * :beers: `:beer:` Writing code drunkenly.
    * :ok_hand: `:ok_hand` Updating code due to code review changes
    * :building_construction: `:building_construction:` Making architectural changes.
    * :wrench: `:wrench:` Tooling
    * :construction: `:construction:` Work In Progress WIP
    * :bookmark: `:bookmark:` version tag

For a detailed why and how please refer to one of the multiple [resources](https://chris.beams.io/posts/git-commit/) regarding git commit messages.

If you use `vi` for your commit message, consider to put the following snippet inside your `~/.vimrc`:

```
autocmd Filetype gitcommit setlocal spell textwidth=72s
```
