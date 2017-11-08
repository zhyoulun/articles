


## 

```
$ php -r 'printf("%u\n", crc32("The quick brown fox jumped over the lazy dog."));'
2191738434
```

```
mysql> select crc32("The quick brown fox jumped over the lazy dog.") \G;
*************************** 1. row ***************************
crc32("The quick brown fox jumped over the lazy dog."): 2191738434
1 row in set (0.00 sec)
```

```
>>> import binascii
>>> print binascii.crc32("The quick brown fox jumped over the lazy dog.") % (1<<32)
```

```
package main

import (
	"fmt"
	"hash/crc32"
)

func main() {
	data := []byte("The quick brown fox jumped over the lazy dog.")
	fmt.Println(crc32.ChecksumIEEE(data))
}

输出：2191738434
```

## 参考

- [PHP crc32函数](http://php.net/manual/zh/function.crc32.php)
- [MySQL crc32函数](https://dev.mysql.com/doc/refman/5.7/en/mathematical-functions.html#function_crc32)
- [Go crc32](https://golang.org/pkg/hash/crc32/)
- [How to calculate CRC32 with Python to match online results?](https://stackoverflow.com/questions/30092226/how-to-calculate-crc32-with-python-to-match-online-results)
- [Cyclic Redundancy Code algorithm](https://www.w3.org/TR/2003/REC-PNG-20031110/#5CRC-algorithm)
- [CRCAppendix](https://www.w3.org/TR/2003/REC-PNG-20031110/#D-CRCAppendix)