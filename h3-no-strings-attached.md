# H3 No Strings Attached

## a) Strings

- Downloaded ezbin-challenges.zip from Tero's website (https://terokarvinen.com/application-hacking/#h3-no-strings-attached-tero)
- `wget https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip`
- Let's unzip the downloaded challenges so we can fully access it's files through home directory `unzip ezbin-challenges`
- Move to the files `cd challenges` and then into `passtr`
- We have to find the hidden password. Let's try to use `strings`
My debian 13 didn't have strings, so I looked it up on web and found `sudo apt-get install binutils`
- Let's try it now `strings passtr`
<img width="722" height="409" alt="image" src="https://github.com/user-attachments/assets/f3fe1856-c013-48bb-9179-af41f40a6ef5" />
- We can see it's `Sala-hakkeri-321` plus we see the flag
<img width="627" height="76" alt="image" src="https://github.com/user-attachments/assets/7abb7f4f-1a9b-4c4e-a052-6548dd007a02" />

## b) Hiding the password from the binary

- I have to tweak the program in C language to hide the password so it won't be detected using the `strings` (Have no idea how so I gotta research)
- Asked ChatGPT for help and it suggested me to make python file which prints me non printable characters
<img width="838" height="569" alt="image" src="https://github.com/user-attachments/assets/bda9e66d-3d66-457d-a0d9-415bff84d8d1" />
- Compile it by `gcc passtr.c -o passtr.5`
- Let's try `strings passtr.5`
<img width="655" height="529" alt="image" src="https://github.com/user-attachments/assets/0cc9d902-9cb5-46af-9d3f-8607a4bb42df" />
- We can see now the password doesn't show up or it's actually reconstructed as in XOR

## c) Unpacking files can reveal secrets :O

<img width="282" height="188" alt="image" src="https://github.com/user-attachments/assets/6437fd71-dc7b-4fda-a585-26fd05c2f3b8" />


<img width="655" height="111" alt="image" src="https://github.com/user-attachments/assets/04d1f707-c248-4c70-bfe8-36fd7e3916df" />

<img width="836" height="186" alt="image" src="https://github.com/user-attachments/assets/d0f3c34d-024a-402a-8af6-5ada2762fb51" />
- The flag and password shows up now with `cat packd`





