+++
title = "flesh: a shell"
date = 2025-12-18
+++

Today we're gonna build a shell in C! Many thanks to [Stephen Brennan](https://brennan.io/2015/01/16/write-a-shell-in-c/) for writing a tutorial for it and putting me in the right direction, but we're gonna do it from scratch for learning.

What does a shell do? It asks the user for a command, then executes that command. Simple enough.

We're gonna use [getline](https://www.man7.org/linux/man-pages/man3/getline.3.html) (`stdio.h`) to read lines, here is an example where the program reads from `stdin` and prints it back:

```c
char *line = NULL;
size_t size = 0;

printf("speak your command: ");
getline(&line, &size, stdin);

printf("received: %s", line);

free(line);
```

First we initialize `line` to `NULL`, this is because `getline` automatically allocs the memory for it. `size` will be the size of the line. **Important:** the line _must_ be freed, no matter if `getline` fails or succeeds.

But what's the use of it if it doesn't execute anything? To do that, we must split the line into _arguments_ (for example the command `nvim main.c` has two arguments: `nvim` and `main.c`). For splitting we're gonna use [strtok](https://www.man7.org/linux/man-pages/man3/strtok.3.html) (`string.h`):

```c
int argc = 0;
char **args = (char **)malloc(ARGS_MAX_COUNT * sizeof(char *));
char *token = strtok(line, " ");
size_t token_len = strlen(token);

while (1) {
  args[argc] = (char *)malloc(token_len);
  memcpy(args[argc], token, token_len);

  argc++;
  token = strtok(NULL, " ");

  if (token == NULL)
    break;

  token_len = strlen(token);
}
```

`argc` will be the count of our arguments. `args` is an array of char arrays (technically its a pointer to a pointer). `token` is the current word (first call to `strtok` will return the string from the start to the delimiter, in this case the first word). `token_len` is the length of the current word. Then we create an infinite loop? nope, we'll break out of it when the line ends (`token` is `NULL`). Inside the loop we allocate memory to `args[argc]` and then copy `token_len` bytes of memory from `token` to `args[argc]`. Then we call `strtok` again, but this time with `NULL` so it parses the same string as before (it continues parsing). I defined `ARGS_MAX_COUNT` as 32.

**Important**: don't forget to `free(args)` later!

Note: after peeking at Stephen Brennan's tutorial I realised I could set directly: `args[argc] = token`.

We can also move the code to a function:

```c
char **get_args(char *line, int *out_argc) {
  int argc = 0;
  char **args = (char **)malloc(ARGS_MAX_COUNT * sizeof(char *));
  char *token = strtok(line, " ");

  while (token != NULL) {
    args[argc++] = token;

    token = strtok(NULL, " ");
  }

  *out_argc = argc;
  return args;
}
```

and call it like this in `main`:

```c
int argc = 0;
char **args = get_args(line, &argc);
```

**Another update\*\***: I ran into a problem where executing a single command would always give the error: `error: No such file or directory` and i spent some good time on it before realising that the last argument included a `\n` for example inputing `echo` would result in `echo\n` and the system couldn't find that in path and couldn't execute it! We can fix this by adding these lines in `get_args` after the while loop:

```c
if (argc > 0) {
  int arg_len = strlen(args[argc - 1]);

  if (args[argc - 1][arg_len - 1] == '\n')
    args[argc - 1][arg_len - 1] = '\0'; // replace \n with \0
}
```

Next we want to execute some things! We're gonna use [fork](https://www.man7.org/linux/man-pages/man2/fork.2.html) (`unistd.h`) for that which duplicates the process. It returns -1 upon error, 0 for the child process and the `pid` of the parent process for the parent process. We're also gonna use [waitpid](https://man7.org/linux/man-pages/man3/wait.3p.html) (`sys/wait.h`) to wait for changes in the child process.

```c
int execute(char **args) {
  pid_t pid = fork();
  int status;

  if (pid == -1) {
    return -1;
  } else if (pid == 0) {
    // child process
    if (execvp(args[0], args) == -1) {
      exit(0);
    }
  } else {
    // parent process
    waitpid(pid, &status, WUNTRACED);

    while (status != WIFEXITED(status) && status != WIFSIGNALED(status)) {
      waitpid(pid, &status, WUNTRACED);
    }
  }

  return 0;
}
```

So what this function does is it _forks_ (clones) our program, the child will execute the `else if (pid == 0)` and the parent will execute the `else`. The child executes using [execvp](https://www.man7.org/linux/man-pages/man3/exec.3.html) from which v - vector, the second argument is a vector (array) of arguments and p - path, it will ask the system for the full path of the program (from $PATH). If it fails it returns -1. Now the parent waits for changes in the child process, and it waits until it is exited or killed. `execute` is called in main like this:

```c
int status = execute(args);

if (status == -1) {
  perror("flesh");
}
```

We also have to handle shell builtins now, we're gonna implement: `cd` and `exit` with simple if's. We're also gonna wrap everything in a while loop. Here is the full `main` function:

```c
int main() {
  char *line = NULL;
  size_t size = 0;

  int argc = 0;
  char **args = NULL;

  int status = 0;

  while (1) {
    free(line);
    line = NULL;

    printf("-> ");
    getline(&line, &size, stdin);

    argc = 0;
    free(args);
    args = NULL;

    args = get_args(line, &argc);

    if (strcmp(args[0], "cd") == 0) {
      if (args[1] == NULL) {
        printf("cd where?\n");
      } else {
        if (chdir(args[1]) != 0)
          perror("flesh");
      }
    } else if (strcmp(args[0], "exit") == 0) {
      exit(0);
    } else {
      status = execute(args);

      if (status == -1) {
        perror("flesh");
      }
    }
  }

  return 0;
}
```

One last thing I want to implement and that's expanding `~` to $HOME envinorment variable. In `get_args`, beside `args[argc++] = token` we also add:

```c
// expand ~ to $HOME
if (token[0] == '~') {
  char *home = getenv("HOME");

  int home_len = strlen(home);
  int token_len = strlen(token);

  if (token_len == 2 && token[1] == '\n') {
    token_len = 1;
    token[1] = '\0';
  }

  // its just ~
  if (token_len == 1) {
    args[argc] = malloc(home_len + 1);

    memcpy(args[argc], home, home_len);

    args[argc][home_len] = '\0';
  } else {
    args[argc] = malloc(
        home_len + token_len); // we don't need to add one since token_len
                               // has one extra char that we won't use: ~

    snprintf(args[argc], home_len + token_len, "%s%s", home,
             token + 1); // token + 1 so we skip ~
  }

  argc++;
} else {
  args[argc++] = token;
}
```

That's it! To run it: `gcc main.c -o flesh -O3 && ./flesh`. This is a very primitive shell probably filled with bugs but we learned something today! God bless.
