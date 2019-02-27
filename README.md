### kit
---
https://github.com/go-kit/kit

https://gokit.io/examples/

```go
import "context"

type StringService interface {
  Uppercase(string) (string, error)
  Count(string) int
}

import (
  "context"
  "errors"
  "strings"
)

type stringService struct{}

func (stringService) Uppercase(s string) (string, error) {
  if s == "" {
    return "", ErrEmpty
  }
  return strings.ToUpper(s), nil
}

func (stringService) Count(s string) int {
  return len(s)
}

var ErrEmpty = errors.New("Empty string")


type uppercaseRequest struct {
  S string `json:"s"`
}

type uppercaseResponse struct {
  V string `json:"v"`
  Err string `json:"err,omitempty"`
}

type countRequest struct {
  S string `json:"s"`
}

type countResponse struct {
  V int `json:"v"`
}

import (
  "context"
  "github.com/go-kit/kit/endpoint"
)

func makeUppercaseEndpoint(svc StringService) endpoint.Endpoint {
  return func(_ context.Context, request interface{}) (interface{}, error) {
    req := request.(uppercaseRequest)
    v, err := svc.Uppercae(req.S)
    if err != nil {
      return uppercaseResponse{v, err.Error()}, nil
    }
    return uppercaseResponse{v, ""}, nil
  }
}

func makeCountEndpoint(svc StringService) endpoint.Endpoint {
  return func(_ context.Context, request interface{}) (interface{}, error) {
    req := request.(countRequest)
    v := svc.Count(req.S)
    return countResponse{v}, nil
  }
}

func proxyingMiddleware()







```

```
```

```
```



