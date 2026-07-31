# sbum

A TUI for [Slurm](https://slurm.schedmd.com/), based on [turm](https://github.com/karimknaebel/turm), which provides a convenient way to manage your cluster jobs.

<img alt="turm demo" src="https://github.com/user-attachments/assets/7daade50-def3-4bf8-bf12-df311438094e" width="100%" />

`sbum` accepts the same options as `squeue` (see [man squeue](https://slurm.schedmd.com/squeue.html#SECTION_OPTIONS)). Use `sbum --help` to get a list of all available options. For example, to show only your own jobs, sorted by descending job ID, including all job states (i.e., including completed and failed jobs):
```shell
sbum --me --sort=-id --states=ALL
```

## Installation

Install from this GitHub repository (requires a Rust toolchain for the first build):

```shell
# Run once via uvx.
uvx --from git+https://github.com/laitifranz/turm sbum --me

# Or install persistently.
uv tool install git+https://github.com/laitifranz/turm

# With cargo.
cargo install --git https://github.com/laitifranz/turm
```

### Shell Completion (optional)

#### Bash

In your `.bashrc`, add the following line:
```bash
eval "$(sbum completion bash)"
```

#### Zsh

In your `.zshrc`, add the following line:
```zsh
eval "$(sbum completion zsh)"
```

#### Fish

In your `config.fish` or in a separate `completions/sbum.fish` file, add the following line:
```fish
sbum completion fish | source
```

## How it works

`sbum` obtains information about jobs by parsing the output of `squeue`.
The reason for this is that `squeue` is available on all Slurm clusters, and running it periodically is not too expensive for the Slurm controller ( particularly when [filtering by user](https://slurm.schedmd.com/squeue.html#OPT_user)).
In contrast, Slurm's C API is unstable, and Slurm's REST API is not always available and can be costly for the Slurm controller.
Another advantage is that we get free support for the exact same CLI flags as `squeue`, which users are already familiar with, for filtering and sorting the jobs.

### Resource usage

TL;DR: `sbum` ≈ `watch -n2 squeue` + `tail -f slurm-log.out`

Special care has been taken to ensure that `sbum` is as lightweight as possible in terms of its impact on the Slurm controller and its file I/O operations.
The job queue is updated every two seconds by running `squeue`.
When there are many jobs in the queue, it is advisable to specify a single user to reduce the load on the Slurm controller (see [squeue --user](https://slurm.schedmd.com/squeue.html#OPT_user)).
`sbum` updates the currently displayed log file on every inotify modify notification, and it only reads the newly appended lines after the initial read.
However, since inotify notifications are not supported for remote file systems, such as NFS, `sbum` also polls the file for newly appended bytes every two seconds.

## Development without Slurm

For local UI testing, this repository includes mocks for `squeue`, `scancel`, `scontrol`, and `sinfo`:

```shell
PATH=scripts/mock-slurm/bin:$PATH cargo run -- --me
```

The mock commands read/write files in `scripts/mock-slurm/logs`, so you can test log rendering and control actions without a Slurm install.

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=karimknaebel/turm&type=Date)](https://www.star-history.com/#karimknaebel/turm&Date)
