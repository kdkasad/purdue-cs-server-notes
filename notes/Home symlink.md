# Home symlink

Your home directory is set to `/homes/<username>`, but this is not actually
a real directory. It's a symbolic link to a directory on one of the NFS
systems, e.g. `/u/riker/u97/kkasad`.

## Starship/prompts

Because of this symlink, we get some weird behavior:

```
$ cd /homes/kkasad
$ pwd
/u/riker/u97/kkasad
```

Because the current directory doesn't start with the value of the `HOME`
variable, my shell prompt shows something like this even when I'm in
a directory inside my home directory:

```
kkasad on 🌐 data in /u/riker/u97/kkasad/cs352
$
```
This is annoying. I want it to show `~/cs352`. To fix this, I added this to my `~/.zshrc`:
```sh
if [ -L $HOME ]; then # if home is a symbolic link
	export HOME=$(readlink $HOME) # set home to the resolved path
	cd $HOME # cd to the resolved path so pwd matches
fi
```
