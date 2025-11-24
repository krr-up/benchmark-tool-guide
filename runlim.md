# runlim

The runlim program by Armin Biere is used to run and monitor our benchmark runs and
is required to use the default templates.

To install runlim simply clone the [repository][repo] and build the executable:

```bash
git clone https://github.com/arminbiere/runlim
cd runlim
./configure.sh && make
```

Afterwards move the resulting `runlim` executable to the `programs/` folder
created by `btool init`. You might have to set the correct file permissions:

```bash
chmod +111 ./runlim
```

[repor]: https://github.com/arminbiere/runlim
