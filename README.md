#Go JSON Marshalling

[**Golang JSON의 거의 모든 것**](https://www.joinc.co.kr/w/man/12/golang/json)를 보고 정리하였습니다.

- `Marshal`: 자료형을 JSON 텍스트로 변환합니다.
- `Marshalling`: 메모리에 저장된 객체를 다른 시스템어 전송하기에 적합한 데이터 형식으로 변환하는 과정입니다.
- `MarshalIndent`: 자료형을 JSON 텍스트로 변환하고 사람이 편하도록 들여쓰기를 해 줍니다.
- `Unmarshal`: JSON 텍스트를 자료형으로 변환합니다.

## Go Struct를 통한 JSON으로 마샬링
사용자 정보를 인터넷을 통해서 제공하는 Go 애플리케이션을 만들려고 가정을 해 보겠습니다.  
이 애플리케이션의 사양은 아래와 같습니다.
1. HTTP(S) 1.1로 제공합니다.
2. REST API로 제공합니다.
3. 웹 브라우저와 안드로이드 앱 모두에서 데이터를 사용할 수 있도록 하기 위해서 JSON을 포맷으로 사용하기로 했습니다.

```go
type UserInfo struct {
UserID   int
Email    string
Age      int
Blog     string
FaceBook string
}
```
사용자 정보는 위와 같은 스트럭처로 정의하였습니다.

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UserInfo struct {
    UserID   int
    Email    string
    Age      int
    Blog     string
    FaceBook string
}

func main() {
    u := UserInfo{
        UserID:   1001,
        Email:    "foo@example.com",
        Age:      32,
        Blog:     "foo.blog.com",
        FaceBook: "foo.facebook.com",
    }
    byteData, _ := json.Marshal(u)
    fmt.Println(string(byteData))
}
```
애플리케이션 요구사항에 따라서 이 스트럭처를 JSON으로 마샬링 하기로 하였습니다.

```shell
$ go run main.go | jsonpp
{
  "UserID": 1001,
  "Email": "foo@example.com",
  "Age": 32,
  "Blog": "foo.blog.com",
  "FaceBook": "foo.facebook.com"
}
```
위 코드를 작성된 프로그램을 실행 시켜 보겠습니다. [jsonpp](https://jmhodges.github.io/jsonpp/)을 통해서 손 쉽게 볼 수 있습니다.