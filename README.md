# Reverse shell for GitHub Actions

This action creates a reverse shell from your GitHub job which can help with debugging your
workflows. You will be able to SSH into your runner to poke around and figure out what is happening.

It should work on a great deal of various GitHub runner configurations. It is compatible with
**Linux**, **macOS** & **Windows** runners. There are very few dependencies so it will also work in
containerized **Docker** jobs even lightweight distributions like **Alpine** and **Arch**. Take a
look at the [test
matrix](https://github.com/laverdet/console/blob/main/.github/workflows/console.yml).

The best shell strategy is [upterm](https://upterm.dev) which provides a pseudo-terminal and free
jump host service. The `laverdet/console@v1` action reads your GitHub public keys and uses them for
authentication.

There is an additional strategy based on [netcat](https://en.wikipedia.org/wiki/Netcat) which should
run on any conceivable host that has an internet connection.


# Getting Started

Just add `uses: laverdet/console@v1`. Also consider adding an `if: failure()` if you only want to
start a terminal on failure.

```yaml
jobs:
  build:
    # [...]
    - uses: laverdet/console@v1
      if: failure()
```

After that take a look at your action log and there will be a jump host you can connect to. Just
copy & paste the `ssh` command. Multiple `ssh` sessions are supported so if you want to connect in
different windows or whatever that's fine.

While you are connected, the workflow will be stopped at the `console` action. You can type
`continue-workflow` if you want the workflow to continue. Once the workflow finishes it will
probably kick you off the runner.


# Netcat

If for some reason the upterm solution doesn't work then you can try netcat. This strategy requires
a separate host with a public IP address-- the GitHub runner will connect directly to your host.

The experience is much worse since you are interacting directly with a command interpreter without a
pseudo-terminal but it will get the job done in a pinch. This tool helped troubleshooting &
development of the above tool.

Netcat is not encrypted so don't do anything you wouldn't mind the world knowing.

```yaml
jobs:
  build:
    # [...]
    - uses: laverdet/console/nc@v1
      with:
        host: 1.2.3.4 # your ip address
        port: 2323    # [optional]
```

If you're a weird hacker you can also use the `laverdet/console/netcat@v1` rule which will setup
some version of netcat on your runner. The action will add a netcat to your path, but it might be
called `nc`, `nc64.exe`, `ncat`, etc depending on which flavor it downloaded. You can access the
`nc` output for the binary name and `flavor` if you need behavioral information [traditional vs. GNU
vs. OpenBSD, etc]. I recommend reading the source code for the action if this is something you need.
