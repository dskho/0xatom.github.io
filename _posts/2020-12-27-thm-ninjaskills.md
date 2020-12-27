---
title: TryHackMe - Ninja Skills
description: My writeup on Ninja Skills box.
categories:
 - tryhackme
tags: tryhackme linux find
---

![](https://i.imgur.com/Dlpg3Cz.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Ninja Skills**  |
| Difficulty :  | **Easy**             |   
| Play :    | [Ninja Skills](https://tryhackme.com/room/ninjaskills){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Hello all, this box requires linux skill nothing advanced mostly you need to know how `find` command works. It gives us 12 files and we have to answer 6 questions about them. Let's start!

What we'll do first is to run `find` on all the files to detect them & then we will use `-exec` option to answer the questions. You will notice that the files are 11 because `bny0` is missing i don't know why.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec ls -la "{}" \; 2>/dev/null
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /mnt/D8B3
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /mnt/c4ZX
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/FHl1
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/log/uqyw
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/PFbD
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/oiMO
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /media/rmfX
-rwxrwxr-x 1 new-user new-user 13545 Oct 23  2019 /etc/8V2L
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /etc/ssh/SRSq
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /home/v2Vb
-rw-rw-r-- 1 newer-user new-user 13545 Oct 23  2019 /X1Uy
```

## Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order)

We already have the answer from above ^

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec ls -la "{}" \; 2>/dev/null
...data...
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /mnt/D8B3
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /home/v2Vb
```

Answer: `D8B3` - `v2Vb`

## Which of these files contain an IP address?

Here i found a cool regex that search for an IP.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec grep -H -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" "{}" \; 2>/dev/null
/opt/oiMO:1.1.1.1
```

Answer: `oiMO`

## Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94

We will use the `sha1sum` and `grep`.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec sha1sum "{}" \; 2>/dev/null | grep -H "9d54da7584015647ba052173b84d45e8007eba94"
(standard input):9d54da7584015647ba052173b84d45e8007eba94  /mnt/c4ZX
```

Answer: `c4ZX`

## Which file contains 230 lines?

All files contain 209 lines, so 1 is missing is the `bny0`.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec wc -l "{}" \; 2>/dev/null
209 /mnt/D8B3
209 /mnt/c4ZX
209 /var/FHl1
209 /var/log/uqyw
209 /opt/PFbD
209 /opt/oiMO
209 /media/rmfX
209 /etc/8V2L
209 /etc/ssh/SRSq
209 /home/v2Vb
209 /X1Uy
```

Answer: `bny0`

## Which file's owner has an ID of 502?

Only 1 file has different file owner.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec ls -la "{}" \; 2>/dev/null
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /mnt/D8B3
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /mnt/c4ZX
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/FHl1
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/log/uqyw
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/PFbD
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/oiMO
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /media/rmfX
-rwxrwxr-x 1 new-user new-user 13545 Oct 23  2019 /etc/8V2L
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /etc/ssh/SRSq
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /home/v2Vb
-rw-rw-r-- 1 newer-user new-user 13545 Oct 23  2019 /X1Uy
```

```
[new-user@ip-10-10-16-64 ~]$ id -u newer-user
502
```

Answer: `X1Uy`

## Which file is executable by everyone?

Just let's check the output.

```
[new-user@ip-10-10-16-64 ~]$ find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec ls -la "{}" \; 2>/dev/null
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /mnt/D8B3
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /mnt/c4ZX
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/FHl1
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /var/log/uqyw
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/PFbD
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /opt/oiMO
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /media/rmfX
-rwxrwxr-x 1 new-user new-user 13545 Oct 23  2019 /etc/8V2L
-rw-rw-r-- 1 new-user new-user 13545 Oct 23  2019 /etc/ssh/SRSq
-rw-rw-r-- 1 new-user best-group 13545 Oct 23  2019 /home/v2Vb
-rw-rw-r-- 1 newer-user new-user 13545 Oct 23  2019 /X1Uy
```

Answer: `8V2L`

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
