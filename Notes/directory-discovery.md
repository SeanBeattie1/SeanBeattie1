# Directory Discovery

### Brute Force directories and files

Start **brute-forcing** from the root folder and be sure to brute-force **all** the **directories found** using **this method** and all the directories **discovered** by the **Spidering** (you can do this brute-forcing **recursively** and appending at the beginning of the used wordlist the names of the found directories).\
Tools:

* **Dirb** / **Dirbuster** - Included in Kali, **old** (and **slow**) but functional. Allow auto-signed certificates and recursive search. Too slow compared with th other options.
* [**Dirsearch**](https://github.com/maurosoria/dirsearch) (python)**: It doesn't allow auto-signed certificates but** allows recursive search.
* [**Gobuster**](https://github.com/OJ/gobuster) (go): It allows auto-signed certificates, it **doesn't** have **recursive** search.
* [**Feroxbuster**](https://github.com/epi052/feroxbuster) **- Fast, supports recursive search.**
* [**wfuzz**](https://github.com/xmendez/wfuzz) `wfuzz -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt https://domain.com/api/FUZZ`
* [**ffuf** ](https://github.com/ffuf/ffuf)- Fast: `ffuf -c -w /usr/share/wordlists/dirb/big.txt -u http://10.10.10.10/FUZZ`
* [**uro**](https://github.com/s0md3v/uro) (python): This isn't a spider but a tool that given the list of found URLs will to delete "duplicated" URLs.
* [**Scavenger**](https://github.com/0xDexter0us/Scavenger): Burp Extension to create a list of directories from the burp history of different pages
* [**TrashCompactor**](https://github.com/michael1026/trashcompactor): Remove URLs with duplicated functionalities (based on js imports)
* [**Chamaleon**](https://github.com/iustin24/chameleon): It uses wapalyzer to detect used technologies and select the wordlists to use.

**Recommended dictionaries:**

* [https://github.com/carlospolop/Auto\_Wordlists/blob/main/wordlists/bf\_directories.txt](https://github.com/carlospolop/Auto\_Wordlists/blob/main/wordlists/bf\_directories.txt)
* [**Dirsearch** included dictionary](https://github.com/maurosoria/dirsearch/blob/master/db/dicc.txt)
* [http://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10](http://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10)
* [Assetnote wordlists](https://wordlists.assetnote.io)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)
  * raft-large-directories-lowercase.txt
  * directory-list-2.3-medium.txt
  * RobotsDisallowed/top10000.txt
* [https://github.com/random-robbie/bruteforce-lists](https://github.com/random-robbie/bruteforce-lists)
* [https://github.com/google/fuzzing/tree/master/dictionaries](https://github.com/google/fuzzing/tree/master/dictionaries)
* [https://github.com/six2dez/OneListForAll](https://github.com/six2dez/OneListForAll)
* [https://github.com/random-robbie/bruteforce-lists](https://github.com/random-robbie/bruteforce-lists)
* _/usr/share/wordlists/dirb/common.txt_
* _/usr/share/wordlists/dirb/big.txt_
* _/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt_

_Note that anytime a new directory is discovered during brute-forcing or spidering, it should be Brute-Forced._&#x20;
