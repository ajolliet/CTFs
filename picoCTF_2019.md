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
