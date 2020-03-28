[JWT](../../01-base/02-HTTP/HTTP.md#JWT)

# 使用

导入依赖：

```xml
<!--JWT核心依赖-->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.3.0</version>
</dependency>
<!--java开发JWT的依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

编写 JWT 工具类，如：

```java
public class JwtUtils {
    private static final ObjectMapper MAPPER = new ObjectMapper(); //jackson中对bean和json互相转换
    //生成密钥
    public static SecretKey generalKey(){
        //xyz作为服务器的key，用于做加解密的key，自定义即可。
        byte[] encodedKey = "xyz".getBytes("UTF-8");
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    /**
     * 签发JWT，创建token的方法
     * @param id JWT的唯一身份标识，主要用来作为一次性token，从而回避重放攻击
     * @param subject JWT面向的用户。payload中记录的public claims
     * @param ttlMillis 有效期，单位为毫秒
     * @return  String token，token是一次性的，是为了一个用户的有效登录周期准备的一个token。用户退出或超时，token失效
     */
    public static String createJWT(String id, String subject, long ttlMillis) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;//加密算法
        long nowMillis = System.currentTimeMillis(); //当前时间
        Date now = new Date(nowMillis); //当前时间的日期对象
        SecretKey secretKey = generalKey();
        //创建JWT的构建器
        JwtBuilder builder = Jwts.builder()
                .setId(id)   //设置身份标志，就是一个客户端的唯一标记，如用户主键、ip、随机数
                .setSubject(subject)
                .setIssuer("user")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey); // 签名算法以及密钥
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date expDate = new Date(expMillis);
            builder.setExpiration(expDate); // 过期时间
        }
        return builder.compact(); //生成token
    }
    /**
     * 验证JWT
     * @param jwtStr
     * @return
     */
    public static JWTResult validateJWT(String jwtStr) {
        //一个自定义对象，里面有错误代码code、校验结果success、Claims对象(校验过程中payload中的数据)
        JWTResult checkResult = new JWTResult();
        Claims claims = null;
        try {
            claims = parseJWT(jwtStr);
            checkResult.setSuccess(true);
            checkResult.setClaims(claims);
        } catch (ExpiredJwtException e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_EXPIRE); //状态码，代表token过时
            checkResult.setSuccess(false);
        } catch (SignatureException e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_FAIL); //代表校验失败
            checkResult.setSuccess(false);
        } catch (Exception e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_FAIL); //其他异常
            checkResult.setSuccess(false);
        }
        return checkResult;
    }

    /**
     * 解析JWT字符串
     * @param jwt 服务器为客户端生成的签名数据，就是token
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(jwt)
            .getBody(); //getBody获取的就是token中记录的payload数据，即payload中所有的claims
    }

    /**
     * 生成subject信息
     * @param subObj 要转换的对象
     * return java对象——>JSON字符串出错时返回null
     */
    public static String generatSubject(Object subObj){
        try{
            return MAPPER.writeValueAsString(subObj);
        } catch(Exceprion e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

使用：

```java
public class LoginController {
    @Autowired
    UserRepository userRepository;

    @RequestMapping(value="login",method = RequestMethod.POST)
    public String login(String username, String password,HttpServletResponse response) {
        User user =  userRepository.findByUsername(username);
        if(user!=null){
            if(user.getPassword().equals(password)){
                //把token返回给客户端-->客户端保存至cookie-->客户端每次请求附带cookie参数
                String JWT = JwtUtils.createJWT("1", username, SystemConstant.JWT_TTL);
                return ReturnVo.ok(JWT); //ReturnVo是一个专门用于返回给前端的响应类
            }else{
                return ReturnVo.error();
            }
        }else{
            return ReturnVo.error();
        }
    }
}
```
