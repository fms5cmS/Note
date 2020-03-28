工具类！！！

# 1.BigDecimalUtil

```java
/**
 * 解决浮点型商业运算中丢失精度的问题的工具类
 * 一定要使用BigDecimal参数类型为String构造器！
 * @author +1day
 */
public class BigDecimalUtil {
    /**禁止外部创建该对象实例*/
    private BigDecimalUtil(){}

    public static BigDecimal add(double v1, double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2);
    }

    public static BigDecimal sub(double v1, double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2);
    }

    public static BigDecimal mul(double v1, double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2);
    }

    public static BigDecimal div(double v1, double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        //四舍五入，保留两位小数
        return b1.divide(b2,2,BigDecimal.ROUND_HALF_UP);
    }
}
```



# 2.DateTimeUtil

```java
import org.apache.commons.lang3.StringUtils;  //导入commons-lang3包
import org.joda.time.DateTime;  //导入joda-time包
import org.joda.time.format.DateTimeFormat;
import org.joda.time.format.DateTimeFormatter;

import java.util.Date;

/**使用joda-time来完成*/
public class DateTimeUtil {

    public static final String STANDARD_FORMAT = "yyyy-MM-dd HH:mm:ss";

    /**
     * 字符串——>str
     * 使用自定义的格式来转换
     * @param dateTimeStr
     * @param formatStr
     * @return
     */
    public static Date str2Date(String dateTimeStr,String formatStr){
        DateTimeFormatter dateTimeFormat = DateTimeFormat.forPattern(formatStr);
        DateTime dateTime = dateTimeFormat.parseDateTime(dateTimeStr);
        return dateTime.toDate();
    }

    /**
     * date——>字符串
     * 使用自定义格式来转换
     * @param date
     * @param formatStr
     * @return
     */
    public static String date2Str(Date date,String formatStr){
        if(date == null){
            return StringUtils.EMPTY;
        }
        DateTime dateTime = new DateTime(date);
        return dateTime.toString(formatStr);
    }

    /**
     * 字符串——>date
     * 使用默认格式转换
     * @param dateTimeStr
     * @return
     */
    public static Date str2Date(String dateTimeStr){
        DateTimeFormatter dateTimeFormat = DateTimeFormat.forPattern(STANDARD_FORMAT);
        DateTime dateTime = dateTimeFormat.parseDateTime(dateTimeStr);
        return dateTime.toDate();
    }

    /**
     * date——>字符串
     * 使用默认格式转换
     * @param date
     * @return
     */
    public static String date2Str(Date date){
        if(date == null){
            return StringUtils.EMPTY;
        }
        DateTime dateTime = new DateTime(date);
        return dateTime.toString(STANDARD_FORMAT);
    }
}
```



# 3.JsonUtil

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.io.IOException;
import java.text.SimpleDateFormat;

/**
 * Json数据和对象之间相互转化的工具类
 * @author +1day
 */
@Slf4j
public class JsonUtil {
    private static ObjectMapper objectMapper = new ObjectMapper();

    //这里直接对objectMapper初始化，也可像RedisPool中那样：创建一个init方法，然后在静态块中调用

    static {
        // =============序列化的相关配置========

        /**
         * ALWAYS：序列化对象的所有属性
         * NON_NULL：仅序列化非NULL的属性，内容为空的集合、""、" "也会被序列化
         * NON_DEFAULT：仅序列化没有默认值的属性，如果属性有默认值，但会给它赋一个不同于默认值的值时，此时也会序列化该属性
         * NON_EMPTY：值为 NULL、"" 的属性不序列化，但是值为" "的属性也会序列化。内容为空的集合也不会被序列化
         */
        objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);

        /**取消默认将日期转换为TIMESTAMPS形式(毫秒)，而使用：2019-01-06T03:19:08.415+0000 这样的格式，在下面对需要的格式进行了设置*/
        objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);

        /**忽略空Bean转json的错误*/
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);

        // =============序列化和反序列化通用=============

        /**所有的日期格式都统一为：yyyy-MM-dd HH:mm:ss*/
        objectMapper.setDateFormat(new SimpleDateFormat(DateTimeUtil.STANDARD_FORMAT));

        // ============反序列化的相关配置================

        /**忽略以下情况：json字符串中存在，但java对象中不存在对应属性。防止错误*/
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }

    /**
     * 将对象序列化为json字符串
     * @param obj 对象
     * @param <T>
     * @return json字符串
     */
    public static <T> String obj2Str(T obj) {
        if (obj == null) {
            return null;
        }
        try {
            return obj instanceof String ? (String) obj :objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            log.warn("Parse Object to String error",e);
            return null;
        }
    }

    /**
     * 将对象序列化为格式化好的json字符串
     * @param obj 对象
     * @param <T>
     * @return 格式化好的json字符串
     */
    public static <T> String obj2StrPretty(T obj) {
        if (obj == null) {
            return null;
        }
        try {
            return obj instanceof String ? (String) obj :objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj);
        } catch (Exception e) {
            log.warn("Parse Object to String error",e);
            return null;
        }
    }

    /**
     * 将json字符串反序列化为对象.
     * 注意：无法将字符串反序列化为集合！
     * @param str json字符串
     * @param clazz 对象类型
     * @param <T>
     * @return
     */
    public static <T> T str2Obj(String str, Class<T> clazz) {
        if (StringUtils.isEmpty(str) || clazz == null) {
            return null;
        }
        try {
            return clazz.equals(String.class) ? (T) str : objectMapper.readValue(str, clazz);
        } catch (IOException e) {
            log.warn("Parse String to Object error", e);
            return null;
        }
    }

    /**
     * 反序列化。支持将字符串反序列化为集合。重要！
     * @param str Json字符串
     * @param typeReference 如：new TypeReference<List<User>>()
     * @param <T>
     * @return
     */
    public static <T> T str2Obj(String str, TypeReference<T> typeReference) {
        if (StringUtils.isEmpty(str) || typeReference == null) {
            return null;
        }
        try {
            return typeReference.getType().equals(String.class) ? (T) str : objectMapper.readValue(str, typeReference);
        } catch (IOException e) {
            log.warn("Parse String to Object error", e);
            return null;
        }
    }

    /**
     * 反序列化。支持将字符串反序列化为集合。重要！
     * @param str Json字符串
     * @param collectionClass 集合类型
     * @param elementClasses 集合中元素的类型
     * @param <T>
     * @return
     */
    public static <T> T str2Obj(String str, Class<?> collectionClass, Class<?>... elementClasses) {
        JavaType javaType = objectMapper.getTypeFactory().constructParametricType(collectionClass, elementClasses);
        try {
            return objectMapper.readValue(str,javaType);
        } catch (IOException e) {
            log.warn("Parse String to Object error", e);
            return null;
        }
    }
}
```



# 4.PropertiesUtil

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Properties;

/**
 * 流读取properties配置文件
 * @author +1day
 */
@Slf4j
public class PropertiesUtil {

    private static Properties props;

    /**
     * 为了在服务器启动时就读取到配置，故使用静态块，在类被加载时执行，且仅执行一次
     */
    static {
        String fileName = "shop.properties";
        props = new Properties();
        try {
            props.load(
                    new InputStreamReader(
                            PropertiesUtil.class.getClassLoader().getResourceAsStream(fileName),
                            "UTF-8"));
        } catch (IOException e) {
            log.error("配置文件读取异常",e);
        }
    }

    public static String getProperty(String key){
        //注意这里要避免key两端的空格
        String value = props.getProperty(key.trim());
        if(StringUtils.isBlank(value)){
            return null;
        }
        return value.trim();
    }

    public static String getProperty(String key,String defaultValue){
        String value = props.getProperty(key.trim());
        if(StringUtils.isBlank(value)){
            value = defaultValue;
        }
        return value.trim();
    }
}
```



# 5.RedisPoolUtil

```java
package com.fms5cms.myshop.utils;

import com.fms5cms.myshop.common.RedisPool;
import lombok.extern.slf4j.Slf4j;
import redis.clients.jedis.Jedis;

/**
 * 封装 Jedis 的 API
 * 主要用于存储Cookie和忘记密码时的token，所以仅使用String类型的相关API。
 * 为了分布式架构，使用RedisShardedPoolUtil
 * @author +1day
 */
@Slf4j
public class RedisPoolUtil {
    /**
     * 设置值
     * @param key 键
     * @param value 值
     * @return 操作结果
     */
    public static String set(String key, String value) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.set(key, value);
        } catch (Exception e) {
            log.error("set key:{} value:{} error",key,value,e);
        }
        //释放资源
        RedisPool.returnResource(jedis);
        return result;
    }

    /**
     * 获取值
     * @param key 键
     * @return 值
     */
    public static String get(String key) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.get(key);
        } catch (Exception e) {
            log.error("get key:{} error",key,e);
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    /**
     * 设置值的同时设置有效期
     * @param key 键
     * @param value 值
     * @param exTime 有效期，单位为秒
     * @return 操作结果
     */
    public static String setEx(String key, String value, int exTime) {
        Jedis jedis = null;
        String result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.setex(key,exTime,value);
        } catch (Exception e) {
            log.error("setEx key:{} exTime:{} value:{} error",key,exTime,value,e);
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    /**
     * 设置key的有效期
     * @param key 键
     * @param exTime 有效期，单位为秒
     * @return 操作结果，1为成功，0为失败
     */
    public static Long expire(String key, int exTime) {
        Jedis jedis = null;
        Long result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.expire(key,exTime);
        } catch (Exception e) {
            log.error("expire key:{} error",key,e);
        }
        RedisPool.returnResource(jedis);
        return result;
    }

    /**
     * 删除操作
     * @param key 键
     * @return 操作结果
     */
    public static Long del(String key) {
        Jedis jedis = null;
        Long result = null;

        try {
            jedis = RedisPool.getJedis();
            result = jedis.del(key);
        } catch (Exception e) {
            log.error("del key:{} error",key,e);
        }
        RedisPool.returnResource(jedis);
        return result;
    }
}
```



# 6.RedisShardedPoolUtil

```java
package com.fms5cms.myshop.utils;

import com.fms5cms.myshop.common.RedisShardedPool;
import lombok.extern.slf4j.Slf4j;
import redis.clients.jedis.ShardedJedis;

/**
 * 封装 Jedis 的 API
 * 主要用于存储Cookie和忘记密码时的token，所以仅使用String类型的相关API。
 * @author +1day
 */
@Slf4j
public class RedisShardedPoolUtil {
    /**
     * 设置值
     * @param key 键
     * @param value 值
     * @return 操作结果
     */
    public static String set(String key, String value) {
        ShardedJedis jedis = null;
        String result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.set(key, value);
        } catch (Exception e) {
            log.error("set key:{} value:{} error",key,value,e);
        }
        //释放资源
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 获取值
     * @param key 键
     * @return 值
     */
    public static String get(String key) {
        ShardedJedis jedis = null;
        String result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.get(key);
        } catch (Exception e) {
            log.error("get key:{} error",key,e);
        }
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 设置值的同时设置有效期
     * @param key 键
     * @param value 值
     * @param exTime 有效期，单位为秒
     * @return 操作结果
     */
    public static String setEx(String key, String value, int exTime) {
        ShardedJedis jedis = null;
        String result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.setex(key,exTime,value);
        } catch (Exception e) {
            log.error("setEx key:{} exTime:{} value:{} error",key,exTime,value,e);
        }
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 设置key的有效期
     * @param key 键
     * @param exTime 有效期，单位为秒
     * @return 操作结果，1为成功，0为失败
     */
    public static Long expire(String key, int exTime) {
        ShardedJedis jedis = null;
        Long result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.expire(key,exTime);
        } catch (Exception e) {
            log.error("expire key:{} error",key,e);
        }
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 删除操作
     * @param key 键
     * @return 操作结果
     */
    public static Long del(String key) {
        ShardedJedis jedis = null;
        Long result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.del(key);
        } catch (Exception e) {
            log.error("del key:{} error",key,e);
        }
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 当key不存在时才设置值。用于分布式锁的
     * @param key
     * @param value
     * @return
     */
    public static Long setnx(String key, String value) {
        ShardedJedis jedis = null;
        Long result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.setnx(key, value);
        } catch (Exception e) {
            log.error("setnx key:{} value:{} error",key,value,e);
        }
        //释放资源
        RedisShardedPool.returnResource(jedis);
        return result;
    }

    /**
     * 先获取再设值。该操作具有原子性
     * @param key
     * @param value
     * @return
     */
    public static String getSet(String key, String value) {
        ShardedJedis jedis = null;
        String result = null;

        try {
            jedis = RedisShardedPool.getJedis();
            result = jedis.getSet(key, value);
        } catch (Exception e) {
            log.error("setnx key:{} value:{} error",key,value,e);
        }
        //释放资源
        RedisShardedPool.returnResource(jedis);
        return result;
    }
}
```



# 7.CookieUtil

```java
package com.fms5cms.myshop.utils;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author +1day
 */
@Slf4j
public class CookieUtil {
    /**
     * 将Cookie放在一级域名下
     * 这样该一级域名下的二级域名如abc.fms5cms.com也可以读到该Cookie
     * 注意：Tomcat8.5以后，解析Cookie发生改变，domain必须以数字或字母开头，而不能以点开始，所以会报“An invalid domain was specified for this cookie”错误，为了使得Tomcat可以解析以点开头的domain，必须修改Tomcat目录下/conf/content.xml文件：
     * 在<context></context>标签中加入：<CookieProcessor className="org.apache.tomcat.util.http.LegacyCookieProcessor" />
     */
    private final static String COOKIE_DOMAIN = ".fms5cms.com";

    private final static String COOKIE_NAME = "fms5cms_login_token";

    /**
     * 写入登录Cookie
     * 简要说明domain和path：
     * X：domain=.fms5cms.com
     * A：A.fms5cms.com              cookie:  domain=A.fms5cms.com   path="/"
     * B：B.fms5cms.com              cookie:  domain=B.fms5cms.com   path="/"
     * C: A.fms5cms.com/test/cc       cookie:  domain=A.fms5cms.com   path="/test/cc"
     * D: A.fms5cms.com/test/dd       cookie:  domain=A.fms5cms.com   path="/test/dd"
     * E: A.fms5cms.com/test          cookie:  domain=A.fms5cms.com   path="/test"
     * @param response 由于要将Cookie写入客户端，所以需要Response
     * @param token 令牌
     */
    public static void writeLoginToken(HttpServletResponse response,String token) {
        Cookie ck = new Cookie(COOKIE_NAME,token);
        ck.setDomain(COOKIE_DOMAIN);
        //设置在根目录，根目录下的所有目录下的页面和代码都能获取到Cookie
        //如果设置为test，则只有test下的目录中的页面和代码才能获取Cookie
        ck.setPath("/");
        ck.setHttpOnly(true);
        //设置Cookie的有效期，单位为秒，-1代表永久。这里设为一年
        //如果不设置这个MaxAge，Cookie就只会写入内存，而不写入硬盘，只在当前页面有效
        ck.setMaxAge(60 * 60 * 24 *365);

        log.info("write cookieName:{},cookieValue:{}", ck.getName(), ck.getValue());
        response.addCookie(ck);
    }

    /**
     * 读取登录Cookie
     * @param request Cookie要从用户发给服务端的请求中获取
     * @return Cookie的值
     */
    public static String readLoginToken(HttpServletRequest request) {
        Cookie[] cks = request.getCookies();
        if (cks != null) {
            for (Cookie ck : cks) {
                log.info("read cookieName:{},cookieValue:{}", ck.getName(), ck.getValue());
                if (StringUtils.equals(ck.getName(), COOKIE_NAME)) {
                    log.info("return cookieName:{},cookieValue:{}", ck.getName(), ck.getValue());
                    return ck.getValue();
                }
            }
        }
        return null;
    }

    /**
     * 注销登录时需要删除Cookie
     * 先从客户端获取Cookie，将Cookie有效期设为0，再重新写入客户端，客户端会自动删除Cookie
     * @param request 客户端发来的请求
     * @param response 服务端发给客户端的响应
     */
    public static void delLoginToken(HttpServletRequest request, HttpServletResponse response) {
        Cookie[] cks = request.getCookies();
        if (cks != null) {
            for (Cookie ck : cks) {
                if (StringUtils.equals(ck.getName(), COOKIE_NAME)) {
                    ck.setDomain(COOKIE_DOMAIN);
                    ck.setPath("/");
                    //将Cookie有效期设置为0即代表删除该Cookie
                    ck.setMaxAge(0);
                    log.info("del cookieName:{},cookieValue:{}", ck.getName(), ck.getValue());
                    response.addCookie(ck);
                    return;
                }
            }
        }
    }
}
```

