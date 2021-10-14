# pig 默认没有开启 Logfile viewer
image
![avator](http://pic.pig4cloud.com/20190706115555_pL1RHZ_Screenshot.jpeg?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


# 以UPMS模块为例
在 bootstrap.yml 配置 logging.file 的配置即可开启Logfile viewr

# 配置Logfile Viewer

logging:
  file: logs/${spring.application.name}/debug.log
# 启动 pig-monitor
选中 PIG-UPMS 服务 
![avator](http://pic.pig4cloud.com/20190221130054_iVj4Yp_Screenshot.jpeg)
![avator](http://pic.pig4cloud.com/20190221130140_1sByxe_Screenshot.jpeg)
![avator](http://pic.pig4cloud.com/20190221130159_3qtI7c_Screenshot.jpeg)