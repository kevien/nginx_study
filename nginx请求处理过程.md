nginx配置文件中的每一个server块都是一个虚拟主机，当一个请求到来时，nginx选择哪一个server块来处理它呢？

###基于名字的虚拟主机

    server {
		listen      80;
		server_name example.org www.example.org;
		...
	}

	server {
		listen      80;
		server_name example.net www.example.net;
		...
	}

	server {
		listen      80;
		server_name example.com www.example.com;
		...
	}

在上述配置中，nginx会根据请求头中的Host字段，来选择交给哪个server块处理. 如果Host字段值不与任何一个server块中的server_name匹配，或者请求头中根本就不包含该字段，nginx会把该请求交给默认server块处理（上述配置文件中，第一个server块为默认server块）  

**显示设置默认server块**

	server {
		listen      80 default_server;  # 关键字，default_server
		server_name example.net www.example.net;
		...
	}

这样它将取代第一个作为默认server块


**如何阻止一个请求头不含Host字段的请求？**

	server {
		listen      80;
		server_name "";  # 注意，这里的server_name被设置成空字符串
		return      444;
	}

上述server块将会匹配一个请求头中不含Host字段的请求，并且返回444响应码给客户端  

  
  

###基于名字和IP混合的虚拟主机

	server {
		listen      192.168.1.1:80;
		server_name example.org www.example.org;
		...
	}

	server {
		listen      192.168.1.1:80;
		server_name example.net www.example.net;
		...
	}

	server {
		listen      192.168.1.2:80;
		server_name example.com www.example.com;
		...
	}



根据上述配置，nginx首先把请求访问的地址（ip & port）与server块中的listen字段做比较，若匹配，则进一步把该server块中的server_name和请求头中的Host字段比较（如文章开始所述）；如果上述两个条件都满足，则把请求交由那个server块处理，否则选择默认server块处理；  

譬如，一个请求访问www.example.com并且访问地址为192.168.1.1:80，那么该请求将会交给上述配置文件中第一个server块处理 

同样，我们可以使用**default_server**关键字来显示设置默认server块，eg:

	server {
		listen      192.168.1.1:80;
		server_name example.org www.example.org;
		...
	}

	server {
		listen      192.168.1.1:80 default_server;
		server_name example.net www.example.net;
		...
	}

	server {
		listen      192.168.1.2:80 default_server;
		server_name example.com www.example.com;
		...
	}

此时，对于刚才的那个请求，则默认由第二个server块处理  



###总结

一个请求匹配server块的过程:  
	
 1. 把请求访问的地址（ip & port）与server块中的listen字段做比较；
 2. 把请求头中的Host字段与server块中的server_name字段做比较；

nginx是根据server块中的listen字段来了解自己需要监听在哪个ip和port，所以不存在接受到一个请求，但是确不匹配任何一个server块中listen字段（因此总能找到一个server块满足条件1），如果没有找到一个server块满足条件2，那么选择默认server块处理，默认server块选择逻辑：
 
 1. 满足条件1，并且用default_server关键字显示声明的server块（显示规则）；
 2. 满足条件1，并且优先出现在配置文件中的server块（隐式规则）；

---

本文大部分内容来源：http://nginx.org/en/docs/http/request_processing.html 

