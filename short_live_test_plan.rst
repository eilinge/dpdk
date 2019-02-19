.. Copyright (c) <2016-2017>, Intel Corporation
   All rights reserved.

   Redistribution and use in source and binary forms, with or without
   modification, are permitted provided that the following conditions
   are met:

   - Redistributions of source code must retain the above copyright
     notice, this list of conditions and the following disclaimer.

   - Redistributions in binary form must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in
     the documentation and/or other materials provided with the
     distribution.

   - Neither the name of Intel Corporation nor the names of its
     contributors may be used to endorse or promote products derived
     from this software without specific prior written permission.

   THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
   "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
   LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
   FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
   COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
   INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
   (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
   SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
   HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
   STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
   ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
   OF THE POSSIBILITY OF SUCH DAMAGE.

=============================
Short-lived Application Tests
=============================

This feature is to reduce application start up time, and when exit, do more
clean up so that it could be re-run many times.

Prerequisites
-------------

To test this feature, need to using linux time and start testpmd by: create
and mount hugepage, must create more hugepages so that could measure time more
obviously::

        # echo 8192 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
        # mount -t hugetlbfs hugetlbfs /mnt/huge

Bind nic to DPDK::

        ./tools/dpdk_nic_bind.py -b igb_uio xxxx:xx:xx.x

Start testpmd using time::

        # echo quit | time ./testpmd -c 0x3 -n 4 -- -i


Test Case 1: basic fwd testing
------------------------------

1. Start testpmd::

      ./testpmd -c 0x3 -n 4 -- -i

2. Set fwd mac
3. Send packet from pkg
4. Check all packets could be fwd back

Test Case 2: Get start up time
------------------------------

1. Start testpmd::

    echo quit | time ./testpmd -c 0x3 -n 4 --huge-dir /mnt/huge -- -i

2. Get the time stats of the startup
3. Repeat step 1~2 for at least 5 times to get the average

Test Case 3: Clean up with Signal -- testpmd
--------------------------------------------

1. Create 4G hugepages, so that could save times when repeat::

    echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
    mount -t hugetlbfs hugetlbfs /mnt/huge1

2. Start testpmd::

    ./testpmd -c 0x3 -n 4 --huge-dir /mnt/huge1 -- -i

3. Set fwd mac
4. Send packets from pkg
5. Check all packets could be fwd back
6. Kill the testpmd in shell using below commands alternately::

      SIGINT:  pkill -2  testpmd
      SIGTERM: pkill -15 testpmd

7. Repeat step 1-6 for 20 times, and packet must be fwd back with no error for each time.


Test Case 4: Clean up with Signal -- l2fwd
------------------------------------------

1. Create 4G hugepages, so that could save times when repeat::

    echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
    mount -t hugetlbfs hugetlbfs /mnt/huge1

2. Start testpmd::

    ./l2fwd -c 0x3 -n 4 --huge-dir /mnt/huge1 -- -p 0x01

3. Set fwd mac
4. Send packets from pkg
5. Check all packets could be fwd back
6. Kill the testpmd in shell using below commands alternately::

      SIGINT:  pkill -2  l2fwd
      SIGTERM: pkill -15 l2fwd

7. Repeat step 1-6 for 20 times, and packet must be fwd back with no error for each time.

Test Case 5: Clean up with Signal -- l3fwd
------------------------------------------

1. Create 4G hugepages, so that could save times when repeat::

      echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
      mount -t hugetlbfs hugetlbfs /mnt/huge1

2. Start testpmd::

     ./l3fwd -c 0x3 -n 4 --huge-dir /mnt/huge1 -- -p 0x01 --config="(0,0,1)"

3. Set fwd mac
4. Send packets from pkg
5. Check all packets could be fwd back
6. Kill the testpmd in shell using below commands alternately::

     SIGINT:  pkill -2  l3fwd
     SIGTERM: pkill -15 l3fwd

7. Repeat step 1-6 for 20 times, and packet must be fwd back with no error for each time.
