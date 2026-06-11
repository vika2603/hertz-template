# Hertz + FX Template

Custom hz template for generating Hertz projects with Uber FX dependency injection.

## Usage

```bash
# Create project. No --idl needed: the layout ships a small hand-written
# ping demo (biz/router/ping/ping.go), so the project compiles and serves
# GET /api/v1/ping -> {"message":"pong"} right after creation.
hz new --mod=github.com/xxx/xxx --service=xxx \
  --customize_layout=./hertz-template/layout.yaml \
  --customize_package=./hertz-template/package.yaml

# Run
go mod tidy && go run ./cmd/server
```

`./hertz-template/` above is a placeholder for wherever you cloned this template
repo — adjust both paths to match your checkout.

The ping demo is just a starter (hand-written, no IDL). It lives in
`biz/handler/ping.go` + `biz/router/ping/`; delete those and the
`ping.Register(r)` line in `biz/router/register.go` once you add real services.

## Adding New Services

1. Create `idl/<svc>/<svc>.proto` (e.g. `idl/echo/echo.proto`). It MUST declare a
   `package`, a `go_package`, import `hertz/api.proto`, and put an HTTP
   annotation on every rpc — otherwise hz falls back to the package name `model`
   and generates empty routers:

   ```proto
   syntax = "proto3";
   package echo;                 // drives the generated package/dir names
   option go_package = "echo";
   import "hertz/api.proto";      // required for the route annotations below

   message EchoRequest  { string message = 1; }
   message EchoResponse { string reply = 1; }

   service EchoService {
     rpc Echo(EchoRequest) returns (EchoResponse) {
       option (api.post) = "/api/v1/echo";   // without this the route is empty
     }
   }
   ```

2. `hz update --idl=idl/echo/echo.proto --customize_package=./hertz-template/package.yaml --proto_path=idl`
   - `--proto_path=idl` lets `import "hertz/api.proto"` resolve to
     `idl/hertz/api.proto` no matter which subdirectory the service proto lives
     in. The new module is auto-registered at the `//INSERT_POINT` marker in
     `biz/router/register.go`.
   - Shortcut: `make hz IDL=idl/echo/echo.proto` (point `HZ_PKG` at this
     template's `package.yaml`).
3. Implement `biz/service/xxx/*.impl.go`
