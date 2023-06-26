# Создадим папку, которую будем использовать в качестве корневого каталога.
    root@185-46-10-155:/# mkdir seminar1
    root@185-46-10-155:/# mkdir seminar1/bin
# Скопируем командную оболочку dash и необходимые библиотеки в наш корневой каталог.
    root@185-46-10-155:/# cp /bin/dash seminar1/bin
    root@185-46-10-155:/# ldd /bin/dash
            linux-vdso.so.1 (0x00007fffe6798000)
            libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd57cb68000)
            /lib64/ld-linux-x86-64.so.2 (0x00007fd57cd84000)
    root@185-46-10-155:/# mkdir -p seminar1/lib/x86_64-linux-gnu
    root@185-46-10-155:/# mkdir seminar1/lib64
    root@185-46-10-155:/# cp lib/x86_64-linux-gnu/libc.so.6 seminar1/lib/x86_64-linux-gnu
    root@185-46-10-155:/# cp lib64/ld-linux-x86-64.so.2 seminar1/lib64
# Скопируем утилиту ls и недостающие библиотеки в наш корневой каталог.
    root@185-46-10-155:/# cp /bin/ls seminar1/bin
    root@185-46-10-155:/# ldd /bin/ls
            linux-vdso.so.1 (0x00007fff15ffc000)
            libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f5d024b3000)
            libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5d022c1000)
            libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007f5d02230000)
            libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f5d0222a000)
            /lib64/ld-linux-x86-64.so.2 (0x00007f5d0250a000)
            libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f5d02207000)
    root@185-46-10-155:/# cp lib/x86_64-linux-gnu/libselinux.so.1 seminar1/lib/x86_64-linux-gnu
    root@185-46-10-155:/# cp lib/x86_64-linux-gnu/libpcre2-8.so.0 seminar1/lib/x86_64-linux-gnu
    root@185-46-10-155:/# cp lib/x86_64-linux-gnu/libdl.so.2 seminar1/lib/x86_64-linux-gnu
    root@185-46-10-155:/# cp lib/x86_64-linux-gnu/libpthread.so.0 seminar1/lib/x86_64-linux-gnu
# Проверим результат и убедимся, что командная оболочка dash запустилась в изолированной файловой системе.
    root@185-46-10-155:/# chroot seminar1 /bin/dash
    # ls
    bin  lib  lib64
    # exit
    root@185-46-10-155:/# ls
    bin   dev  home  lib32  libx32      media  opt   root  sbin      srv  tmp  var
    boot  etc  lib   lib64  lost+found  mnt    proc  run   seminar1  sys  usr
# Создадим новое сетевое пространство имен, запустим dash в его контексте, и проверим результат.
    root@185-46-10-155:/# ip netns add testns
    root@185-46-10-155:/# ip netns exec testns dash
    # ip a
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    # exit
    root@185-46-10-155:/# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 52:54:00:ef:28:45 brd ff:ff:ff:ff:ff:ff
        inet 185.46.10.155/24 brd 185.46.10.255 scope global eth0
        valid_lft forever preferred_lft forever
        inet6 2a00:f940:2:4:2::34bf/64 scope global
        valid_lft forever preferred_lft forever
        inet6 fe80::5054:ff:feef:2845/64 scope link
        valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether 52:54:00:00:cc:7a brd ff:ff:ff:ff:ff:ff
# Запустим оболочку dash в новом пространстве имен PID и проверим результат.
    root@185-46-10-155:/# unshare --fork --pid --mount-proc  /bin/dash
    # ps aux
    USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root           1  0.0  0.0   2608   600 pts/0    S    15:02   0:00 /bin/dash
    root           2  0.0  0.3   9200  3180 pts/0    R+   15:02   0:00 ps aux
    # exit
    /bin/dash: 4: Cannot set tty process group (No such process)
    root@185-46-10-155:/# ps aux
    USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root           1  0.0  1.2 103236 12056 ?        Ss   10:29   0:03 /sbin/init
    root           2  0.0  0.0      0     0 ?        S    10:29   0:00 [kthreadd]
    root           3  0.0  0.0      0     0 ?        I<   10:29   0:00 [rcu_gp]
    root           4  0.0  0.0      0     0 ?        I<   10:29   0:00 [rcu_par_gp]
    root           6  0.0  0.0      0     0 ?        I<   10:29   0:00 [kworker/0:0H-kblockd]
    root           8  0.0  0.0      0     0 ?        I<   10:29   0:00 [mm_percpu_wq]
    root           9  0.0  0.0      0     0 ?        S    10:29   0:00 [ksoftirqd/0]
    root          10  0.0  0.0      0     0 ?        I    10:29   0:00 [rcu_sched]
    root          11  0.0  0.0      0     0 ?        S    10:29   0:00 [migration/0]
    root          12  0.0  0.0      0     0 ?        S    10:29   0:00 [idle_inject/0]
    root          14  0.0  0.0      0     0 ?        S    10:29   0:00 [cpuhp/0]
    root          15  0.0  0.0      0     0 ?        S    10:29   0:00 [kdevtmpfs]
    root          16  0.0  0.0      0     0 ?        I<   10:29   0:00 [netns]
    root          17  0.0  0.0      0     0 ?        S    10:29   0:00 [rcu_tasks_kthre]
    root          18  0.0  0.0      0     0 ?        S    10:29   0:00 [kauditd]
    root          19  0.0  0.0      0     0 ?        S    10:29   0:00 [khungtaskd]
    root          20  0.0  0.0      0     0 ?        S    10:29   0:00 [oom_reaper]
    root          21  0.0  0.0      0     0 ?        I<   10:29   0:00 [writeback]
    root          22  0.0  0.0      0     0 ?        S    10:29   0:00 [kcompactd0]
    root          23  0.0  0.0      0     0 ?        SN   10:29   0:00 [ksmd]
    root          24  0.0  0.0      0     0 ?        SN   10:29   0:00 [khugepaged]
    root          70  0.0  0.0      0     0 ?        I<   10:29   0:00 [kintegrityd]
    root          71  0.0  0.0      0     0 ?        I<   10:29   0:00 [kblockd]
    root          72  0.0  0.0      0     0 ?        I<   10:29   0:00 [blkcg_punt_bio]
    root          73  0.0  0.0      0     0 ?        I<   10:29   0:00 [tpm_dev_wq]
    root          74  0.0  0.0      0     0 ?        I<   10:29   0:00 [ata_sff]
    root          75  0.0  0.0      0     0 ?        I<   10:29   0:00 [md]
    root          76  0.0  0.0      0     0 ?        I<   10:29   0:00 [edac-poller]
    root          77  0.0  0.0      0     0 ?        I<   10:29   0:00 [devfreq_wq]
    root          78  0.0  0.0      0     0 ?        S    10:29   0:00 [watchdogd]
    root          81  0.0  0.0      0     0 ?        S    10:29   0:00 [kswapd0]
    root          82  0.0  0.0      0     0 ?        S    10:29   0:00 [ecryptfs-kthrea]
    root          84  0.0  0.0      0     0 ?        I<   10:29   0:00 [kthrotld]
    root          85  0.0  0.0      0     0 ?        I<   10:29   0:00 [acpi_thermal_pm]
    root          86  0.0  0.0      0     0 ?        S    10:29   0:00 [scsi_eh_0]
    root          87  0.0  0.0      0     0 ?        I<   10:29   0:00 [scsi_tmf_0]
    root          88  0.0  0.0      0     0 ?        S    10:29   0:00 [scsi_eh_1]
    root          89  0.0  0.0      0     0 ?        I<   10:29   0:00 [scsi_tmf_1]
    root          91  0.0  0.0      0     0 ?        I<   10:29   0:00 [vfio-irqfd-clea]
    root          93  0.0  0.0      0     0 ?        I<   10:29   0:00 [ipv6_addrconf]
    root         102  0.0  0.0      0     0 ?        I<   10:29   0:00 [kstrp]
    root         105  0.0  0.0      0     0 ?        I<   10:29   0:00 [kworker/u3:0]
    root         118  0.0  0.0      0     0 ?        I<   10:29   0:00 [charger_manager]
    root         152  0.0  0.0      0     0 ?        S    10:29   0:00 [scsi_eh_2]
    root         153  0.0  0.0      0     0 ?        I<   10:29   0:00 [scsi_tmf_2]
    root         154  0.0  0.0      0     0 ?        I<   10:29   0:00 [kworker/0:1H-kblockd]
    root         174  0.0  0.0      0     0 ?        S    10:29   0:00 [jbd2/sda1-8]
    root         175  0.0  0.0      0     0 ?        I<   10:29   0:00 [ext4-rsv-conver]
    root         219  0.0  1.2  35752 11868 ?        S<s  10:29   0:01 /lib/systemd/systemd-journald
    root         259  0.0  0.5  19384  5088 ?        Ss   10:29   0:00 /lib/systemd/systemd-udevd
    systemd+     260  0.0  0.6  90884  6064 ?        Ssl  10:29   0:00 /lib/systemd/systemd-timesyncd
    root         287  0.0  0.0      0     0 ?        I<   10:29   0:00 [cryptd]
    systemd+     393  0.0  0.7  19076  7312 ?        Ss   10:29   0:00 /lib/systemd/systemd-networkd
    systemd+     404  0.0  1.2  24684 12348 ?        Ss   10:29   0:00 /lib/systemd/systemd-resolved
    root         424  0.0  0.7 235776  7056 ?        Ssl  10:31   0:00 /usr/lib/accountsservice/accounts-daemon
    root         426  0.0  0.3   7128  2988 ?        Ss   10:31   0:00 /usr/sbin/cron -f
    message+     429  0.0  0.4   7404  4508 ?        Ss   10:31   0:00 /usr/bin/dbus-daemon --system --address=syst
    root         439  0.0  1.8  29612 18532 ?        Ss   10:31   0:00 /usr/bin/python3 /usr/bin/networkd-dispatche
    root         440  0.0  0.3  80180  3808 ?        Ssl  10:31   0:00 /usr/sbin/qemu-ga
    syslog       441  0.0  0.4 224316  4832 ?        Ssl  10:31   0:00 /usr/sbin/rsyslogd -n -iNONE
    root         443  0.0  0.7  17364  7744 ?        Ss   10:31   0:00 /lib/systemd/systemd-logind
    root         454  0.0  2.1 396328 21156 ?        Ssl  10:31   0:15 /usr/bin/python3 /usr/bin/fail2ban-server -x
    root         523  0.0  0.1   6140  1808 tty1     Ss+  10:31   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linu
    root         604  0.0  0.7  12176  7316 ?        Ss   10:31   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-1
    root         861  0.0  0.0   2488   508 ?        S    10:57   0:00 bpfilter_umh
    root        2063  0.0  0.8  13900  8804 ?        Ss   12:41   0:00 sshd: root@pts/0
    root        2078  0.0  0.9  18796  9056 ?        Ss   12:42   0:00 /lib/systemd/systemd --user
    root        2079  0.0  0.0      0     0 ?        I    12:42   0:06 [kworker/0:1-events]
    root        2080  0.0  0.4 104452  4044 ?        S    12:42   0:00 (sd-pam)
    root        2102  0.0  0.5   8604  5144 pts/0    Ss   12:42   0:00 -bash
    root        2204  0.0  0.0      0     0 ?        I    12:57   0:00 [kworker/0:2]
    root        2263  0.0  0.0      0     0 ?        I    13:41   0:00 [kworker/u2:1-events_unbound]
    root        2338  0.0  0.0      0     0 ?        I    14:43   0:00 [kworker/u2:2-events_power_efficient]
    root        2509  0.0  0.0      0     0 ?        I    15:02   0:00 [kworker/u2:0-events_power_efficient]
    root        2520  0.0  0.3   9200  3244 pts/0    R+   15:03   0:00 ps aux