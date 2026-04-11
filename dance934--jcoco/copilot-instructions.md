## jcoco

> - 請幫忙用 maven 建立一個 Java 8 的專案

# 目標: 
- 請幫忙用 maven 建立一個 Java 8 的專案
- 產生程式時，請一起產生單元測試(*.Test.java) & 整合測試(*.IT.java)
- 專案有多個模組分別為 module-a, module-b, module-c
- module-a有簡單的 controller 會顯示 hello world
- module-b & module-c 都有簡單的程式顯示歡迎詞



# 技術選用如下
Java 8
Maven
org.apache.maven.plugins:jacoco-maven-plugin: 0.8.12
org.apache.maven.plugins:maven-failsafe-plugin:3.5.3
org.apache.maven.plugins:maven-surefire-report-plugin:3.5.3


# jacoco 指令
java -jar .\lib\lib\jacococli.jar report .\target\jacoco-agg.exec --classfiles .\module-a\target\classes\ --classfiles .\module-b\target\classes\ --classfiles .\module-c\target\classes\ --html .\target\jacoco-html

# jacoco jar 
https://www.jacoco.org/jacoco/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dance934)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dance934)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
