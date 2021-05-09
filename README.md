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
Golang의 encoding 패키지는 데이터를 바이트 수준 및 텍스트 수준에서 변환하는 다양한 서브 패키지들을 제공합니다.  
여기에는 JSON, XML, Base64 등등 다양한 포맷들을 사용할 수 있습니다. JSON의 마샬/언마샬을 위해서 `encoding/json` 패키지를 import 하시면 됩니다.

```go
import "encoding/json"
```

JSON 패키지의 [Marsal 함수](https://golang.org/pkg/encoding/json/#Marshal)를 이용해서, 입력 데이터를 JSON 형태로 마샬링 할 수 있습니다.
```go
func Marshal(v interface{}) ([]byte, error)
```

입력 값은 interface이므로 primitve, array, slice, struct 모두 사용할 수 있습니다.  
마샬링에 성공하면 `[]byte`를 리턴합니다.
```go
u := UserInfo{
    UserID:   1001,
    Email:    "foo@example.com",
    Age:      32,
    Blog:     "foo.blog.com",
    FaceBook: "foo.facebook.com",
}
byteData, _ := json.Marshal(u)
fmt.Println(string(byteData))
```

## JSON Unmarshaling
본 애플리케이션은 유저정보를 저장하기 위한 기능을 제공합니다. 유저 정보는 JSON 형태로 전달됩니다.  
애플리케이션은 JSON 데이터를 언마샬링 해서, 사용 할 수 있는 객체로 만들어야 합니다.
```go
data := `{"UserID":1001, "Email":"foo@example.com",
"Age":32,"Blog":"foo.blog.com",
"FaceBook":"foo.facebook.com"}`

inputData := UserInfo{}
json.Unmarshal([]byte(data), &inputData)
fmt.Printf("%#v", inputData)
```
Unmarshal 함수를 이용해서, JSON 형식 데이터를 Go Struct로 Unmarshal 할 수 있습니다.

## JSON Tag
Go Struct는 Tag 기능을 지원합니다(개인적으로 제일 좋아하는 기능). Tag를 이용해서 Struct를 구성하는 필드에 메타 정보를 설정할 수 있습니다.
`encoding/json` 패키지의 경우 아래와 같이 Tag를 이용해서 마샬/언마샬 방식을 설정할 수 있습니다.
```go
type UserInfo struct {
    UserID int `json:"user_id"`
    Email string `json:"email"`
    Age int `json:"age"`
    Blog string `json:"blog"`
    FaceBook string `json:"facebook"`
    Description string `json:"-"`
}
```

## omitempty
```go
type UserInfo struct {
    UserID      int    `json:"user_id"`
    Email       string `json:"email"`
    Age         int    `json:"age"`
    Blog        string `json:"blog"`
    FaceBook    string `json:"facebook"`
    Description string
}
data := UserInfo{
    UserID:      1001,
    Email:       "foo@example.com",
    Blog:        "https://blog.example.com",
    FaceBook:    "https://foo.facebook.com",
    Description: "Hello World",
}
sndData, _ := json.Marshal(data)
fmt.Println(string(sndData))
```
위 코드를 실행시 아래와 같은 내용이 출력 되게 됩니다.

```json
{
    "Description" : "Hello World",
    "age" : 0,
    "blog" : "https://blog.example.com",
    "email" : "foo@example.com",
    "facebook" : "https://foo.facebook.com",
    "user_id" : 1001
}
```
위 코드에는 문제점이 있습니다. 
- 사용자가 Age를 입력하지 않는 경우에는 0이 출력 되어선 안 됩니다.

위 문제를 해결하기 위해서 Go Struct의 각 필드는 값을 수정하지 않는 경우에는 "기본값"이 입력되게 됩니다.  
int 타입의 기본값은 0으로 설정되어 있습니다. 하지만 JSON의 omitempty를 설정하면 해당 필드의 JSON에서 제외할 수 있게 됩니다.
```go
type UserInfo struct {
    UserID      int    `json:"user_id"`
    Email       string `json:"email"`
    Age         int    `json:"age,omitempty"`
    Blog        string `json:"blog"`
    FaceBook    string `json:"facebook"`
    Description string
}
```
이제 코드를 다시 수행시켜 보자.
```json
{
  "Description" : "Hello World",
  "blog" : "https://blog.example.com",
  "email" : "foo@example.com",
  "facebook" : "https://foo.facebook.com",
  "user_id" : 1001
}
```
이제 JSON을 언마샬링 해 보겠습니다. 위 출력 데이터를 사용하였습니다.

```go
inputData := `{
    "Description" : "Hello World",
    "blog" : "https://blog.example.com",
    "email" : "foo@example.com",
    "facebook" : "https://foo.facebook.com",
    "user_id" : 1001
}`
out := UserInfo{}
json.Unmarshal([]byte(inputData), &out)
fmt.Printf("%#v", out)
```
이제 다 수정하였으니 코드를 실행 해 보겠습니다.

```json
main.UserInfo{UserID:1001, Email:"foo@example.com", Age:0, Blog:"https://blog.example.com", FaceBook:"https://foo.facebook.com", Description:"Hello World"}%                                                                    
```
아직까지도 Age 값이 0입니다. 원하는 결과가 아니죠! 저희가 원하는 결과는 nil이 되어서 값이 출력 되지 않아야 합니다.
