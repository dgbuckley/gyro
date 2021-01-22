# Getting Started

## Installation

In order to install gyro, all you need to do is extract one of the [release
tarballs](https://github.com/mattnite/gyro/releases) for your system and add
the single static binary to your PATH.

## Building

If you'd like to build from source, the only thing you need is the zig compiler:

```
git clone https://github.com/mattnite/gyro.git --recursive
zig build -Dbootstrap
```

The `-Dbootstrap` is required because gyro uses git submodules to do the
initial build. After that one can build gyro with gyro, this will pull packages
from the package index instead of using git submodules.

```
gyro build
```

(Note: you might need to move the original gyro binary from the `zig-cache`
first).  This command wraps `zig build`, so you can pass arguements like you
normally would, like `gyro build test` to run your unit tests.

## Consuming packages

To find potential zig packages you'd like to use:
- [astrolabe.pm](https://astrolabe.pm), the default package index
- [zpm](https://zpm.random-projects.net), a site that lists cool zig projects
  and where to find them
- search github for `#zig` and `#zig-package` tags

If you want to use code from a package from astrolabe, then all you need to do
is `gyro add <package name>`, else if you want to use a Github repository as a
dependency then all that's required is `gyro add <user>/<repo>`.

Packages are exposed to your `build.zig` file through a struct in
`@import("gyro")`, and you can simply add them using a `addAllTo()` function,
and then `@import()` in your code.

Assume there is a `hello_world` package available to on the index, we'd add it
to our project like so:

```
gyro add hello_world
```

build.zig:

```zig
const Builder = @import("std").build.Builder;
const pkgs = @import("gyro").pkgs;

pub fn build(b: *Builder) void {
    const exe = b.addExecutable("main", "src/main.zig");
    pkgs.addAllTo(exe);
    exe.install();
}
``` 

main.zig:

```zig
const hw = @import("hello_world");

pub fn main() !void {
    try hw.greet();
}
```

If you want to "link" a specific package to an object, the packages you depend
on are accessed like `pkgs.<package name>` so in the example above you could
instead do `exe.addPackage(pkgs.hello_world)`.

### Build dependencies

It's also possible to use packaged code in your `build.zig`, since this would
only run at build time and most likely not required in your application or
library these are kept separate from your regular dependencies in your project file.

When you want to add a dependency as a build dep, all you need to do is add
`--build-dep` to the gyro invocation. For example, let's assume I need to do
some parsing with a package called `mecha`:

```
gyro add --build-dep mecha
```

and in my `build.zig`:

```zig
const Builder = @import("std").build.Builder;
const pkgs = @import("gyro").pkgs;
const mecha = @import("mecha");

pub fn build(b: *Builder) void {
    const exe = b.addExecutable("main", "src/main.zig");
    pkgs.addAllTo(exe);
    exe.install();
}
```

## Producing packages

The easiest way for an existing project to adopt gyro is to start by running
`gyro init <user>/<repo>` to grab metadata from their Github project. From
there the package maintainer to finish the init process by defining a few more things:
- the root file, it is `src/main.zig` by default
- file globs describing which files are actually part of the package. It is
  encouraged to include the license and readme, as well as testing code.
- any other packages if the repo exports multiple repos (and their
  corresponding root files of course)
- dependencies (see previous section).

### A note on projects and dependencies

A project may export multiple packages, each with their own root file. For the
sake of simplicity all packages in a project share the same dependencies.
