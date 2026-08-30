# File descriptor
## Everything is a file
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/ab3377ee-4756-4908-8bbb-82abcbc4562d" />

there are a lot of feature on the computer system , but in a nutshell , what the computer mainly do is input something , do some magic and output something.
this is what **Everythings is a file** comes in for .

by abstracting all sorts of operation into the **single** things a.k.a. file , the developer can only use one thought to interact with various resource such as 
- file
- device (e.g keyboard...)
- pipe (from other program)
- network (from other computer via network )
- special sequence (from the program that generate some sequence , such as `dev/zero` `dev/random`
- `stdin`
- `stdout` and `stderr`
