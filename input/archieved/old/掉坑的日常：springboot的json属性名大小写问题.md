title: 掉坑的日常：springboot的json属性名大小写问题
author: 归零幻想
publishDate: 2020-03-24
editDate: 2020-03-24
tags: [springboot, json, github, 升级]

<!--config-->

这两天想给自己小组的项目添加一个自动升级功能。这事听着复杂，但毕竟是个很常见的功能，github上的轮子很多。经过考虑，我决定选择[这个组件](https://github.com/xuexiangjys/XUpdate)，然后自己写后端。反正后端就返回一个json的事。然后就掉了坑。

## 起因

案发现场没什么好说的，就是我发现自己设置的不能实现升级，在客户端调试半天发现用官方的json能升级。然后我就找我的json和官方的json有什么区别，看ContentType也没设置错，仔细比对发现属性大小写竟然不对。我当时还挺惊讶的，毕竟是直接复制的。

<!--summary-->

springboot可以使用`@ResponseBody`返回对象自动转换json，而转换成的json属性名首字母会被转换成小写。

## 解决方案

### 引入`fastjson`

```xml
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.66</version>
		</dependency>
```

### 注入`bean`

```java
    @Bean
    public HttpMessageConverters httpMessageConverters() {
        FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter(); // 添加fastJson的配置信息
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat); // 处理中文乱码问题
        List<MediaType> fastMediaTypes = new ArrayList<>();
        fastMediaTypes.add(MediaType.APPLICATION_JSON); // 在convert中添加配置信息.
        fastJsonHttpMessageConverter.setSupportedMediaTypes(fastMediaTypes);
        fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
        HttpMessageConverter<?> converter = fastJsonHttpMessageConverter;
        return new HttpMessageConverters(converter);
    }
```

### 在属性上加注解

```java
package edu.upc.mishuserver.vo;

import com.alibaba.fastjson.annotation.JSONField;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * UpdateInfo
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UpdateInfo {

    @JSONField(name="Code")
    private Long code;
    @JSONField(name = "Msg")
    private String msg;
    @JSONField(name = "UpdateStatus")
    private Integer updateStatus;
    @JSONField(name = "VersionCode")
    private Long versionCode;
    @JSONField(name = "VersionName")
    private String versionName;
    @JSONField(name = "ModifyContent")
    private String modifyContent;
    @JSONField(name = "DownloadUrl")
    private String downloadUrl;
    @JSONField(name = "ApkSize")
    private Long apkSize;
    @JSONField(name = "ApkMd5")
    private String apkMd5;
}
```
