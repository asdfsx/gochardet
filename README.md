# chardet

chardet is library to automatically detect
[charset](http://en.wikipedia.org/wiki/Character_encoding) of texts for [Go
programming language](http://golang.org/). It's based on the algorithm and data
in [ICU](http://icu-project.org/)'s implementation.

## Documentation and Usage

See [pkgdoc](http://go.pkgdoc.org/github.com/saintfish/chardet)

##　关于多字节字符号判断的说明

简单看了下shift-jis的处理逻辑。内部有几个限制

* 最后一个字符可能不会读取。每次调用 DecodeOneChar 都会读取两个字节，即一个双字节字符。循环中如果第一次调用就读完了所有数据，那么就会直接退出循环，导致最后一个读到的数据不会计入到 totalCharCount 等变量中。也就是说10个日文字符，可能只有前9个会参与判断。  
    
    ```
    for c, raw, err = r.decoder.DecodeOneChar(raw); len(raw) > 0; c, raw, err = r.decoder.DecodeOneChar(raw)
    ```

* totalCharCount一定要大于 10。但是结合上一条。起码要有 11 个字符才能满足这个条件  

    ```
    if doubleByteCharCount <= 10 && badCharCount == 0 
		if doubleByteCharCount == 0 && totalCharCount < 10 
    ```


* 双字节字符的总数要大于等于4。如果双字节字符不够，在下边的计算中 maxVal=0 ，然后之后的 scaleFactor、confidence 都会收到影响。  

    ```
    maxVal := math.Log(float64(doubleByteCharCount) / 4)
    scaleFactor := 90 / maxVal
    confidence := int(math.Log(float64(commonCharCount)+1)*scaleFactor + 10)
    ```
