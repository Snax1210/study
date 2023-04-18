# Golang中JSON tag标签的作用和用法讲解

tag 是结构体的原信息，运行时通过反射机制读取。结构体的tag一般定义在相应字段的后面，
格式为：

```go
    Field1 string `tag1:"value1" tag2:"value2"`
```

同一个结构体字段可以设置多个键值对tag，不同的键值对之间使用空格分隔。

## JSON tag

默认情况下序列化与反序列化使用的都是结构体原生的字段名，可以通过给结构体字段添加json tag来指
定序列化后的字段名。标签冒号前是类型，后面是标签名。

```go
// Product _
type Product struct {
    Name      string  `json:"name"`
    ProductID int64   `json:"-"` // 表示不进行序列化
    Number    int     `json:"number"`
    Price     float64 `json:"price"`
    IsOnSale  bool    `json:"is_on_sale,string"`
}

// 序列化过后，可以看见
{"name":"Xiao mi 6", "number":10000, "price":2499, "is_on_sale":"false"}

```

omitempty，tag里面加上omitempty，可以在序列化的时候忽略0值或者空值。注意此时在“omitempty”
前一定指定一个字段名，否则“omitempty”将作为字段名处理。

**注意：\`json:"product_id,omitempty"` ，omitempty前面和逗号之间不能有空格！**

```go

package main

import (
	"encoding/json"
	"fmt"
)

// Product _
type Product struct {
	Name      string  `json:"name"`
	ProductID int64   `json:"product_id,omitempty"`
	Number    int     `json:"number"`
	Price     float64 `json:"price"`
	IsOnSale  bool    `json:"is_on_sale,omitempty"`
}

func main() {
	p := &Product{}
	p.Name = "Xiao mi 6"
	p.IsOnSale = false
	p.Number = 10000
	p.Price = 2499.00
	p.ProductID = 0

	data, _ := json.Marshal(p)
	fmt.Println(string(data))
}
// 结果（省略掉了p.IsOnSale 和 p.ProductID）
{"name":"Xiao mi 6","number":10000,"price":2499}

```

若要在被嵌套结构体整体为空时使其在序列化结果中被忽略，不仅要在被嵌套结构体字段后加上
json:"fileName,omitempty"，还要将其改为结构体指针。如：

```go
package main

import (
    "encoding/json"
    "fmt"
)

type BodyInfo struct {
    Weight float64
    Height float64
}

type Student struct {
    Name      string `json:"name"`
    Age       int64
    *BodyInfo `json:"bodyinfo,omitempty"`
}

func main() {
    s1 := Student{
        Name: "jack",
        Age:  20,
    }

    data, _ := json.Marshal(s1)
    fmt.Println(string(data))
}

//结果
{"name":"jack","Age":20}
```

\`json:",inline"`通常作用于内嵌的结构体类型，具体用法看下面这个例子：


type,有些时候，我们在序列化或者反序列化的时候，可能结构体类型和需要的类型不一致，这个时候可以
指定，支持string，number和boolean

```go
package main

import (
    "encoding/json"
    "fmt"
)

// Product _
type Product struct {
    Name      string  `json:"name"`
    ProductID int64   `json:"product_id,string"`
    Number    int     `json:"number,string"`
    Price     float64 `json:"price,string"`
    IsOnSale  bool    `json:"is_on_sale,string"`
}

func main() {

    var data = `{"name":"Xiao mi 6","product_id":"10","number":"10000","price":"2499","is_on_sale":"true"}`
    p := &Product{}
    err := json.Unmarshal([]byte(data), p)
    fmt.Println(err)
    fmt.Println(*p)
}
// 结果
<nil>
{Xiao mi 6 10 10000 2499 true}

```