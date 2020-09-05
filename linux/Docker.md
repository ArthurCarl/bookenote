# Docker Deep Dive

## Daemon 

In a default Linux installation, the client talks to the daemon via a local IPC/Unix socket at `/var/run/docker.sock`.

## Docker engine

`Docker client` `Docker daemon` `containerd` `runc`


------

# Docker Troubleshooting

### get Container ID through PID
最近使用 `top` 查看服务器状态时，总是看到有几个应用的占用的资源很高，需要知道容器信息;
1. `ps -axfo pid,uname,cmd | grep -C 4 $pid` --获取到容器内运行的ID对应于宿主机的ID
2. `docker ps -q | xargs docker inspect -f '{{.State.Pid}}  {{.Config.Hostname}}' | grep $pid`

