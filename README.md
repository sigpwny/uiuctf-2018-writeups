# sigpwnyctf2018-writeups
TODO: markdown relative links to everything
If nobody got the solve, no need to writeup, we can re-use next time.
* openssl: Pipe the content of the file into `openssl` with the appropriate
flags to decrypt. `cat ciphertext | openssl aes-256-cbc -d` and the
password is `password`
* "Cryptanalysis" TODO Evan
* [forensics fun](forensics.md)
* [bof (simple writeup)][1] [bof (cool writeup)][2]
* what is base (2^6): base64 encoding. very common in ctf. `echo
 Q1RGe2Jhc2VkX29mX3RoZV82NHRofQ== | base64 --decode`
 * [mondai.zip][3] (vietnamese, google translate it)
 * what's ian's password? - send me an email asking for my password
 * [who is the queen of cyber?][4]


[1]: https://github.com/smholsen/pwnable.kr/tree/master/3-bof
[2]: https://github.com/USCGA/writeups/tree/master/pwnable.kr/bof
[3]: https://github.com/TryCTFAgain/CTF-Writeups/tree/master/2018/Tokyo%20Western%20CTF/%5BMisc%5D%20mondai.zip
[4]: https://twitter.com/SwiftOnSecurity/status/858092845886046209
