# Notes on yocto class

https://bootlin.com/doc/training/sessions/hp-yocto-sep-2018/

## Questions

* how do you select different bbappend files? we need different changes to the mesa package depending on our target.  We did this by having 2 versions in different layers...is this right?
  Answer: use the conditional machine/soc_family postfixes to change config based on these variables

* is the rootfs already extracted somewhere? what about specific package builds that aren't installed?
  is there a way to get package binaries without installing to rootfs (like hprdinit)?
* Is there a way to build something, no include it in the rootfs, and then access it later for the initramfs? where would I find hprdinit if I remove it from install?
* where are MACHINEOVERRIDES and SOC_FAMILY defined? <layer>/conf/machine?

All documentation:

yoctoproject.org/docs/current/mega-manual/mega-manual.html

# Very Useful Yocto Build Info

* `${WORKDIR}` the parent directory of all the following directories (specific to the recipe)
* `${S}` where the sources are untarred (unless it's a git repo...then you should set `S = ${WORKDIR}/git`)
* `${WORKDIR}/${PN}-sysroot` where all the dependencies of the recipe are installed before building it (as of pyro)
* `${B}` the build directory where the sources are built, populated by `do_build`
* `${D}` the "staging dir" where you install files to for staging, populated by `do_install`

`do_package` takes all the files in `${D}` and creates various packages from them (i.e. `<pkg>.rpm`, `<pkg>-src.rpm`, `<pkg>-dev.rpm`). In OpenEmbedded `meta/conf/bitbake.conf` it wil show the "file filters" that are used to create all the packages.  The convention for these filters are `FILES_${PN} = "..."` or `FILES_${PN}-dev = "..."` and all the package filters are added to the `PACKAGES` variable.  If you want to create another package, you would add a new "file filter" and then add that variable to the `PACKAGES` variable. (slide 245)

Yocto knows how to package recipes based on what path you install it to.  We need to look up what paths yocto handles specially.  For example, you MUST install your headers files to a specific directory so that they get included in the -dev



Yocto uses the "pkg-dev.rpm" to install dependencies to build other recipes.


```
<VAR>_append = " something"
                ^
                need separator (space or ':', etc)
<VAR>_prepend = "something "
                          ^
                          need separator (space or ':', etc)

# remove all occurences of 'something' in <VAR>
<VAR>_remove = "something"


<VAR> += "something"
          ^
          no separator

# configure something for a specific machine
# MACHINE is any value in MACHINEOVERRIDES and/or SOC_FAMILY
<VAR>_<MACHINE>

# machine name cannot have underscores, but they can have dashes

# bitbake will always take the more specific one
# but both appends will occur if it matches both, but a direct set will
# only use the more specific one


Assignment operators:
=   expand the value when using the variable
:=  immediately expand the value
+=  append (with space) (not used very often, use _append usually)
=+  prepend (with space) (not used very often, use _prepend usually)
.=  append (without space)
=.  prepend (without space)
?=  assign if no value was previous assigned
??= same as previous with a lower precedence?
```

## Scopes

2 scopes, global, shared accross everything (`local.conf`, distribution conf and machine conf).
Then each recipe gets its own scope, their definitions are not shared with each other (I think)

## Common bitbake commands

```
# Compile a specific package
bitbake -c compile <package>
# add -b <recipe> to only parse the recipe you need

# List tasks for a package
bitbake -c listtasks <package>
```
