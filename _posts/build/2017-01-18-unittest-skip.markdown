---
layout: post
title:  "How to skip unit test when deploying with both jenkins and maven"
date:   2017-01-16 20:10:53 +0900
categories: maven, maven
---


# 1
<plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>2.19.1</version>
      <configuration>
          <skipTests>true</skipTests>
      </configuration>
</plugin>
