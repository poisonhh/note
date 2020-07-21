# Spring

## bean的作用域

- singleton：单例模式，在整个Spring IOC容器中，使用singleton定义的bean只有一个实例
- prototype：原先模式，每次通过容器的getBean方法获取prototype定义的bean时，都产生一个新的bean实例

**只有在Web应用中使用Spring时，request、session、global-session作用域才有效**

- request：对于每次HTTP请求，使用request定义的bean都将产生一个新实例，即每次HTTP请求将会产生不用的bean实例
- session：同一个session共享一个bean实例
- global-session：同session作用域不同的是，所有的session共享一个bean实例

