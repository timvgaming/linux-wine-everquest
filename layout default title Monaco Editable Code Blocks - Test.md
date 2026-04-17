## layout: default title: Monaco Editable Code Blocks - Test

# Monaco Editable Code Blocks (Test)

This page demonstrates turning fenced code blocks marked with `editable` into live Monaco editors (VS Code editor in the browser).



> ⚠️ Important: This only works on your GitHub Pages site (e.g. `https://timvg.github.io/repo/`). It will not run when viewing the file on `github.com`.



## Normal (non-editable) code block — stays read-only



```en-us
echo "This one should remain a normal code block."
```



## Editable block (bash)



```en-us
# Change these values for your system
APP_PORT=8080
DATA_DIR=/var/lib/myapp
LOG_LEVEL=info

docker run --rm \
  -p ${APP_PORT}:${APP_PORT} \
  -v ${DATA_DIR}:/data \
  -e LOG_LEVEL=${LOG_LEVEL} \
  myapp:latest
```



## Editable block (YAML)



```en-us
server:
  port: 8080
storage:
  path: /var/lib/myapp
logging:
  level: info
```



## Editable block (PowerShell)



```en-us
$AppPort = 8080
$DataDir = "C:\\myapp\\data"
$LogLevel = "info"

docker run --rm `
  -p "$AppPort:$AppPort" `
  -v "${DataDir}:/data" `
  -e "LOG_LEVEL=$LogLevel" `
  myapp:latest
```