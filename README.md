# Styleguides

## Coding Style

- If the existing coding style is sane - keep to it.
- Keep the files tidy. No superfluous line breaks, align ports on a common boundary.
- Don't use tabs, use spaces. If you really want to use tabs then use them consistently.
- Use 4 spaces to open a new indentation level.
- All signal and module names should be lower case with underscores as whitespace replacements (e.g.: `fetch_busy`).
- Instantiation of modules should be prefix with `i_`, e.g.: `i_prefetcher`
- For port definitions keep a post-fix direction (`_o`, `_i`).
- For active low signals put an additional (`_no`, `_ni`).
- Denote output of ff with `_q` and the input with `_n`.
- Name dedicated signals wiring `module A` (output) with `module B` (input) `signal_a_b`
- Do not use CamelCase!
- Do not put overly large comment headers. Nevertheless, try to structure your HDL code, e.g.:
```
  // ------------------------------------
  // CSR - Control and Status Registers
  // ------------------------------------
```
- Put `begin` statements on the same level as the block qualifier, for example:
```verilog
module A (
    input logic flush_i
);

    logic whatever_signal;

    always_comb begin
        if (flush_i) begin
            // do some stuff here
        end else if (whatever_signal) begin
            // do some other stuff
        end
    end
endmodule
```
- The exception to the former rule can be the begin of an `always` block where the `begin` can be placed on a new line, for example:
```verilog
module A (
    input logic flush_i
);

    logic whatever_signal;

    always_comb
    begin
        if (flush_i) begin
            // do some stuff here
        end else if (whatever_signal) begin
            // do some other stuff
        end
    end
endmodule
```
The intention behind this is to keep code which is closely related together (like the code in an `always` block). It then should easily fit on a single screen.
- Give generics a meaningful type e.g.: `parameter int unsigned ASID_WIDTH = 1`. The default type is a signed integer which in most of the time does not make an awful lot of sense for hardware.
- Name `structs` which you are going to use as a signal bundle without a postfix, for example
```verilog
typedef struct packed {
     logic [63:0] cause; // cause of exception
     logic [63:0] tval;  // additional information of causing exception (e.g.: instruction causing it),
                         // address of LD/ST fault
     logic        valid;
} exception;
```
    ```verilog
    module A (
        input exception ex_i
    );
    
        // ......
    endmodule
    ```
- On the other hand name `structs` which are used as types with a post-fix `_t`:
```verilog
typedef struct packed {
    logic [1:0]  rw;
    priv_lvl_t   priv_lvl;
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
            if (csr_addr.priv_lvl == U_MODE) begin
                // do something fancy with this signal
            end
        end
    endmodule
    ```
## Git Considerations

- Do not push to master, if you want to add a feature do it in your branch
- Separate subject from body with a blank line
- Limit the subject line to 50 characters
- Capitalize the subject line
- Do not end the subject line with a period
- Use the imperative mood in the subject line
- Use the present tense ("Add feature" not "Added feature")
- Wrap the body at 72 characters
- Use the body to explain what and why vs. how
- Consider starting the commit message with an applicable emoji:
    * :art: `:art:` when improving the format/structure of the code
    * :racehorse: `:racehorse:` when improving performance
    * :memo: `:memo:` when writing docs
    * :penguin: `:penguin:` when fixing something on Linux
    * :apple: `:apple:` when fixing something on macOS
    * :checkered_flag: `:checkered_flag:` when fixing something on Windows
    * :bug: `:bug:` when fixing a bug
    * :fire: `:fire:` when removing code or files
    * :green_heart: `:green_heart:` when fixing the CI build
    * :white_check_mark: `:white_check_mark:` when adding tests
    * :lock: `:lock:` when dealing with security
    * :arrow_up: `:arrow_up:` when upgrading dependencies
    * :arrow_down: `:arrow_down:` when downgrading dependencies
    * :shirt: `:shirt:` when removing linter warnings
    * :scissors: `:scissors:` when restructuring your HDL
    * :space_invader: `:space_invader:` when fixing something synthesis related

For a detailed why and how please refer to one of the multiple [resources](https://chris.beams.io/posts/git-commit/) regarding git commit messages.

If you use `vi` for your commit message, consider to put the following snippet inside your `~/.vimrc`:
```
autocmd Filetype gitcommit setlocal spell textwidth=72s
```
