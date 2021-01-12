---
title: "Java正则"
layout: page
date: 2020-11-24 00:00
---

[TOC]

# Java正则

leetcode: 《468. 验证IP地址》

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

class Solution {
    public String validIPAddress(String IP) {
        if(IP == null){
            return "Neither";
        }
        if(ipV4(IP)){
            return "IPv4";
        }
        String[] IP6Arr = IP.split(":",-1);
        if(IP6Arr.length == 8){
            return isIP6Arr(IP6Arr);
        }
        return "Neither";
    }

    boolean ipV4(String ip){
        if(ip == null){
            return false;
        }
        String p1 = "25[0-5]";
        String p2 = "2[0-4][0-9]";
        String p3 = "1[0-9]{2}";
        String p4 = "[1-9]?[0-9]";
        String patternOne = String.format("((%s)|(%s)|(%s)|(%s))", p1, p2, p3, p4);
         String patternIp = String.format("^(%s\\.){3}%s$", patternOne, patternOne);
        Pattern pattern = Pattern.compile(patternIp);
        // 匹配
        Matcher matcher = pattern.matcher(ip);
        return matcher.find();
    }

    public String isIP6Arr(String[] IP6Arr){
        for(String ip : IP6Arr){
            if(ip.length() > 4 || ip.length() <= 0){
                return "Neither";
            }
            for(int i = 0 ;i < ip.length();++i){
                char c = ip.charAt(i);
                if(!Character.isDigit(c) && !( 'a' <= c && c <= 'f') && !('A' <= c && c <= 'F')){
                    return "Neither";
                }
            }
        }
        return "IPv6";
    }

}
```

* 常规方法

```java
class Solution {
    public String validIPAddress(String IP) {
        if(IP == null){
            return "Neither";
        }
        if(ipV4(IP)){
            return "IPv4";
        }
        String[] IP6Arr = IP.split(":",-1);
        if(IP6Arr.length == 8){
            return isIP6Arr(IP6Arr);
        }
        return "Neither";
    }

     boolean ipV4(String ip){
        if(ip == null){
            return false;
        }
        int n = ip.length();
        if(n < 7 || n > 15){
            return false;
        }
        if(ip.charAt(0) == '.' || ip.charAt(n-1) == '.'){
            return false;
        }
        for(int i=0;i<n;i++){
            if(!(ip.charAt(i) >= '0' && ip.charAt(i) <= '9') && ip.charAt(i) != '.' ){
                return false;
            }
        }
        String[] arr = ip.split("\\.");
        if(arr.length != 4){
            return false;
        }
        for(int i=0;i<4;i++){
            String s = arr[i];
            if(s.length() > 3 || s.length() <= 0){
                return false;
            }
            if(s.length() > 1 && s.charAt(0) == '0'){
                return false;
            }
            Integer val = Integer.valueOf(s);
            if(val < 0 || val > 255){
                return false;
            }
        }
        return true;
    }

    public String isIP6Arr(String[] IP6Arr){
        for(String ip : IP6Arr){
            if(ip.length() > 4 || ip.length() <= 0){
                return "Neither";
            }
            for(int i = 0 ;i < ip.length();++i){
                char c = ip.charAt(i);
                if(!Character.isDigit(c) && !( 'a' <= c && c <= 'f') && !('A' <= c && c <= 'F')){
                    return "Neither";
                }
            }
        }
        return "IPv6";
    }

}
```
