- 主机代理软件以V2ray为例
- v2ray中允许局域网连接
	- ![image.png](../assets/image_1765119567410_0.png){:height 325, :width 397}
- 确定局域网连接端口
	- ![image.png](../assets/image_1765119602273_0.png)
- 在`.bashrc`中添加如下命令：
	- ```bash
	  export hostip=$(ip route | grep default | awk '{print $3}')
	  export hostport=10810
	  alias proxy='
	      echo "got hostip${hostip} and port ${hostport}"
	      export https_proxy="http://${hostip}:${hostport}";
	      export http_proxy="http://${hostip}:${hostport}";
	      export all_proxy="http://${hostip}:${hostport}";
	      git config --global http.proxy "http://${hostip}:${hostport}"
	      git config --global https.proxy "http://${hostip}:${hostport}"
	      echo -e "Acquire::http::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
	      echo -e "Acquire::https::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
	  '
	  alias unproxy='
	      unset https_proxy;
	      unset http_proxy;
	      unset all_proxy;
	      git config --global --unset http.proxy
	      git config --global --unset https.proxy
	      sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
	      sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
	  '
	  ```
- 在`sourcce ~/.bashrc`之后，即可使用`proxy`与`unproxy`开启与关闭代理