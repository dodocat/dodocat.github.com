---
layout: post
title: "JAVA 中使用Json"
date: 2013-03-13 16:25
comments: true
categories: 
---

## JSON
<image src="http://www.json.org/img/json.png" alt="json logo" title="JSON LOGO" width="400"/>

我们使用 [org.json](http://json.org/java/) 的库， 
源码可以在 [github](https://github.com/douglascrockford/JSON-java) 上获得.

``` java
package com.test;

import java.text.ParseException;
import java.util.ArrayList;

import org.json.JSONArray;
import org.json.JSONObject;

public class Utili {

    public static void Json2Obj(String s) {
        JSONArray array;
        try {
            array = new JSONArray(s);
        } catch (ParseException e) {
            e.printStackTrace();
            return;
        }
        StringBuilder sb = new StringBuilder("\r\r----------------------\r");
        for (int i = 0; i < array.length(); i++) {
            JSONObject obj = array.getJSONObject(i);
            sb.append("id:").append(obj.getInt("id")).append("\r");
            sb.append("name:").append(obj.getString("name")).append("\r");
            sb.append("gender:").append(obj.getString("gender")).append("\r");
            sb.append("email:").append(obj.getString("email")).append("\r");
            sb.append("----------------------\r");
        }
        System.out.println(sb.toString());
    }

    public static String Obj2Json() {
        ArrayList<User> list = new ArrayList<User>();
        User bean1;
        for (int i = 0; i < 10; i++) {
            bean1 = new User();
            bean1.setId(i);
            bean1.setName("kilo_" + i);
            bean1.setEmail("kilonet@kilonet.cn");
            bean1.setGender("male");
            list.add(bean1);
        }

        JSONArray array = new JSONArray();
        for (User bean : list) {
            JSONObject obj = new JSONObject();

            try {
                obj.put("id", bean.getId());
                obj.put("name", bean.getName());
                obj.put("email", bean.getEmail());
                obj.put("gender", bean.getGender());
            } catch (Exception e) {
            }

            array.put(obj);
        }

        System.out.print(array.toString());
        return array.toString();

    }

    public static class User {
        private int id;
        private String name;
        private String email;
        private String gender;

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getEmail() {
            return email;
        }

        public void setEmail(String email) {
            this.email = email;
        }

        public String getGender() {
            return gender;
        }

        public void setGender(String gender) {
            this.gender = gender;
        }

    }
}
```


