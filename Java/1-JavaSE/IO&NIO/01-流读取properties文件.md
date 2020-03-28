这里以一个工具来说明：

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

