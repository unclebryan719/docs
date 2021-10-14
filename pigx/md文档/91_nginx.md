# nginx代理，前端请求，解决跨域
在生产中，我们建议使用nginx 代理前端请求，解决跨域。 并且把上步打包dist 目录的文件部署到nginx

# nginx.conf

location ^~/admin/ {
proxy_pass   http://127.0.0.1:9999/admin/;
}


location ^~/auth/ {
proxy_pass   http://127.0.0.1:9999/auth/;
}

location ^~/gen/ {
proxy_pass   http://127.0.0.1:9999/gen/;
}
以上配置类似于，vue-cli的proxy配置

# pig 演示环境的配置
server {
    listen 80;
    server_name xxxxx.com;
    
    # 讲打包好的dist目录文件，放置到这个目录下
    root /data/pig-ui/;
          
    location ~* ^/(code|auth|admin|gen) {
       proxy_pass http://127.0.0.1:9999;
       #proxy_set_header Host $http_host;
       proxy_connect_timeout 15s;
       proxy_send_timeout 15s;
       proxy_read_timeout 15s;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
} 