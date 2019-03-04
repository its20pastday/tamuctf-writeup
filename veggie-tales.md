# Description

It's my favorite show to watch while practing my python skills! I've seen episode 5 at least 13 times.

```
nc pwn.tamuctf.com 8448
```

Difficulty: easy-medium

# Solution

Connecting to the server through nc, we are given a list of options

![](/assets/nc.png)

Adding an episode gives you a list of episodes that you can choose from to add to your list to which you select the number and it gets added to your watch list. Printing your watch list does as you expect printing what is currently on your list. Backup your watch list returns a base64 encoded string that can be used in the fourth option of loading your watch list to return your watch list to the state that it was in when you made the backup.

To start off, I examined the backup string by decoding the base64 however doing so returned garbage. This led me to believe that there must be one additional form of encoding on top of the base64 so that I would be able to get a cleartext string of some sorts.

Looking at the challenge description again, it mentions episode 5 so looking at the title of the fifth option of the episode list we see that it is Dave and the Giant Pickle. Given that this channel was likely coded in Python, this led me to the idea that it was using pickles to load and restore the watch list. Pickle is a python module that allows you to allow object permanence between python sessions, as you can save an object and its state to a file and then load it back up again to restore the state of that object. This means that the load the backup string, loads whatever Python object is given to it, which allows us to gain remote code execution, so we could write python code that should give us a shell and if we could give it to the load function then we would be good.

The only problem was determining what the additional encoding was in addition to the base64. Checking the description again, we see the word 13 appear. This made me look once again at the episode list but there was nothing useful with episode 13. After messing around a little, I saw that when comparing backup strings that contained the same episodes, you could locate where each episode's string lied and you could then put the same episode multiple times in the list however this was not useful to us. Moving past that, the only thing that I considered having to do with the number 13 was ROT13. Performing ROT13 on the backup string of the base64 encoded string, then decoding that gave me the cleartext version of the episode list. Now we know the "encryption process", the server first base64 encodes the pickle of the episode list, then performs ROT13 on that and hands us the string.

Looking online for pickle exploits online, I came across this code:

```
class Exploit(object):
    def __reduce__(self):
        return (os.system, ('/bin/bash',))
```

So all we need to do is run:

```
pickle.dumps(Exploit())
```

and then write that to a file. We then just repeat the encryption process of encoding the pickle with base64 then performing ROT13 on that. 

```
cat exploit | base64 | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

This gives us the string tNAwpT9mnKtXp3ymqTIgPaRNJNxNNNNiLzyhY2Wup2ukNLIkNyWkNl4=. Heading back into nc, we load the watch list give them the string, and voila we have a shell. Running ls shows that they have flag.txt, so running cat flag.txt we get the flag:

```
gigem{d0nt_7rust_th3_g1ant_pick1e}
```



