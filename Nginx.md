# Nginx

## 负载均衡的6种策略及原理

- 轮询（默认）

  每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

  轮询是Nginx支持的默认负载均衡策略，轮询策略就是指每个请求会按时间顺序逐一分配到不同的后台服务器上。比如说一个集群中只有服务器A和服务器B，第一次访问是服务器A，第二次访问就是服务器B，第三次访问就是服务器A...以此类推。

  ```nginx
  upstream balanceServer {
      server localhost:8081;
      server localhost:8082;
      server localhost:8083;
      server localhost:8084;
  }
  ```

  轮询策略提供如下参数：

  | 参数         | 描述                                                         |
  | :----------- | ------------------------------------------------------------ |
  | fail_timeout | 与max_fails结合使用，表示max_fails次失败后服务器暂停的时间。 |
  | max_fails    | 设置在fail_timeout参数设置的时间内最大失败次数，默认是1，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了，返回proxy_next_upstream模块定义的错误。 |
  | fail_time    | 服务器会被认为停机的时间长度,默认为10s。                     |
  | backup       | 标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里，当其他所有的非backup机器down掉或者繁忙的时候才会请求backup服务器，因此这台机器压力会最低。 |
  | down         | 标记服务器永久停机了，表示当前的server暂时不参与负载。       |
  | weight       | 负载的权重，默认为1。weight越大，表示这台服务器被访问的几率就越大。 |

  在轮询策略下，如果集群中的某个服务器挂掉了，就会自动剔除该服务器。

  轮询策略适合服务器配置相当，无状态且短平快的服务使用。

- 指定权重

  指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况

  ```nginx
  upstream balanceServer {
      server localhost:8081;
      server localhost:8082 backup;
      server localhost:8083 max_fails=3 fail_timeout=20s;
      server localhost:8084 weight=2;
  }
  ```

  在上面的例子中，weight参数用来指定轮询的几率，weight的默认值为1，weight的数值与访问比率成正比，比如8084端口的服务器被访问的几率是其他服务器的两倍。

  权重越高，分配到需要处理的请求就越多。

  权重策略可以与least_conn和ip_hash结合使用。

  权重策略比较适合服务器的硬件配置差别较大的情况。

- IP绑定ip_hash

  每个请求按访问ip的hash结果分配，这样每个访问固定访问一个后端服务器，可以解决session的问题

  ```nginx
  upstream balanceServer {
      ip_hash; # 指定负载均衡策略为ip_hash
      server localhost:8081;
      server localhost:8082 backup;
      server localhost:8083 max_fails=3 fail_timeout=20s;
      server localhost:8084 weight=2;
  }
  ```

  在Nginx的1.3.1版本之前，不能在这种策略种使用权重（weight）。

  ip_hash不能和backup参数同时使用。

  这个策略适用于有状态服务，比如说Session会话。

  当有服务器需要剔除的时候，必须手动down掉

- 最少连接（least_conn）

  这个策略是把请求转发给连接数较少的后端服务器。前面的轮询策略是把请求平均地转发给集群中的每个后台服务器，使得它们的负载大致相同，但是有些请求可能占用的时间会很长，可能导致所在的后端负载过高。这种情况下选用least_conn策略就能达到更好的负载均衡效果。

  ```nginx
  upstream balanceServer {
      least_conn; # 指定负载均衡策略为least_conn
      server localhost:8081;
      server localhost:8082 backup;
      server localhost:8083 max_fails=3 fail_timeout=20s;
      server localhost:8084 weight=2;
  }
  ```

  这个策略适合用在请求处理时间长短不一造成服务器过载的场景

- fair（第三方）

  按后端服务器的响应时间来分配请求，响应时间短的优先分配

  ```nginx
  upstream balanceServer {
      fair; # 指定负载均衡策略为fair
      server localhost:8081;
      server localhost:8082;
      server localhost:8083;
      server localhost:8084;
  }
  ```

  fair策略是一种第三方策略，需要安装第三方插件。

- url_hash（第三方）

  按访问url的hash结果类分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效

  这种策略是按照访问url的hash结果来分配请求，使得每个url定向到同一个后端服务器，要配合缓存命中来使用。同一个资源多次请求，可能会到达不同的服务器上，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。而使用url_hash的话，就可以使得同一个url（也就是同一个资源请求）到达同一台服务器，一旦缓存住了资源，再次收到请求，就可以从缓存中读取。

  ```nginx
  upstream backserver { 
      server squid1:3128; 
      server squid2:3128; 
      hash $request_uri; 
      hash_method crc32; 
  } 
  ```

  在需要使用负载均衡的server中增加

  ```nginx
  proxy_pass http://backserver/; 
  upstream backserver{ 
      ip_hash; 
      server 127.0.0.1:9090 down; (down 表示当前的server暂时不参与负载) 
      server 127.0.0.1:8080 weight=2; (weight 默认为1.weight越大，负载的权重就越大) 
      server 127.0.0.1:6060; 
      server 127.0.0.1:7070 backup; (其它所有的非backup机器down或者忙的时候，请求backup机器) 
  } 
  ```

  max_fails：允许请求失败的次数默认为1，当超过最大次数时，返回proxy_next_upstream模块定义的错误

  fail_timeout: max_fails 次失败后，暂停的时间