# go struct method

在golang 定義屬於structure method時

我們有兩種回傳值 

1 struct本身

2 struct pointer

一般來說

比較習慣 pointer來做 處理 因為

如果要存取 member func 的結果

可以透過 reference方式

而不需要複製一份新的 struct

然而在使用go routine時 由於 可能不斷Context Switch

就要考慮 是否 global的 reference 因為race condition 造成 非預期的結果

例如用 go routine 去執行 產生email的程式

```golang===
type email struct {
    from string
    to string
}

func (e *email) From(s string) {
    e.from = s
}

func (e *email) To(s string) {
    e.to = s
}

func (e *email) Send() {
    fmt.Printf("From: %s, To: %s\n", e.from, e.to)
}

func main() {
    for i:=0; i < 10; i++ {
        go func(int i) {
            e := &email{}
            e.From(fmt.Sprintf("example%02d@email.com", i))
            e.To(fmt.Sprintf("example%02d@email.com", i+1))
            e.Send()
        }(i)
    }
}
```

在上面範例中 為了避免 在context switch時 reference到 相同的物件造成 結果不一致 

所以每個 func 都各自new 了一個屬於自己的struct

但是也可透過 return value的方式改成以下寫法

```golang===
type email struct {
    from string
    to string
}

func (e email) From(s string) email {
    e.from = s
    return e
}

func (e email) To(s string)  email{
    e.to = s
    return e
}

func (e email) Send() {
    fmt.Printf("From: %s, To: %s\n", e.from, e.to)
}
e := &email{}
func main() {
    for i:=0; i < 10; i++ {
        go func(int i) {
            
            e.From(fmt.Sprintf("example%02d@email.com", i)).
               To(fmt.Sprintf("example%02d@email.com", i+1)).
               Send()
        }(i)
    }
}
```