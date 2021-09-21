使用go代码构建原生的http的post请求：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	request, err := http.NewRequest("GET", "http://www.baidu.com", nil)
	if err != nil {
		log.Fatal("request err： %s", err.Error())
	}
	client := http.Client{}
	do, err := client.Do(request)
	if err != nil {
		log.Fatal("http do err： %s", err.Error())
	}
	body, err := ioutil.ReadAll(do.Body)
	fmt.Println(string(body))
	defer func() {
		err = do.Body.Close()
		if err != nil{
			log.Println("close request err:", err.Error())
		}
	}() 
}

```

