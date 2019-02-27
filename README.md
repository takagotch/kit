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

func proxyingMiddleware(instances string, logger log.Logger) Service Middleware {
  
  if instaces == "" {
    logger.Log("proxy_to", "none")
    return func(next StringService) StringService { return next }
  }
  
  var (
    ops = 100
    maxAttempts = 3
    maxTime = 250 * time.Millisecond
  )
  
  var (
    instanceList = split(instances)
    subscriber sd.FixedSubscriber
  )
  logger.Log("proxy_to", fmt.Sprint(instanceList))
  for _, instance := range instanceList {
    var e endpoint.Endpoint
    e = makeUppercaseProxy(instance)
    e = circuitbreaker.Gobreaker(gobreaker.NewCircuitBreaker(gobreaker.Settings{}))(e)
    e = kitratelimit.NewTokenBucketLimiter(jujuratelimit.NewBucketWithRate(float64(qps), int64(qps)))(e)
    subscriber = append(subscriber, e)
  }
  
  balancer := lb.NewRoundRobin(subscriber)
  retry := lb.Retry(maxAttempts, maxtime, balancer)
  
  return func(next StringService) StringService {
    return proxymw{next, retry}
  }
}


import (
  stdprometheus "github.com/prometheus/client-golang/rometheus"
  kitprometheus "github.com/go-kit/kit/metrics/prometheus"
  "github.com/go-kit/kit/metrics"
)

func main() {
  logger := log.NewLogfmtLogger(os.Stderr)
  
  fieldKeys := []string{"method", "error"}
  requestCount := kitprometheus.NewCounterFrom(stdprometheus.CounterOpts{
    Namespace: "my_group",
    Subsystem: "string_service",
    Name: "request_count",
    Help: "Number of requests received."
  }, fieldKeys)
  requestlatency := kitprometheus.NewSummaryFrom(stdprometheus.SummaryOpts {
    Namespace: "my_gropu",
    Subsystem: "stringservice",
    Name: "request_latency_microseconds",
    Help: "Total duration of requests in microseconds.",
  }, fieldKeys)
  countResult := kitprometheus.NewSummaryFrom(stdprometheus.SummaryOpts{
    Namespace: "my_group",
    Subsystem: "string_service",
    Name: "count_result",
    Help: "The result of each count method.",
  }, []string{})
  
  var svc StringService
  svc = stringService{}
  svc = logginMiddleware{logger, svc}
  svc = instrumentingMiddleware{requestCount, requstLatency, countresult, svc}
  
  uppercaseHandler := httptransport.NewServer(
    makerUpperecaseEndpoint(svc),
    decodeUppercaseRequst,
    encodeResponse,
  )
  
  countHandler := httptransport.NewServer(
    makeCountEndpoint(svc),
    decodeCountRequest,
    encodeResponse,
  )
  
  http.Handle("/uppercase", uppercaseHandler)
  http.Handle("/count", countHandler)
  http.Handle("/metrics", promhttp.Handler())
  logger.Log("msg", "HTTP", "addr", ":8080")
  logger.Log("err", http.ListenAndServe(":8080", nil))
}
```

```
```

```
```



