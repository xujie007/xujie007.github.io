---
layout:     post
title:      springboot实现配置文件热加载
subtitle:   完结
date:       2018-10-06
author:     xujie
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - springboot
---

> 正所谓前人栽树，后人乘凉。
> 感谢[Huxpro](https://github.com/huxpro)提供的博客模板
> [我的的博客](http://my.happy-coding.cn)

# 前言
#### 在开发项目中，有时候需要我们将一些可配置的参数做成一个配置文件或者多个，但是在使用过程可能会出现这种情况：项目正在运行，但是又需要改配置文件。而且配置文件修改后还要重启服务才能生效。这样的操作就有点不太理想化，当项目有一定的用户量时，这种操作也肯定会影响到实际使用体验。那么重点来了，这篇文章就是为了解决这个问题才诞生的。

#### 写这个教程的时候我也是查询了相关的文档，发现其实有框架可以解决这个问题，比如百度的disconf和spring cloud config，但是这两个框架是解决分布式架构的。由于项目还没有使用分布式的结构，如果使用这些框架就未免大材小用了。


##### 解决思路：
- 配置文件项目外部（这么做也是为了兼容springboot项目，避免修改配置文件还需要重新打包上传）
- 定时读取配置文件
- 读取的数据放入内存

==忘了说了，我这里使用框架是springboot==


##### 详细教程：
>1.新建配置文件，将配置文件放置到指定盘符（我这里放到D:/configData文件夹下，window服务器可以指定盘符，luinx系统我一般放置到/opt/configData文件夹下) 

>2.代码
```
//新建GlobalVariable的class文件，之后读取的配置文件内容都会放到这个map集合中
//只要服务器不重启，一直会存在内存中，但是要注意，haspmap的容量是2的30次方，
//如果超过这个容量，可能会失效
public class GlobalVariable {
    private static Map<String, Object> shoolinfoSet = new HashMap<>();
    public static  Map<String, Object> getShoolinfoSet() {
        return shoolinfoSet;
    }
    public static void setShoolinfo(Map<String, Object> map) {
        GlobalVariable.shoolinfoSet = map;
    }
}

————————————————————————————————————————————————————————————————————————

//创建配置文件读取工具ConfigUtil
public class ConfigUtil {
    private static InputStream ins = null;
    private static Properties p = null;
    private static Map<String, Object>  map = GlobalVariable.getShoolinfoSet();

    static {
        try {
            File file = ResourceUtils.getFile(AutoLoadUtils.checkOSType());
            ins = new BufferedInputStream(new FileInputStream(file));
            p = new Properties();
            p.load(ins);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (ins != null) {
                try {
                    ins.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    
    /**
     * 根据key查询value,返回string类型的数据
     * @param key
     * @return
     */
    public static String getConfig(String key){
        if (!map.containsKey(key)){
            map.put(key, p.getProperty(key));
        }
        return (String) map.get(key);
    }
}

————————————————————————————————————————————————————————————————————————
//定时读取配置文件
@Configuration
@EnableScheduling
public class AutoLoadUtils {

    private final static String WINDOWS_MEMCACHED_LOG = "D:\\configData\\schoolInfo.properties";
    private final static String LINUX_MEMCACHED_LOG = "/opt/configData/schoolInfo.properties";

    private static InputStream ins = null;
    private static Properties p = null;

    private Logger log = LoggerFactory.getLogger(this.getClass());
    private static Map<String, Object>  map = GlobalVariable.getShoolinfoSet();


    /**
     * @description 自动加载学校信息（每隔5分钟扫描一次，获取最新的数据）
     * @version V1.0.0
     * @motto 昨夜西风凋碧树。独上高楼，望尽天涯路
     * @methodName AutoLoadSchoolInfo
     * @author Gypsophila
     * @date 2018/11/26
     * @param
     * @return
     */
//    @Scheduled(cron="0 0/5 * * * ?")
    @Scheduled(cron="*/5 * * * * ?")
    public void AutoLoadSchoolInfo(){
        try {
            File file = ResourceUtils.getFile(checkOSType());
            ins = new BufferedInputStream(new FileInputStream(file));
            p = new Properties();
            log.debug("[自动加载学校信息]时间：{},加载数据：{}",new Date().getTime(),p);
            p.load(ins);
            for (Enumeration e = p.propertyNames(); e.hasMoreElements(); ) {
                String key = (String)e.nextElement();
                map.put(key, p.getProperty(key));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (ins != null) {
                try {
                    ins.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    /**
     * @description 检查服务器类型
     * @version V1.0.0
     * @motto 昨夜西风凋碧树。独上高楼，望尽天涯路
     * @methodName checkOSType
     * @author Gypsophila
     * @date 2018/11/26
     * @param
     * @return
     */
    public static String checkOSType(){
        Properties properties = System.getProperties();
        String property = properties.getProperty("os.name");
        if (property.contains("Windows")){
            return WINDOWS_MEMCACHED_LOG;
        }
        return LINUX_MEMCACHED_LOG;
    }


}
```



