---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: 'text-center'
highlighter: shiki
---
# Git 101

---
layout: center
class: 'text-white'
---

## whoami ? 
<style type="text/css">
    img {
        width: 250px;
    }
</style>
![Me](index.jpeg)

1. Java/Linux/DB/whatever business needs dev
2. Open source enthusiast
3. <logos-Blogger/> **Blog** - https://strogiyotec.github.io/
4. <logos-git/> **Github** - https://github.com/strogiyotec
4. <logos-telegram/> **Telegram** - @Lollipopster

---
layout: center
class: 'text-white'
---
# What will be covered today?
1. Git filesystem
2. What are git Objects
    1. blob
    2. index
    3. tree
    4. commit
    5. branch
3. What is merge and rebase
---
layout: center
class: 'text-white'
---
# What won't be covered ? 
1. Git flow
2. Branching strategy

## Why ? 
**Because it's up to the company you will work on**

---
layout: center
class: 'text-white'
---
![Soydev](soydev.jpg)

---
layout: center
class: 'text-white'
---

## Some history
![Torval](torvalds.jpg)
Mailing lists do not work for kernel
---
layout: center
class: 'text-white'
---
# It all starts with blobs
```
git hash-object -w Main.java
```
`7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037`

What the hell is it ? 

---
layout: center
class: 'text-white'
---

<style type="text/css">
    img {
        width: 400px;
    }
</style>
![blobs](object.png)

---
layout: center
class: 'text-white'
---
1. `7f` - directory
2. `4e1ed98d0c0d3f8a80271b1e61cb91eeefb037` - filename

(but Why ? )

---
layout: center
class: 'text-white'
---
## What is inside ? Ugly ha ? 
`vim 7f/4e1ed98d0c0d3f8a80271b1e61cb91eeefb037`
<style type="text/css">
    img {
        width: 400px;
    }
</style>
![inside](inside.png)

---
layout: center
class: 'text-white'
---
## Let's parse it in Python( Why not Java? 🤔 )
```python
import zlib  # A compression / decompression library

filename = '.git/objects/7f/4e1ed98d0c0d3f8a80271b1e61cb91eeefb037'

compressed_contents = open(filename, 'rb').read()

decompressed_contents = zlib.decompress(compressed_contents)
print(decompressed_contents)

```


---
layout: center
class: 'text-white'
---
# Let's write encoder now
```go
    data = read("Main.java")
	header := header(data, objType)// header is "object_type(blob) content_length null"
	result := append(header, data...)//append data to header
	hash, err := GenerateHash(result)//generate SHA1 hash - 7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037
    compressed :=zlib.Compress(result)
    write_to_file(
        hash,//file name
        compressed //content
    )

```
---
layout: center
class: 'text-white'
---
# Can I avoid writing my own Parse ? Hhhh... You are boring
`git cat-file -p 7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037`

---
layout: center
class: 'text-white'
---
Each time you call `git add` it creates a new file in `objects` folder
![Blobs](blob_store.jpg)
> ! Important , blob files DO NOT contain information about original file name. Who does then ? 

---
layout: center
class: 'text-white'
---
## Meet index file(staging area)
But first let's stage some files
```
git update-index --add --cacheinfo 100644 \
7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037 Main.java
```
1. 100644 - means regular text file
2. 7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037 - Hash
3. Main.java file name

Now try `git status`
```
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   Main.java

```
---
layout: center
class: 'text-white'
---
![Index](index.png)

---
layout: center
class: 'text-white'
---
## What is inside of the index file ? 
```
> git ls-files --stage
100644 7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037 0       Main.java
```
1. 100644 - file type (text file)
2. 7f4e1ed98d0c0d3f8a80271b1e61cb91eeefb037 - SHA-1 has
3. 0 - stage numbers(usefull for merge conflicts)
4. Main.java - file name

---
layout: center
class: 'text-white'
---
# Big picture
1. All your files are compressed and stored in **objects** folder
2. Index file contains name of the file and corresponding hash

But what If I change `Main.java` and add it again to index file ? It will override previous entry

---
layout: center
class: 'text-white'
---
## Meet tree objects(snapshot of index file)
![Tree](Tree.png)
`git write-tree` will create a new tree object<br>
You can verify it using `git-cat` mentioned above

---
layout: center
class: 'text-white'
---
![Why](5pv7gt.jpg)

---
layout: center
class: 'text-white'
---
# Meet a commit object
Motivation. We have a list of blobs saved in tree object. But there is no information such as
1. Who is an author ? 
2. When tree object was created ?
3. Why it was created ?

---
layout: center
class: 'text-white'
---
## Let's create a commit object(as always, it's stored inside objects folder )
` git commit-tree -m 'I created Main.java' c49ca09403970a3fee23aefefd229fdf07fd58b9`

Let's see the content
```
> git cat-file -p bdcbd425c75c1b96da83e7a30f6188d4ecdb2bac
tree c49ca09403970a3fee23aefefd229fdf07fd58b9
author strogiyotec <almas337519@gmail.com> 1633823156 -0700 
committer strogiyotec <almas337519@gmail.com> 1633823156 -0700

I created Main.java

```
1. Tree hash
2. author - with time
3. committer with time
4. Message
---
layout: center
class: 'text-white'
---
## Why do we need author and committer ? 
![Torvald](torvalds.jpg)
> You can be an author but I am the one who commit your changes to main repository


---
layout: center
class: 'text-white'
---
## Hey Almas can you checkout my commit<br> with a hash `asdsadasdwssdfsd`

#### asds... what ? 

TODO: branches
