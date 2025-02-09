# Flash Translation Layer

## Overview

This repository contains the simple Flash Translation Layers made on the Linux Environments.

Furthermore, this repository not only works on the ramdisk but also works
on various devices(e.g. Zoned Block Device, and bluedbm, etc.).

If you have any questions about this project, please contact the maintainer (ss5kijun@gmail.com).

## Build

> These instructions based on the Ubuntu(>=16.04) environments

### Prerequisite

Before you build this repository, you must install some packages from
the package manager.

```bash
sudo apt update -y
sudo apt install -y git make gcc g++ libglib2.0-dev libiberty-dev
```

After you download the packages you must receive this project code
by using `git clone` like below.

```
git clone --recursive ${REPOSITORY_URL}
```

Now you move to this repository's project directory root by using `mv`.

> If you want to use a module like Zoned Block Device module, you must set the value
> of the `USE_ZONE_DEVICE` variable to 1 in the `Makefile`.

Additionally, you must install a valid library for each module.
Each module's requirements are as follows.

- zone: [libzbd](https://github.com/westerndigitalcorporation/libzbd)
- bluedbm: [libmemio](https://github.com/pnuoslab/Flash-Board-Tester)

> Moreover, before you run the Zoned Block Device-based program,
> You must check that you give the super-user privileges to run
> the Zoned Block Device-based programs.

### Test

Before you execute the test and related things, you must install the below tools.

- [flawfinder](https://dwheeler.com/flawfinder/)
- [cppcheck](https://cppcheck.sourceforge.io/)
- [lizard](https://github.com/terryyin/lizard)

You can check the source code status by using `make check`.

If you want to generate test files, execute the below command.

```bash
make clean && make -j$(nproc) test USE_LOG_SILENT=1
```

After the build finish, you can get the various test files from the results.
Run those files to test the project's module works correctly.

### Execution

If you want to generate a release program through the `main.c`,
then you must execute the below commands.

```bash
make clean && make -j$(nproc)
```

Now, you can run that program by using `./a.out`

Note that this repositories `main.c` file conducts
the integration test of this project.

### Installation

This project also supports generating the static library
for using the external project. If you want to use it,
please follow the below commands.

```bash
make clean
make -j$(nproc)
sudo make install
```

Then you must add some items to your compile commands.

```makefile
LIBS = -lpthread -lftl $(shell pkg-config --libs glib-2.0)
CFLAGS = $(shell pkg-config --cflags glib-2.0) -I${FTL_INCLUDE_PATH}
```

Normally, `libftl.a` file's include path is existed in the
`/usr/local/include/ftl`. Therefore, you just set the `${FTL_INCLUDE_PATH}`
to `/usr/local/include/ftl`.

## Example

You can make a program like the below. This program executes simple random read/write.

```c
#include <assert.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>

#include "module.h"
#include "flash.h"
#include "page.h"
#include "log.h"
#include "device.h"

int main(void)
{
        struct flash_device *flash = NULL;
        char buffer[8192];
        assert(0 == module_init(PAGE_FTL_MODULE, &flash, RAMDISK_MODULE));
        pr_info("module initialize\n");
        flash->f_op->open(flash, NULL, O_CREAT | O_RDWR);
        for (int i = 0; i < 8192 * 10; i++) {
                int num;
                size_t sector;
                num = i * 2;
                memset(buffer, 0, 8192);
                *(int *)buffer = num;
                sector = rand() % (1 << 31);
                flash->f_op->write(flash, buffer, sizeof(int), sector);
                pr_info("write value: %d\n", *(int *)buffer);
                memset(buffer, 0, 8192);
                flash->f_op->read(flash, buffer, sizeof(int), sector);
                pr_info("read value: %d\n", *(int *)buffer);
                if (i % 8192 * 5 == 0) {
                        flash->f_op->ioctl(flash, PAGE_FTL_IOCTL_TRIM);
                }
        }
        flash->f_op->close(flash);
        assert(0 == module_exit(flash));
        pr_info("module deallcation\n");

        return 0;
}
```

## How to get this project's documents

You can get this program's documentation file by using `doxygen -s Doxyfile`.
