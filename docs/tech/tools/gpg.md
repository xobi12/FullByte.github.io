# GPG

## Info

|What|Where|
|-|-|
|Official Page||
|Source||
|Download||
|Install||

## Using a symmetric cypher (AES 256)

Encrypt with symmetric cypher

```shell
gpg --symmetric --cipher-algo aes256 -o test.txt.gpg test.txt
```

Decrypt with symmetric cypher

```shell
gpg -d -o test.txt test.txt.gpg
```
