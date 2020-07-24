# tetherback

Tools to create TWRP and nandroid-style backups of an Android device via a USB connection,
without using the device's internal storage or SD card.

To guarantee against backup corruption during transfer, it generates
[md5sums](https://en.wikipedia.org/wiki/md5sum) of the backup files on
the device and then verifies that they match on the host.

**WARNING:** This is a work in progress. I have personally tested it on the
following device/recovery/host combinations…

Other users have reported success—and
[issues](https://github.com/dlenski/tetherback/issues?q=is%3Aissue+is%3Aclosed)
and other operating
systems, various versions of Windows and Mac OS X.

My target is Huawei / Honor LDN-Lxx devices and modded/workaround are done for them. Should works for all others Oreo devices too.

## Requirements and installation

tetherback requires Python 3.3+. In addition, it depends on:

* [TWRP recovery](https://twrp.me/) installed on your rooted Android device
* [`adb`](https://en.wikipedia.org/wiki/Android_software_development#ADB) (Android Debug Bridge) command-line tools
* `progressbar2` and `tabulate` packages from PyPI (fetched automatically during `pip install`; see below)

Install with `pip3` to automatically fetch Python dependencies. (Note that on most systems, `pip` invokes
the Python 2.x version, while `pip3` invokes the Python 3.x version.)

```
# Install Python3 (this is Windows installation) from https://www.python.org/downloads/windows/
# or for Linux https://www.python.org/downloads/source/

# Install latest development version
$ pip3 install https://github.com/dlenski/tetherback/archive/HEAD.zip

# Install a tagged release
# (replace "0.9.1" with newer of the tag/release version numbers on the "Releases" page,if there is any.)
$ pip3 install https://github.com/dlenski/tetherback/archive/0.9.1.zip
```

## Usage

Boot your device into TWRP recovery and connect it via USB. Ensure that it's visible to `adb`:
If you use Windows, open cmd instead of bash and ignore all $ from commands.

```bash 
$ adb devices
List of devices attached
0123deadbeaf5f5f	recovery
```

* Make a TWRP-style backup over ADB. This saves a gzipped image of the
  `ramdisk` partition as `ramdisk.emmc.win`,
   and saves the *contents* of the
  `/system` and `/data` partitions as tarballs named
  `system.ext4.win` and `data.ext4.win`:
  
* WORKAROUND - tetherback can not unmount "ramdisk" partition on its own, stops with failure at begining of back up.
               So, that line is bypassed and you had manually unmount ALL partitions in TWRP -> Mount field.

    ```bash
    $ tetherback
    tetherback v0.9.1
    Found ADB version 1.0.36
    Using default transfer method: adb exec-out pipe (--exec-out)
    Device reports kernel 3.18.66-gc555e98-dirty
    Device reports TWRP version 3.3.1-0
    Reading partition map for mmcblk0 (55 partitions)...
    partition map: 100%
    Reading partition map for mmcblk0rpmb (0 partitions)...
    partition map: 100%
    Reading partition map for mmcblk1 (1 partitions)...
    partition map: N/A%
    NG: partition mmcblk1p1 has no PARTNAME in its uevent file
    Please report this issue at https://github.com/dlenski/tetherback/issues
    Please post the entire output from tetherback!
    partition map: 100%
    Saving backup images in .\twrp-backup-2020-07-24--10-09-48/ ...
    Saving partition ramdisk (mmcblk0p42), 16 MiB uncompressed...
    ramdisk.emmc.win: 100%  11.3 MiB/s  11.2 MiB
    Saving tarball of mmcblk0p54 (mounted at /system), 2624 MiB uncompressed...
    system.ext4.win: 100%  10.4 MiB/s   1.1 GiB
    Saving tarball of mmcblk0p55 (mounted at /data), 25403 MiB uncompressed...
    data.ext4.win: 100%  10.6 MiB/s   7.8 GiB
    Backup complete.
    ```

* Make a "nandroid"-style backup over ADB. This saves gzipped images
  of the partitions labeled `boot`, `system`, and `userdata` (named
  `<label>.img.gz`):

    ```bash
    $ tetherback -N
    tetherback v0.8
    Found ADB version 1.0.32
    Using default transfer method: adb exec-out pipe (--exec-out)
    Device reports kernel 3.4.0-bricked-hammerhead-twrp-g7b77eb4
    Device reports TWRP version 3.0.0-0
    Reading partition map for mmcblk0 (29 partitions)...
      partition map: 100% Time: 0:00:03
    Reading partition map for mmcblk0rpmb (0 partitions)...
      partition map: 100%
    Saving backup images in nandroid-backup-2016-07-03--18-15-03/ ...
    Saving partition boot (mmcblk0p19), 22 MiB uncompressed...
      mmcblk0p19: 100%   3.07 MB/s  16.3 MiB
    Saving partition system (mmcblk0p25), 1024 MiB uncompressed...
      mmcblk0p25: 100%   1.76 MB/s  343.7 MiB
    Saving partition userdata (mmcblk0p28), 13089 MiB uncompressed...
      mmcblk0p28: 100%   1.80 MB/s  6.4 GiB
    ```

### Additional options

* Extra partitions can be included with the `-X`/`--extra` and `--extra-raw`
  options; for example, `-X kernel -X vendor` to backup the
  [Huawei Y7 Prime 2018 partitions](https://github.com/adeii/huawei_london_twrp/blob/omni-7.1/recovery/root/etc/recovery.fstab).

    * With `--extra-raw`, the extra partition will *always* be saved as a raw image, rather than as a tarball, even if it is a
      mountable filesystem and tetherback is run in TWRP backup mode. <--- Use this if you still has encrypted /data and internal SD! Like: 
      
          tetherback -U --extra-raw data --extra-raw media

* The partition map and backup plan will be printed with
  `-v`/`--verbose` (or use `-0`/`--dry-run` to **only** print it, and
  skip the actual backup). For example, the following partition map
  and backup plan will be shown for a Nexus 5 with the standard
  partition layout:

    ```
    BLOCK DEVICE    PARTITION NAME      SIZE (KiB)  MOUNT POINT    FSTYPE
    --------------  ----------------  ------------  -------------  --------
    mmcblk0p1       modem                    65536
    ...
    mmcblk0p19      boot                     22528
    ...
    mmcblk0p25      system                 1048576  /system        ext4
    ...
    mmcblk0p28      userdata              13404138  /data          ext4
    mmcblk0p29      grow                         5
                    Total:                15388143

    PARTITION NAME    FILENAME         FORMAT
    ----------------  ---------------  -------------------------------------------------
    boot              boot.emmc.win    gzipped raw image
    system            system.ext4.win  tar -cz -p
    userdata          data.ext4.win    tar -cz -p --exclude="media*" --exclude="*-cache"
    ```

* Additional options allow exclusion or inclusion of standard partitions:

    ```
    -M, --media           Include /data/media* in TWRP backup (deprecated: default behavior)
    -D, --data-cache      Include /data/*-cache in TWRP backup
    -R, --recovery        Include recovery partition in backup
    -C, --cache           Include /cache partition in backup
    -U, --no-userdata     Omit /data partition from backup (implies --no-media)
    -E, --no-media        Omit /data/media* from TWRP backup
    -S, --no-system       Omit /system partition from backup
    -B, --no-boot         Omit boot partition from backup
    ```

## Motivation

I've been frustrated by the fact that all the Android recovery backup
tools save their backups _on a filesystem on the device itself_.

* [TWRP recovery](https://twrp.me/)
  ([code](https://github.com/omnirom/android_bootable_recovery))
  creates a mixture of raw partition images and tarballs, and **stores
  the backups on the device itself.**
* Same with [CWM recovery](http://clockworkmod.com/rommanager) , which
  creates nandroid-style backup images (just raw partition images) and
  again **stores them on the device itself.**

This is problematic for several reasons:

1. Most modern Android smartphones don't have a microSD card slot.
2. There may not be enough space on the device's own filesystem to back up its own contents.
3. Getting the large backup files off of the device requires an extra, slow transfer step.

Clearly I'm not the only one with this problem:

* http://android.stackexchange.com/questions/64354/how-to-do-a-full-nandroid-backup-via-pc
* http://android.stackexchange.com/questions/47975/is-there-a-way-to-do-nandroid-backup-directly-to-pc-and-then-restore-it-directly

I found that [**@inhies**](https://github.com/inhies) had already
created a shell script to do a TWRP-style backup over USB
([Gist](https://gist.github.com/inhies/5069663)) and decided to try to
put together a more polished version of this.

## Issues

One of the very annoying issues with `adb` is that
[`adb shell` is not 8-bit-clean](http://stackoverflow.com/questions/13578416):
line endings in the input and output get mangled, so it cannot easily
be used to pipe binary data to and from the device. The common
workaround for this is to use TCP forwarding and `netcat` (see
[this answer on StackOverflow](http://stackoverflow.com/a/34216105/20789)),
but this is more cumbersome to code, and prone to strange timing
issues. There is a better way to make the output pipe 8-bit-clean, by
changing the terminal settings
([another StackOverflow answer](http://stackoverflow.com/a/20141481/20789)),
though apparently it does not work with Windows builds of `adb`.

*By default*, tetherback uses TCP forwarding with older versions of `adb`, and an `exec-out` binary pipe with newer versions (1.0.32+).
If you have problems, please try
`--base64` for a slow but reliable transfer method, and please
[report any data corruption issues](http://github.com/dlenski/tetherback/issues). If
your host OS is Linux, `--pipe` should be faster and more reliable.

  ```
  -t, --tcp             ADB TCP forwarding (fast, should work with any host
                        OS, but prone to timing problems)
  -x, --exec-out        ADB exec-out binary pipe (should work with any host
                        OS, but only with newer versions of adb and TWRP)
  -6, --base64          Base64 pipe (very slow, should work with any host OS)
  -P, --pipe            Binary pipe (fast, but probably only works
                        on Linux hosts)
  ```


## License

GPL v3 or newer
