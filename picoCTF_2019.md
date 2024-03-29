# picoCTF 2019

The picoCTF 2019 competition runs from September 27, 2019 12:00PM EST to October 11, 2019 12:00PM EST.

## Background

PicoCTF is **the first** CTF I have _ever_ participated in! Thanks to my friend and colleague Keanu (hitosis) for creating a CTF group/team at our work, without him I would never have had the inkling to participate!

Since this is my first CTF, I started out rather anxious -- though it turns out some of the challenges were more than approachable enough for me to handle. The CTF itself does a nice job of ramping itself up slowly, as to keep challenging new participants without making them feel like total n00bs.

I'm gonna try to approach this writeup as a service to my past and future self, since most likely no one is ever going to read this, and as a way to document my progress over this week. (Note that I'm writing this introductory background section on a Tuesday, and I started hacking away on a Sunday.) For everyone's sake, I'll skip the _easy_ challenges that don't require much thought.

### whats-the-difference (200 point challenge)

- Started 9/30/2019
- Completed 10/1/2019

This particular challenge was interesting because I had no idea where to start in the first place. The challenge itself had the following "instructions," if you can even call it that...

> Can you spot the difference? kitters cattos. They are also available at /problems/whats-the-difference_0_00862749a2aeb45993f36cc9cf98a47a on the shell server

When navigating to that directory within the interactive shell, a quick `ls -la` displayed two files: `kitters.jpg` and `cattos.jpg`.

_Hmm..._ I thought to myself, _how does one go about opening up 2 `.jpg` files in a terminal?_ Clearly, the terminal wouldn't be able to render a picture file... Having said that, I `cat`ted into the file anyway, which brought up a bunch of unreadable gibberish. This was, very obviously, not helpful.

A quick Google search informed me that `.jpg` files are simply binary, which in retrospect seemed quite obvious. So, I figured I ought to do some kind of `hexdump` wizardry in order to compare the two files.

However, being new at this hacking thing, I probably overthought things way too much and couldn't figure out how to order/redirect my commands. Yet, I'll write out the thought process I tried to follow below.

#### What I knew

- **`hexdump`**: I knew I would need to use `hexdump` to dump the binary into hex, which I could then read and analyze more closely.
   - Looking at the `man` page for `hexdump` made it clear I would probably need to use the `-C` option, to view the hex ***and*** the matching ASCII of what the heck the hex meant.

- **`diff`**: I also knew I would need to use `diff` to compare the two files, and find which parts of the binary/hex/ASCII were different.

#### What I did not know

- What order to use with my commands: Do I dump the hex first *and then* pipe it to `diff` to compare the results?
- Or, do I invoke `diff` first and try to pass it hex as input?
   - Moreover, *how* do I pass the files to `diff` and make sure they are in hex/ASCII at the same time? Doing `diff kitters.jpg cattos.jpg` wouldn't get me anywhere, because I hadn't dumped the hex before passing the files over to `diff`. What a headache...

#### What I tried (but didn't work)

At first, I tried a few things, but none of them worked. As it was already 11:30 PM and I was getting frustrated, I decided to call it a day and head to bed. Here are things that **didn't** work and the accompanying thoughts.

- `hexdump -C kitters.jpg && hexdump -C cattos.jpg | diff`: I tried running `hexdump` on the files and then piping the output to `diff`. What ended up happening was that it dumped the hex, but then gave me this error: `diff: missing operand after 'diff'`.
   - I realized this wouldn't work because I wasn't passing any files for `diff` to compare. But hey, at least my brilliant idea to dump the hex was working! (Gotta look on the bright side, right?)
   
- `diff | hexdump -C kitters.jpg && hexdump -C cattos.jpg`: Since `diff` needs 2 files to compare, I figured I could pipe `hexdump -C file1 file2` into it to use as input. The commands executed, hex(es) started to run down my screen, and I was starting to get excited. *Wow, too easy!* I thought. But once it had finished executing, I realized this didn't actually compare the two files. *Whomp whomp*.
   - ***Haha, what a scrub!*** I'd totally forgotten, or did I just not know or not realize, that piping takes the `stdout` (!) of the first command and sends it to a second command to process. So clearly, there was nothing for `diff` to send to `hexdump -C kitters.jpg && hexdump -C cattos.jpg` because by itself it had no output. Thus, it turns out, I was staring at a whole lot of nothing. (Except the hexdump, which I guess was progress? Maybe?) I still needed a find a way for `diff` to accept `kitters.jpg` and `cattos.jpg` as my input...
   
By this point, I was getting upset and cursing myself out for being dumb and not being able to figure it out. Since it was late, I decided to go to bed and try again later, because at the moment I couldn't figure out what I was doing wrong!

For full disclosure, it was only when compiling this writeup that I actually sat down and figured out why what I had tried didn't work. Yay for hindsight. And for writing this up, which made me think it through! Thank you random Internet strangers who inspired me to write this down for their future selves.

#### What I tried that worked

The next morning, I knew I wanted to get this challenge solved. Asking one of our more experience penetration testers how he would approach it, I was told the easiest way around this challenge would be to redirect the output of `hexdump -C` to a file, which would make `diff`ing the files easier; and so this is what I did.

- `hexdump -C kitters.jpg > kittersfile` and `hexdump -C cattos.jpg > cattosfile`: This allowed me to direct the `hexdump` output to 2 separate files, that would then make it easier to compare using `diff`.

A quick read through of the manpage for `diff` led me to realize the `-c` option might allow me to find the differences between files. Indeed, the output looked like the below:

```
*** kittersfile 2019-10-01 17:53:04.988000000 +0000
--- cattosfile  2019-10-01 17:53:24.956000000 +0000
***************
*** 3106,3112 ****
  0000c210  9c 9a 95 9c 80 a3 c6 e7  1d b1 c6 3b d7 2b 95 d5  |...........;.+..|
  0000c220  98 b9 9e c8 fc 2e f8 69  ab 44 23 6d 32 3d 3d 2c  |.......i.D#m2==,|
  0000c230  8c 23 cb 98 17 c0 46 1d  46 3b 74 af 59 17 e8 6f  |.#....F.F;t.Y..o|
! 0000c240  21 d3 6d a6 4b 99 9d 98  c8 60 1b 95 41 e8 b9 3d  |!.m.K....`..A..=|
  0000c250  1b ad 51 fd a7 be 15 eb  3f 04 3e 21 5b 78 c7 4f  |..Q.....?.>![x.O|
  0000c260  d3 5e 7f 0c 6a 64 33 c0  8c 0b 45 92 78 c7 42 7d  |.^..jd3...E.x.B}|
  0000c270  cd 51 1a 96 bf 7d 60 47  82 f5 1f 0d a2 01 b4 4f  |.Q...}`G.......O|
  --- 3106,3112 ----
  0000c210  9c 9a 95 9c 80 a3 c6 e7  1d b1 c6 3b d7 2b 95 d5  |...........;.+..|
  0000c220  98 b9 9e c8 fc 2e f8 69  ab 44 23 6d 32 3d 3d 2c  |.......i.D#m2==,|
  0000c230  8c 23 cb 98 17 c0 46 1d  46 3b 74 af 59 17 e8 6f  |.#....F.F;t.Y..o|
! 0000c240  21 d3 6d a6 4b 70 69 63  6f 60 1b 95 41 e8 b9 3d  |!.m.Kpico`..A..=|
  0000c250  1b ad 51 fd a7 be 15 eb  3f 04 3e 21 5b 78 c7 4f  |..Q.....?.>![x.O|
  0000c260  d3 5e 7f 0c 6a 64 33 c0  8c 0b 45 92 78 c7 42 7d  |.^..jd3...E.x.B}|
  0000c270  cd 51 1a 96 bf 7d 60 47  82 f5 1f 0d a2 01 b4 4f  |.Q...}`G.......O|
***************
```
  
  Based on the output, I knew that I had something going. The different lines were marked with a `!`, which made identifying them easier. However, having to compare lines that were far removed from each other would make it difficult. So, I realized I would need to filter out the *noise* and focus only on the different lines. To do so, I piped the output and `grep`ed out the different lines with the following command: `grep "^!"`. This allowed me identify the different lines more easily. The output is below:
  
```
! 0000c240  21 d3 6d a6 4b 99 9d 98  c8 60 1b 95 41 e8 b9 3d  |!.m.K....`..A..=|
! 0000c240  21 d3 6d a6 4b 70 69 63  6f 60 1b 95 41 e8 b9 3d  |!.m.Kpico`..A..=|
! 00015660  e5 7c 47 79 71 fd a1 14  5e 64 7e 4b 06 62 0a 90  |.|Gyq...^d~K.b..|
! 00015660  e5 7c 47 79 71 fd a1 14  5e 64 7e 4b 06 62 43 90  |.|Gyq...^d~K.bC.|
! 00027b50  5b 5b 3e d2 d1 42 a0 03  dc f4 c1 f4 ff 00 eb 53  |[[>..B.........S|
! 00027b50  5b 5b 3e d2 d1 42 a0 03  dc 54 c1 f4 ff 00 eb 53  |[[>..B...T.....S|
! 0002ac70  72 df 45 2d a4 76 c2 59  01 01 88 fe f7 1d 6b b1  |r.E-.v.Y......k.|
! 0002ac70  72 df 45 2d a4 76 c2 59  01 01 88 fe f7 1d 46 7b  |r.E-.v.Y......F{|
! 00033c10  27 98 92 0d d8 ec 42 91  c1 f7 cd 1d 06 95 c8 ec  |'.....B.........|
! 00033c10  27 74 68 33 79 ec 42 91  c1 f7 cd 1d 06 95 c8 ec  |'th3y.B.........|
! 00045700  f5 ad 24 96 1f 3a d6 78  80 75 52 a0 38 00 83 d3  |..$..:.x.uR.8...|
! 00045700  f5 ad 24 96 1f 3a d6 78  80 75 72 a0 38 00 83 d3  |..$..:.x.ur.8...|
! 000475f0  d4 ec 6d 7c f9 2c 9e 2b  90 a3 05 5f 9c 1f 5c 7e  |..m|.,.+..._..\~|
! 000475f0  d4 ec 6d 33 5f 2c 9e 2b  90 a3 05 5f 9c 1f 5c 7e  |..m3_,.+..._..\~|
! 00051030  2c a3 8c 76 e7 8e f5 0e  89 27 93 6d 23 00 81 0a  |,..v.....'.m#...|
! 00051030  2c a3 8c 76 e7 61 35 0e  89 27 93 6d 23 00 81 0a  |,..v.a5..'.m#...|
! 00068280  f5 7d 7e 69 22 64 92 e6  13 14 af e6 7c a5 01 6f  |.}~i"d......|..o|
! 00068280  f5 7d 7e 69 22 64 92 5f  13 14 af e6 7c a5 01 6f  |.}~i"d._....|..o|
! 0006b650  d6 65 69 42 8d ce ac 09  6f 53 c5 69 fc 66 f0 e2  |.eiB....oS.i.f..|
! 0006b650  d6 65 69 42 8d ce ac 09  6f 53 c5 69 fc 66 64 e2  |.eiB....oS.i.fd.|
! 0007deb0  03 5d 76 92 d7 57 36 66  19 4a aa a3 12 80 f2 5b  |.]v..W6f.J.....[|
! 0007deb0  03 5d 76 92 d7 57 36 66  19 31 66 a3 12 80 f2 5b  |.]v..W6f.1f....[|
! 0008e7b0  d8 35 04 5a 96 9f 61 e1  4f 19 e9 76 8b 7f a6 cf  |.5.Z..a.O..v....|
! 0008e7b0  d8 35 04 5a 96 9f 61 66  4f 19 e9 76 8b 7f a6 cf  |.5.Z..afO..v....|
! 0009c7e0  58 22 87 c8 55 38 cf 1d  eb 5f 4e 75 71 34 b2 a3  |X"..U8..._Nuq4..|
! 0009c7e0  58 22 87 33 55 38 cf 1d  eb 5f 4e 75 71 34 b2 a3  |X".3U8..._Nuq4..|
! 000a8290  73 a5 68 7e 1f d7 f5 4b  2b 4d 3f 47 b7 bb 6b 9b  |s.h~...K+M?G..k.|
! 000a8290  73 a5 68 7e 1f d7 f5 4b  2b 4d 72 33 b7 bb 6b 9b  |s.h~...K+Mr3..k.|
! 000ab9d0  88 f5 af 2e ba d4 13 c3  be 27 f1 37 c3 af 13 e9  |.........'.7....|
! 000ab9d0  88 f5 af 2e ba d4 13 c3  be 27 f1 37 c3 af 6e e9  |.........'.7..n.|
! 000b7730  1f f7 c9 63 1f 96 84 a1  7c 00 ea 00 fc c0 ae a3  |...c....|.......|
! 000b7730  1f f7 c9 63 1f 96 84 a1  7c 00 ea 00 fc c0 ae 74  |...c....|......t|
! 000b8420  7a 46 8b aa ce f0 41 77  7d 1c 31 4b bc c4 db 18  |zF....Aw}.1K....|
! 000b8420  7a 46 8b aa ce f0 41 77  7d 1c 5f 34 73 5f db 18  |zF....Aw}._4s_..|
! 000c2640  5a 23 da c5 6f 7d 23 1b  99 5b 20 23 64 38 f7 f7  |Z#..o}#..[ #d8..|
! 000c2640  5a 62 da c5 6f 7d 23 1b  99 5b 20 23 64 38 f7 f7  |Zb..o}#..[ #d8..|
! 000d4af0  1b 3c 60 e7 1d 78 ae 06  ce e6 34 8a 3b 54 8f 6a  |.<`..x....4.;T.j|
! 000d4af0  1b 3c 60 e7 1d 78 75 37  37 33 34 8a 3b 54 8f 6a  |.<`..xu7734.;T.j|
! 000e2710  d4 e7 8a 52 57 dc c5 bb  bb 9e db 64 6d 34 dd 36  |...RW......dm4.6|
! 000e2710  d4 72 5f 34 57 dc c5 bb  bb 9e db 64 6d 34 dd 36  |.r_4W......dm4.6|
! 000f2d60  3d c4 2a 60 4b c1 88 e6  c8 fe 23 ff 00 d7 ad a3  |=.*`K.....#.....|
! 000f2d60  3d c4 2a 60 4b c1 88 e6  c8 6e 64 5f 00 d7 ad a3  |=.*`K....nd_....|
! 00104e20  9c e4 83 92 72 0f e2 6b  da 74 1f ec 1b eb 06 83  |....r..k.t......|
```
  
  Then, the rest was child's play and I was able to compare the different lines and pick out the different characters and concatenate them to create the flag, which ended up being `picoCTF{th3yr3_a5_d1ff3r3nt_4s_bu773r_4nd_j311y_aslkjfdsalkfslkflkjdsfdszmz10548}`.
  
  #### What I learned  after the fact...
  
So even though I'd already captured this particular flag, I wasn't satisfied with the number of steps it took to resolve the challenge. My colleagues Scott and Brett always joke that most things can be achieved in one line of Bash, so I wanted to see if I could do just that (or at least get close to it).
 
The first challenge was figuring out how to direct commands as arguments. By this point, I knew that I needed to `diff` the two files, but how would I pass `hexdump` as an argument in `diff`? Some investigation led me to some articles on Stackoverflow, which showed this achieved via a `command <(command as argument)`. Since diff takes two arguments, some tinkering allowed me to conclude that I could do `command <(command as argument) <(command as argument)`.

Thus, I could do the following: `diff -c <(hexdump -C kitters.jpg) <(hexdump -C cattos.jpg)`.

This would allow me to dump the hex and run `diff` all in one command. The rest was child's play insofar as all I had to do was pipe the command to `grep "^!"` to filter out the noise and be left with the useful stuff.

So far, this has proved the most difficult challenge for me to complete, and by far the one that took me the longest to think through!
