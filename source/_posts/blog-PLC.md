---
title: JAVA读取PLC设备数据
date: 2024-12-13 09:49:14
categories:
  - 猛男技巧


---


# JAVA读取PLC设备数据

在工业自动化中，PLC（可编程逻辑控制器）广泛用于控制和监测工业设备。如果需要在Java应用程序中读取PLC设备的数据，可以通过第三方库与PLC进行通信。本文将介绍如何使用Java连接和读取PLC设备的数据。

## 一、环境准备

### 1.1 开发环境

- **开发工具：** IntelliJ IDEA
- **JDK版本：** 8或更高

## 二、项目设置

### 2.1 创建Maven项目

新建一个Maven项目，在`pom.xml`中添加依赖：

```
<dependency>
    <groupId>com.github.dathlin</groupId>
    <artifactId>HslCommunication</artifactId>
    <version>3.0.1</version>
</dependency>
```

### 2.2 配置PLC连接参数

定义PLC连接参数，如IP地址、端口、站点等信息。

#### 站点信息

```
private final Map<String, String> lastReadValues = new ConcurrentHashMap<>();
private final String numberAddress = "DB401.DBB0";
private final String dateAddress = "DB401.DBD22";
private final String timeAddress = "DB401.DBD26";
private final String oldPowerAddress = "DB401.DBD10";
private final String newPowerAddress = "DB401.DBD14";
private final String diffAddress = "DB401.DBD18";
```

#### IP信息

```
@Override
public void readDeviceIntoTheDB() {
    List<String> ipAddresses = Arrays.asList("192.168.10.99", "192.168.10.239");
    ipAddresses.forEach(this::insertDeviceDataIntoDB);
}
```

#### 连接配置

```
private Map<String, String> insertDeviceDataIntoDB(String ipAddress) {
    SiemensS7Net siemensS7Net = new SiemensS7Net(SiemensPLCS.S200Smart);
    siemensS7Net.setIpAddress(ipAddress);
    siemensS7Net.setPort(102);
    OperateResult result = siemensS7Net.ConnectServer();

    if (result.IsSuccess) {
        log.info("PLC设备连接成功");
    } else {
        log.error("PLC设备连接失败，请检查IP地址是否正确");
        return null;
    }

    Map<String, String> readValues = readValuesFromDevice(siemensS7Net);
    if (readValues == null) return null;

    String readPlcDataTime = readValues.get("date") + readValues.get("time");
    String cachedDateTime = lastReadValues.get(ipAddress);

    if (cachedDateTime != null && cachedDateTime.equals(readPlcDataTime)) {
        return null;
    }

    lastReadValues.put(ipAddress, readPlcDataTime);
    return lastReadValues;
}
```

### 2.3 封装读取数据逻辑

```
private Map<String, String> readValuesFromDevice(SiemensS7Net siemensS7Net) {
    Map<String, String> readValues = new HashMap<>();

    readValues.put("date", readAndLog(siemensS7Net.ReadInt32(dateAddress), "日期"));
    readValues.put("time", readAndLog(siemensS7Net.ReadInt32(timeAddress), "时间"));
    readValues.put("number", readAndLog(siemensS7Net.ReadByte(numberAddress), "炉号"));
    readValues.put("oldPower", readAndLog(siemensS7Net.ReadFloat(oldPowerAddress), "原功率"));
    readValues.put("newPower", readAndLog(siemensS7Net.ReadFloat(newPowerAddress), "现功率"));
    readValues.put("differenceValue", readAndLog(siemensS7Net.ReadFloat(diffAddress), "差值"));

    if (readValues.containsValue(null)) {
        log.warn("读取数据失败，放弃此次数据库更新操作");
        return null;
    }
    return readValues;
}
```

## 三、读取结果
![plc-01](../images/plc/plc-01.jpg)

## 四、破解限时配置（仅供学习参考）

由于HslCommunication包的免费版本有24小时读取限制，可以通过以下代码进行破解：

```
package com.iwip.config;

import HslCommunication.Authorization;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.lang.reflect.Field;

@Component
public class PLCConfig {

    @PostConstruct
    public void init() {
        Class<Authorization> authorizationClass = Authorization.class;
        try {
            Authorization authorization = authorizationClass.newInstance();
            Field field1 = authorizationClass.getDeclaredField("nuasgdawydbishcgas");
            field1.setAccessible(true);
            field1.set(authorization, 8);

            Field field2 = authorizationClass.getDeclaredField("nuasgdaaydbishdgas");
            field2.setAccessible(true);
            field2.set(authorization, 10000);

            Field field3 = authorizationClass.getDeclaredField("naihsdadaasdasdiwasdaid");
            field3.setAccessible(true);
            field3.set(authorization, 12345);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**注意：** 此配置仅供学习参考，生产环境中请遵循相关法律法规。
