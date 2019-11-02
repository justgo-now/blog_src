---
title: Helm模板文件chart编写语法总结
date: 2019-11-02 10:14:06
top:
categories: 持续集成
tags:
- helm
- k8s
---

[helm-charts用户指南](https://whmzsu.github.io/helm-doc-zh-cn/)

## 开始

快速创建一个chart模板,`helm create mychart`,执行命令后本地生成一个mychart目录.

## chart目录结构

- Chart.yaml: 该chart的描述文件,包括ico地址,版本信息等
- vakues.yaml: 给模板文件使用的变量
- charts: 依赖其他包的charts文件
- requirements.yaml: 依赖的charts
- README.md: 开发人员自己阅读的文件
- templates: 存放k8s模板文件目录
  - NOTES.txt 说明文件,helm install之后展示给用户看的内容
  - deployment.yaml 创建k8s资源的yaml文件
  - _helpers.tpl: 下划线开头的文件,可以被其他模板引用.

一个最小的chart目录,只需要包含一个Chart.yaml,和templates目录下一个k8s资源文件.如:

```
    # mychart/Chart.yaml
    apiVersion: v1
    appVersion: 2.9.0
    version: 1.1.1

    # mychart/templates/configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mychart-configmap
    data:
      myvalue: "Hello World"
```

## helm模板语法

1. 模板引用方式
   ```
{{ .Release.Name }}
   ```
通过双括号注入,小数点开头表示从最顶层命名空间引用.

2. helm内置对象
> Release, release相关属性
> Chart, Chart.yaml文件中定义的内容
> Values, values.yaml文件中定义的内容

3. 模板中使用管道

   ```
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
 myvalue: "Hello World"
 drink: {{ .Values.favorite.drink | repeat 5 | quote }}
 food: {{ .Values.favorite.food | upper | quote }}
   ```

4. if语句

   ```text
   {{ if PIPELINE }}
   # Do something
   {{ else if OTHER PIPELINE }}
   # Do something else
   {{ else }}
   # Default case
   {{ end }}
   ```

5. 操作符, and/eq/or/not

   ```text
   {{/* include the body of this if statement when the variable .Values.fooString exists and is set to "foo" */}}
   {{ if and .Values.fooString (eq .Values.fooString "foo") }}
       {{ ... }}
   {{ end }}
   
   {{/* do not include the body of this if statement because unset variables evaluate to false and .Values.setVariable was negated with the not function. */}}
   {{ if or .Values.anUnsetVariable (not .Values.aSetVariable) }}
      {{ ... }}
   {{ end }}
   ```

6. 控制语句块在渲染后生成模板会多出空行,需要使用

   ```
   {{- if ...}}
   ```

   的方式消除此空行.如:

   ```text
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: {{ .Release.Name }}-configmap
   data:
     myvalue: "Hello World"
     {{- if eq .Values.favorite.drink "coffee"}}
     mug: true
     {{- end}}
   ```

7. 引入相对命名空间,with命令:

   ```text
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: {{ .Release.Name }}-configmap
   data:
     myvalue: "Hello World"
     {{- with .Values.favorite }}
     drink: {{ .drink | default "tea" | quote }}
     food: {{ .food | upper | quote }}
     {{- end }}
   ```

8. range命令实现循环,如:

   ```text
   # values.yaml
   favorite:
     drink: coffee
     food: pizza
   pizzaToppings:
     - mushrooms
     - cheese
     - peppers
     - onions
   
   #configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: {{ .Release.Name }}-configmap
   data:
     myvalue: "Hello World"
     toppings: |-
       {{- range .Values.pizzaToppings }}
       - {{ . }}
       # .表示range的命令空间下的取值
       {{- end }}
       {{- range $key, $val := .Values.favorite }}
       {{ $key }}: {{ $val | quote }}
       {{- end}} 
   ```

9. 变量赋值

   ```text
   ApiVersion: v1
   Kind: ConfigMap
   Metadata:
     name: {{ .Release.Name }}-configmap
   Data:
     myvalue: "Hello World"
     # 由于下方的with语句引入相对命令空间,无法通过.Release引入,提前定义relname变量
     {{- $relname := .Release.Name -}}
     {{- with .Values.favorite }}
     food: {{ .food }}
     release: {{ $relname }}
     # 或者可以使用$符号,引入全局命名空间
     release: {{ $.Release.Name }}
     {{- end }}
   ```

10. 公共模板,define定义,template引入,在templates目录中默认下划线_开头的文件为公共模板(_helpers.tpl)

    ```text
    # _helpers.tpl文件
    {{- define "mychart.labels" }}
      labels:
        generator: helm
        date: {{ now | htmlDate }}
    {{- end }}
    
    # configmap.yaml文件
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ .Release.Name }}-configmap
      {{- template "mychart.labels" }}
    data:
      myvalue: "Hello World"
    ```

11. template语句的升级版本include,template是语句无法在后面接管道符来对引入变量做定义,
    include实现了此功能.

    ```text
    # _helpers.tpl文件
    {{- define "mychart.app" -}}
    app_name: {{ .Chart.Name }}
    app_version: "{{ .Chart.Version }}+{{ .Release.Time.Seconds }}"
    {{- end -}}
    
    # configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ .Release.Name }}-configmap
      labels:
        {{- include "mychart.app" . | nindent 4 }}
    data:
      myvalue: "Hello World"
      {{- range $key, $val := .Values.favorite }}
      {{ $key }}: {{ $val | quote }}
      {{- end }}
      {{- include "mychart.app" . | nindent 2 }}
    
    # 如果使用template只能手动空格,不能使用管道后的nindent函数来做缩进
    ```

12. 一个坑
> `helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null<br/>`

livenessProbe在values.yaml中定义了httpGet,需要手动设置为null,然后设置exec的探针.