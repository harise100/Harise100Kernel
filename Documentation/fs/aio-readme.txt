aio: convert the ioctx list to radix tree
Date	Wed, 3 Apr 2013 16:20:48 +0300

When using a large number of threads performing AIO operations the
IOCTX list may get a significant number of entries which will cause
significant overhead. For example, when running this fio script:

rw=randrw; size=256k ;directory=/mnt/fio; ioengine=libaio; iodepth=1
blocksize=1024; numjobs=512; thread; loops=100

on an EXT2 filesystem mounted on top of a ramdisk we can observe up to
30% CPU time spent by lookup_ioctx:

 32.51%  [guest.kernel]  [g] lookup_ioctx
  9.19%  [guest.kernel]  [g] __lock_acquire.isra.28
  4.40%  [guest.kernel]  [g] lock_release
  4.19%  [guest.kernel]  [g] sched_clock_local
  3.86%  [guest.kernel]  [g] local_clock
  3.68%  [guest.kernel]  [g] native_sched_clock
  3.08%  [guest.kernel]  [g] sched_clock_cpu
  2.64%  [guest.kernel]  [g] lock_release_holdtime.part.11
  2.60%  [guest.kernel]  [g] memcpy
  2.33%  [guest.kernel]  [g] lock_acquired
  2.25%  [guest.kernel]  [g] lock_acquire
  1.84%  [guest.kernel]  [g] do_io_submit

This patchs converts the ioctx list to a radix tree. For a performance
comparison the above FIO script was run on a 2 sockets 8 core
machine. This are the results for the original list based
implementation and for the radix tree based implementation:

cores         1         2         4         8         16        32
list        111025 ms  62219 ms  34193 ms  22998 ms  19335 ms  15956 ms
radix        75400 ms  42668 ms  23923 ms  17206 ms  15820 ms  13295 ms
% of radix
relative      68%       69%       70%       75%       82%       83%
to list

To consider the impact of the patch on the typical case of having
only one ctx per process the following FIO script was run:

rw=randrw; size=100m ;directory=/mnt/fio; ioengine=libaio; iodepth=1
blocksize=1024; numjobs=1; thread; loops=100

on the same system and the results are the following:

list        65241 ms
radix       65402 ms
% of radix
relative    100.25%
to list

Cc: Andi Kleen <ak@linux.intel.com>

Signed-off-by: Octavian Purdila <octavian.purdila@intel.com>
modified for Mako hybrid from LKML

Signed-off-by: Paul Reioux <reioux@gmail.com>
