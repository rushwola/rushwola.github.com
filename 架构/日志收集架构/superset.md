启动命令。

nohup /usr/local/bin/gunicorn -w 2 --timeout 60 -b  0.0.0.0:8088 --limit-request-line 0 --limit-request-field_size 0 superset:app & >>/usr/local/superset.log 2>&1  &
