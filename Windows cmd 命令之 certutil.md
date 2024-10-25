# CertUtil

## hashfile

- 计算 SHA-1

```shell
//sha1 可以省略
certutil -hashfile test.txt sha1
```

- 计算 SHA-256

```shell
certutil -hashfile test.txt sha256
```

- 计算 SHA-512

```shell
certutil -hashfile test.txt sha512
```

- 计算 MD5

```shell
certutil -hashfile test.txt md5
```

