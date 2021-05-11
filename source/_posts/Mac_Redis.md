---

title: 【Mac】Redis
date: 2019.9.12
categories: 
 - [环境配置]

tags: 
 - Mac
 - IntelliJ IDEA

description: 在 IntelliJ IDEA 中使用 Redis
---




# 安装Redis

1.下载Redis，得到redis-5.0.5.tar.gz，解压并移动到 `/usr/local/`
https://redis.io/download

解压：`tar zxvf redis-5.0.5.tar.gz`
移动： `mv redis-5.0.5 /usr/local/`
切换到：`cd /usr/local/redis-5.0.5/`

2.编译测试
`sudo make test`

3.编译安装
`sudo make install`

4.启动 Redis
`redis-server` ，得到下图：
{% img https://vikingstudyhard.github.io/images/Mac_Redis/AD3DC62B085E706C54C404A12E56273C.jpg 976 %}

5.测试
在pom.xml中加入Java Redis的jar包

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```
新建测试文件
```java
package jedis;
import org.junit.Test;
import redis.clients.jedis.Jedis;

/**
 * Jedis的测试方法
 */
public class JedisTest {
    @Test
    public void textJedis() throws Exception {
        //创建一个连接Jedis对象，参数:host、port
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        //直接使用Jedis操作Redis，所有Jedis命令都对应一个方法
        jedis.set("test123", "my first jedis test");
        String string = jedis.get("test123");
        System.out.println(string);
        //关闭连接
        jedis.close();
    }
}
```
运行后得到下图即可。
{% img https://vikingstudyhard.github.io/images/Mac_Redis/35EBB0C5F9102C30D124F3AF3D81A529.jpg  1018 %}

# 命令行操作数据 redis-cli
1.新开一个终端窗口，访问redis根目录
2.操作
登录redis：`redis-cli -h 127.0.0.1 -p 6379`
设置 key 的值：`set key value`
获取 key 的值：`get key`
查看此 key 是否存在：`exists key`
查看所有key值：`keys *`
删除指定索引的值：`del key`
清空整个 Redis 服务器的数据：`flushall`
清空当前库中的所有 key：`flushdb`
退出：`redis-cli shutdown`

# 安装可视化工具rdm
redis-desktop-manager 
{% img https://vikingstudyhard.github.io/images/Mac_Redis/142328514716BCAA49680ED6955853D1.jpg 374 %}
重复执行上面的测试可见
{% img https://vikingstudyhard.github.io/images/Mac_Redis/01DE26A86C6E1263B037752A00B32374.jpg  1000 %}



https://www.jianshu.com/p/56999f2b8e3b
https://blog.csdn.net/Jason_M_Ho/article/details/80007330
https://blog.csdn.net/Future_LL/article/details/84592120