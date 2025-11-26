# Changes

Code differences compared to source project demokratos.

## internal/server/http.go (+33 -0)

```diff
@@ -4,11 +4,38 @@
 	"github.com/go-kratos/kratos/v2/log"
 	"github.com/go-kratos/kratos/v2/middleware/recovery"
 	"github.com/go-kratos/kratos/v2/transport/http"
+	"github.com/gorilla/handlers"
 	v1 "github.com/orzkratos/demokratos/demo2kratos/api/helloworld/v1"
 	"github.com/orzkratos/demokratos/demo2kratos/internal/conf"
 	"github.com/orzkratos/demokratos/demo2kratos/internal/service"
 )
 
+/*
+测试 CORS 是否生效的最简单方案：
+
+在 postman 的 GET http://127.0.0.1:28000/helloworld/kratos
+在请求的 Headers 里面添加 Origin: https://www.google.com 这项
+在 postman 里面响应头里面就能看到 Access-Control-Allow-Origin: https://www.google.com 信息。
+
+或者
+
+使用 curl 命令行工具：
+curl -v --location 'http://127.0.0.1:28000/helloworld/kratos' --header 'Origin: https://www.google.com'
+输出里面有 > Origin: https://www.google.com (这是收到的请求头信息)
+输出里面有 < Access-Control-Allow-Origin: https://www.google.com 的信息。
+还有 < Vary: Origin 的信息（不重要）。
+就基本证明是大功告成啦。
+
+curl -v --location 'http://127.0.0.1:28000/helloworld/kratos' --header 'Origin: http://localhost:5173'
+输出里面有 < Access-Control-Allow-Origin: http://localhost:5173 的信息。
+还有 < Vary: Origin 的信息（不重要）。
+就基本证明是大功告成啦。
+
+curl -v --location 'http://127.0.0.1:28000/helloworld/kratos' --header 'Origin: https://www.unknown.com'
+确保里面没有 < Access-Control-Allow-Origin: 相关的信息。
+就基本证明是大功告成啦。
+*/
+
 // NewHTTPServer new an HTTP server.
 func NewHTTPServer(c *conf.Server, greeter *service.GreeterService, logger log.Logger) *http.Server {
 	var opts = []http.ServerOption{
@@ -21,6 +48,12 @@
 	}
 	if c.Http.Addr != "" {
 		opts = append(opts, http.Address(c.Http.Addr))
+		// cp from https://github.com/go-kratos/examples/blob/61daed1ec4d5a94d689bc8fab9bc960c6af73ead/http/cors/main.go#L19
+		opts = append(opts, http.Filter(handlers.CORS(
+			handlers.AllowedOrigins([]string{"https://www.google.com", "http://localhost:5173"}), //按需配置
+			handlers.AllowedMethods([]string{"GET", "POST", "OPTIONS"}),
+			handlers.AllowedHeaders([]string{"Content-Type", "Authorization"}),
+		)))
 	}
 	if c.Http.Timeout != nil {
 		opts = append(opts, http.Timeout(c.Http.Timeout.AsDuration()))
```

