+++
title = "nvim virtual colors"
date = 2025-12-13
+++

the first technical post of my blog will be about my neovim plugin that displays css colors directly in the editor. like "color decorators" in vscode

the plugin's source code is available on [github](https://github.com/aki-masaki/nvim-virtual-colors)

![screenshot](https://raw.githubusercontent.com/aki-masaki/nvim-virtual-colors/refs/heads/main/screenshot.png)

this is the folder structure of the project

```
.
├── lua
│   └── nvim-virtual-colors.lua
├── README.md
└── screenshot.png

2 directories, 3 files
```

code inside `lua/` will not run unless required. it has only one file which exposes a module (in the file the module is called M, but in `init.lua` neovim config file it is exposed as the name of the file, in this case `nvim-virtual-colors`)

let's take a look inside that file

we can see it creates a variable called `M` (for module), it declares functions in the module and then returns it (by returning we can call `require` on it)

the functions in the module, briefly explained:

- `hide` - hides the virtual colors
- `show` - hides the virtual colors if already present, then shows them
- `init_autocmd` - autocmd is used to listen to events, here we initialize the listeners
- `detect_colors` - the name is a bit misleading since this function is also responsible for creating the virtual text
- `setup` - this function is meant to be called in the neovim config file to configure the plugin (and in this plugin's case to actually turn it on)

now lets dive into each function's implementation and explain every part:)

```lua
function M.hide()
  if M.ns_id ~= nil then
    vim.api.nvim_buf_clear_namespace(0, M.ns_id, 0, -1)
  end
end
```

`M.ns_id` is the id of the [namespace](https://neovim.io/doc/user/api.html#namespace) of my plugin. each virtual text must belong to a namespace

we call [`nvim_buf_clear_namespace`](https://neovim.io/doc/user/api.html#nvim_buf_clear_namespace()) to clear extmarks (virtual text) from the entire file (the function is used to clear from a region but we pass line_start = 0 and line_end = -1 to clear the whole buffer)

```lua
function M.show()
  if M.ns_id ~= nil then
    vim.api.nvim_buf_clear_namespace(0, M.ns_id, 0, -1)
  end

  M.detect_colors()
end
```

nothing too interesting here...we just check if we already created a namespace (we create it inside `setup`), and if yes, clear it

```lua
function M.init_autocmd()
  local files_pattern = { '*.css' }

  vim.api.nvim_create_autocmd('InsertEnter', {
    pattern = files_pattern,
    callback = function(ev)
      M.hide()
    end
  })

  vim.api.nvim_create_autocmd('InsertLeave', {
    pattern = files_pattern,
    callback = function(ev)
      M.detect_colors()
    end
  })

  vim.api.nvim_create_autocmd('BufEnter', {
    pattern = files_pattern,
    callback = function(ev)
      M.show()
    end
  })
end
```

here `files_pattern` says for which files to bind to those events. '*.css' means every file that ends in `.css`

as we stated before, autocmd's are used to listen to events. the events present here are:

- `InsertEnter` - when the user *enters* insert mode (generally with `a` or `i`)
- `InsertLeave` - when the user *leaves* insert mode (generally with `esc`)
- `BufEnter` - when the user enters a buffer (also fires when reopened)

documentation for `nvim_create_autocmd` can be found [here](https://neovim.io/doc/user/api.html#nvim_create_autocmd()) and for `autocmd` in general [here](https://neovim.io/doc/user/autocmd.html)

before addressing the elephant in the room (the `detect_colors` function) let's see the `setup` function first

```lua
function M.setup(opts)
  opts = opts or {}

  M.opts = {
    display = opts.display or 'bg',
    display_on_sign_column = opts.display_on_sign_column or true
  }

  M.ns_id = vim.api.nvim_create_namespace("virtual-colors")

  M.init_autocmd()

  vim.api.nvim_create_user_command("HideVirtualColors", M.hide, {})
  vim.api.nvim_create_user_command("ShowVirtualColors", M.show, {})
end
```

it takes `opts` (a table) as a parameter for *options* (configuring the plugin). if not present, it defaults to `{}`

then we set the options in `M.opts` (defaulting them if not present in `opts`), we create the namespace and store the id, we init the autocmd event listeners and we define two user commands: `HideVirtualColors` and `ShowVirtualColors` (this permits the user to type these into the command line) 

now lets see what `detect_colors` contains!

```lua
function M.detect_colors()
  local lines = vim.api.nvim_buf_get_lines(0, 0, -1, false)
  local offset = 0

  for i, line in ipairs(lines) do
    while true do
      local rgb_index = line:find('rgb', offset)
      local color_index = line:find('#', offset)
      local semic_index = line:find(';', offset)

      if color_index ~= nil and semic_index ~= nil then
        local len = semic_index - color_index - 1

        if not (len == 3 or len == 6) then
          return
        end

        local color = line:sub(color_index + 1, semic_index - 1)

        if len == 3 then
          color = color:rep(2)
        end

        local red   = tonumber(color:sub(1, 2), 16)
        local green = tonumber(color:sub(3, 4), 16)
        local blue  = tonumber(color:sub(5, 6), 16)

        local opts

        if M.opts.display == 'bg' or M.opts.display == 'bg-fn' then
          local fg

          if red + green + blue < 200 then
            fg = "#ffffff"
          else
            fg = "#000000"
          end

          vim.api.nvim_set_hl(0, color, { bg = '#' .. color , fg = fg})

          if M.opts.display == 'bg' then
            opts = {
              virt_text = {{'#' .. color, color}},
              virt_text_pos = 'overlay',
            }
          else
            opts = {
              virt_text = {{'#', color}},
              virt_text_pos = 'overlay',
            }
          end
        else
          vim.api.nvim_set_hl(0, color, { fg = '#' .. color})

          opts = {
            virt_text = {{'█', color}},
            virt_text_pos = 'inline',
          }
        end

        if M.opts.display_on_sign_column == true then
          opts.sign_text = '  '
          opts.sign_hl_group = color
        end

        local mark_id = vim.api.nvim_buf_set_extmark(0, M.ns_id, i - 1, color_index - 1, opts)

        offset = offset + semic_index + 1
      elseif rgb_index ~= nil and semic_index ~= nil then
        local lpar = line:find('%(', offset)
        local rpar = line:find('%)', offset)

        if lpar == nil or rpar == nil then
          break
        end

        local comma1 = line:find(',', lpar)
        
        if comma1 == nil then
          break
        end

        local comma2 = line:find(',', comma1 + 1)

        if comma2 == nil then
          break
        end

        local red = tonumber(line:sub(lpar + 1, comma1 - 1))
        local green = tonumber(line:sub(comma1 + 1, comma2 - 1))
        local blue = tonumber(line:sub(comma2 + 1, rpar - 1))

        if red == nil or red < 0 or red > 255 or green == nil or green < 0 or green > 255 or blue == nil or blue < 0 or blue > 255 then
          break
        end

        local color = string.format("%02x%02x%02x", red, green, blue)

        if M.opts.display == 'bg' or M.opts.display == 'bg-fn' then
          local fg

          if red + green + blue < 200 then
            fg = "#ffffff"
          else
            fg = "#000000"
          end

          vim.api.nvim_set_hl(0, color, { bg = '#' .. color , fg = fg})

          if M.opts.display == 'bg' then
            opts = {
              virt_text = {{string.format("rgb(%d, %d, %d)", red, green, blue), color}},
              virt_text_pos = 'overlay',
            }
          else
            opts = {
              virt_text = {{'rgb', color}},
              virt_text_pos = 'overlay',
            }
          end
        else
          vim.api.nvim_set_hl(0, color, { fg = '#' .. color})

          opts = {
            virt_text = {{'█', color}},
            virt_text_pos = 'inline',
          }
        end

        if M.opts.display_on_sign_column == true then
          opts.sign_text = '  '
          opts.sign_hl_group = color
        end

        local mark_id = vim.api.nvim_buf_set_extmark(0, M.ns_id, i - 1, rgb_index - 1, opts)

        offset = offset + rpar + 1
      else
        offset = 0

        break
      end
    end
  end
end
```

woa! this function is 144 lines of code, thats a lot... lets split it up

first we get the lines of the buffer and we initialise `offset` (youll see what this is used for later) to `0`

then we create a for loop `for i, line in ipairs(lines) do` and inside another while loop? the while loop is used to scan multiple colors in one line (example css: `color: #fff; background-color: #000;`), we break out of the loop when no color is present (after the ones we parsed already)

now inside the while loop

we first declare three variables:
```lua
local rgb_index = line:find('rgb', offset)
local color_index = line:find('#', offset)
local semic_index = line:find(';', offset)
```

they store the indeces in line where the specific character is found (starting from `offset` so if a line has two '#', and we wish to find the second one, we can start scanning *after* the first)\
**important**: they are `nil` when they are not found

```
color: #000;
1234567^ color_index = 8
```

after that, we can see a conditional: `if color_index ~= nil and semic_index ~= nil then` this means that a '#' is found **and** a ';' is found (color must be hex)

inside that if, we can see this:
```lua
local len = semic_index - color_index - 1

if not (len == 3 or len == 6) then
    return
end
```

why does `semic_index - color_index - 1` return the length?
```
color: #000;
       ^   $    ^ = color_index = 8; $ = semic_index = 12;
```

but...`12 - 8` is 4, but there are only 3 zeroes! thats because we also include ';' in our length, thats why we substract 1 from the difference

then we check to make sure that len is either 3 or 6

```lua
local color = line:sub(color_index + 1, semic_index - 1)

if len == 3 then
  color = color:rep(2)
end
```

the function `sub` takes a substring of line, from `color_index + 1` (the character after '#') until `semic_index - 1` (the character before ';') (`#00ff00;` -> `00ff00`). then we check if the length is 3, in which case we just repeat it (`000` -> `000000`)

```lua
local red   = tonumber(color:sub(1, 2), 16)
local green = tonumber(color:sub(3, 4), 16)
local blue  = tonumber(color:sub(5, 6), 16)
```

then we convert the hex color into red, green and blue components. the second parameter to `tonumber` represents the base from which to convert from.

then, we have conditionals so we have to split up:

- if display is either 'bg' or 'bg-fn':

```lua
local fg

if red + green + blue < 200 then
    fg = "#ffffff"
else
    fg = "#000000"
end
```

here we check the contrast of the color, if its low (dark) we set the fg (text color) to white, otherwise if its high (light) we set the fg to black.

```lua
vim.api.nvim_set_hl(0, color, { bg = '#' .. color , fg = fg})
```

then we create a highlight group (in neovim, they are used to define the syntax color)

then we split up...again...

```lua
if M.opts.display == 'bg' then
    opts = {
      virt_text = {{'#' .. color, color}},
      virt_text_pos = 'overlay',
    }
else
    opts = {
      virt_text = {{'#', color}},
      virt_text_pos = 'overlay',
    }
end
```

not so different tho, if its 'bg' we set the virtual text to be the whole expression, otherwise just '#'.
the second item inside `virt_text` is the name of the highlight group to apply (here we used the one we created earlier)
`virt_text_pos = 'overlay'` means the virtual text will be displayed *over* the original text, replacing it.

- if display is not those:

```lua
vim.api.nvim_set_hl(0, color, { fg = '#' .. color})

opts = {
    virt_text = {{'█', color}},
    virt_text_pos = 'inline',
}
```

here we dont have to worry about contrast since no background color is used. instead it displays a [block character](https://en.wikipedia.org/wiki/Block_Elements#:~:text=Full%20block)

notice how here we used `virt_text_pos = 'inline'`? this means that, it will shift the characters to the right to make space for the virtual text.

```lua
if M.opts.display_on_sign_column == true then
  opts.sign_text = '  '
  opts.sign_hl_group = color
end
```

this shows the color on the signcolumn (which is before the line numbers)

```lua
local mark_id = vim.api.nvim_buf_set_extmark(0, M.ns_id, i - 1, color_index - 1, opts)
```

we *finally* get to create the actual virtual text (called [extmark](https://neovim.io/doc/user/api.html#extmark))

we pass `0` as the first parameter for the current buffer, `M.ns_id` for the namespace it belongs to, `i - 1` is the line, we substract one because lua is 1-based but this function is 0-based...the same for `color_index - 1` (column), and we pass the `opts` we defined earlier

and in the end

```lua
offset = offset + semic_index + 1
```

we add `semic_index + 1` to offset, which means next time start scanning from the character after ';'

now we got to this: `elseif rgb_index ~= nil and semic_index ~= nil then` which means 'rgb' was found **and** a ';' was found (color must be rgb)

```lua
local lpar = line:find('%(', offset)
local rpar = line:find('%)', offset)

if lpar == nil or rpar == nil then
    break
end
```

we try to find `(` and `)`, we use `%` before them since in lua, `()` are called [*magic characters*](https://www.lua.org/pil/20.2.html#:~:text=magic%20characters) which must be escaped. if we can't find them, we break the loop

then we do the same thing for commas:

```lua
local comma1 = line:find(',', lpar)

if comma1 == nil then
    break
end

local comma2 = line:find(',', comma1 + 1)

if comma2 == nil then
    break
end
```

we start searching for the first comma after '(' and for the second comma after the character after the first comma. if either cant be found, break the loop

```lua
local red = tonumber(line:sub(lpar + 1, comma1 - 1))
local green = tonumber(line:sub(comma1 + 1, comma2 - 1))
local blue = tonumber(line:sub(comma2 + 1, rpar - 1))
```

then we extract the red, green and blue components from the string. red is between '(' and the first comma, green is after the first comma and before the second comma, and blue is after the second comma and before ')'. we substract and add `1` so we dont include those characters in the number

```lua
if red == nil or red < 0 or red > 255 or green == nil or green < 0 or green > 255 or blue == nil or blue < 0 or blue > 255 then
    break
end
```

then we see a long if, that checks that all three components: are *not* `nil` and `0 <= value <= 255`

```lua
local color = string.format("%02x%02x%02x", red, green, blue)
```

whats with that?? in lua `%x` formats a number as a hexadecimal ([doc](https://www.luadocs.com/docs/functions/string/format)). but we also need the number to be always two digits long, thats what `02` is for, if its smaller than `10` (two characters) it *pads* (adds before it) a `0`

doing that for every component gives us the hexadecimal representation of the color

then the next section is as before, for 'bg' or 'bg-fn' we do this we do that. the only difference for rgb is here:

```lua
if M.opts.display == 'bg' then
    opts = {
      virt_text = {{string.format("rgb(%d, %d, %d)", red, green, blue), color}},
      virt_text_pos = 'overlay',
    }
else
    opts = {
      virt_text = {{'rgb', color}},
      virt_text_pos = 'overlay',
    }
end
```

we cant just use `'rgb' .. components` since we have parantheses, commas. so we just format it, in lua `%d` formats a number as an integer

the process for setting the extmark is the same but instead of `color_index` we use `rgb_index`. and for `offset` we add `rpar + 1` which means start scanning after ')'

then we have an `else`, branch that gets executed when neither 'rgb' nor a '#' are found, which means the line doesn't contain any color. in which case we set `offset` to `0` and we break the loop, going to parse the next line

and...thats all!
