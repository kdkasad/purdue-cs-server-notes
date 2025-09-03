# Cleanroom (temporary empty home filesystem)

Sometimes it's useful to test stuff with an empty home filesystem. For example,
if you want to test installing/setting up something as a new user without an
existing configuration. Normally, you might just make a new user on the machine
you're using, but we obviously can't do that, so the next best thing is to just
clean the home filesystem.

To do this, we'll use `namespaces(7)` to let us mount arbitrary filesystems
without affecting the filesystem for all of the other users/processes on the
machine.

To easily manipulate namespaces, we can use `bwrap(1)`. I make a shell function
for this so I don't have to remember the command:

```sh
cleanroom () {
        bwrap --ro-bind / / --dev /dev --proc /proc --tmpfs /u --tmpfs $(realpath $HOME) /bin/zsh
}
```

> [!NOTE]
> This mounts the `tmpfs` on the destination of your
> [home symlink](./Home symlink.md), so the `tmpfs` will be at something like
> `/u/riker/u97/kkasad` and `/homes/kkasad` will still point to that directory.
