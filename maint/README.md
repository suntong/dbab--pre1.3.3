# To Build

## Building structure

Directory structure:

```
.
`--bld
  `--dbab-1.3.1
    `--assets
    `--bin
    `--debian
      `--dbab
        `--DEBIAN
        `--etc
          `--dbab
          `--init.d
        `--lib
		...
`--dbab
  `--.git
    `--branches
    `--hooks
    `--info
    `--logs
	...
  `--debian
    `--source
  `--maint
  `--src
    `--assets
    `--bin

```

## Building commands

It is under `dbab/bld/dbab-1.3.x` to issue


```
../../dbab/maint/build-d
debuild -i -us -uc -b
debuild -us -uc -S -sa
```
