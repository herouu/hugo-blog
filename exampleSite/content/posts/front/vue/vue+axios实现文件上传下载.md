---
title: "Vue+axios实现文件上传下载"
subtitle: ""
date: 2020-07-23T20:56:19+08:00
lastmod: 2020-07-23T20:56:19+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ['Vue']
categories: ['Vue']

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

这里记录vue+axios上传文件和下载文件的实现，方便以后cp

<!--more-->

### 前端

```vue
<template>
  <div class="index-container">
    <div>
      <el-form>
        <el-form-item label="文件上传：">
          <el-upload
            class="upload-demo"
            action="/api/file/upload"
            :multiple="false"
            :limit="1"
            :http-request="fileUpload"
            :show-file-list="false"
          >
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
        <el-form-item label="文件下载：">
          <el-button size="small" type="primary" @click="download"
            >点击下载
          </el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import { upload, download } from "@/api/upload";
import downloadFile from "@/utils/file";
import axios from "axios";
export default {
  name: "Index",
  components: {},
  data() {
    return {
      nodeEnv: process.env.NODE_ENV,
    };
  },
  created() {},
  mounted() {},
  methods: {
    //文件下载
    download() {
      download().then((res) => {
        downloadFile(res);
      });
    },
    //文件上传
    fileUpload(fileObj) {
      let form = new FormData();
      form.append("file", fileObj.file);
      upload(form)
        .then((res) => {
          this.$message({ message: res.data, type: "success" });
        })
        .catch();
    },
  },
};
</script>

<style lang="scss" scoped>
.index-container {
  ::v-deep {
    .el-card__body {
      > div {
        min-height: 70vh;
        padding: 20px;

        > ul {
          > li {
            line-height: 30px;
          }
        }

        > img {
          display: block;
          margin: 40px auto;
          border: 1px solid #dedede;
        }
      }
    }
  }
}
</style>

```

### 封装工具类file.js

```js
export default function downloadFile(reponse) {
  const fileName = reponse.headers["content-disposition"]
    .split(";")[1]
    .split("fileName=")[1];
  var link = document.createElement("a");
  link.href = window.URL.createObjectURL(new Blob([reponse.data]));
  link.download = fileName;
  link.click();
  window.URL.revokeObjectURL(link.href);
}

```

### 请求api

```js
import request from "@/utils/request";

export function upload(formData) {
  return request({
    url: "/file/upload",
    method: "post",
    headers: { "Content-type": "multipart/form-data" },
    data: formData,
  });
}

export function download() {
  return request({
    url: "/file/download",
    method: "get",
    
    responseType: "blob",
  });
}

```



### 后端提供接口

```java
package top.alertcode.adelina.modules.system.controller;

import lombok.SneakyThrows;
import org.apache.commons.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import top.alertcode.adelina.common.annotations.Resources;
import top.alertcode.adelina.framework.responses.R;
import top.alertcode.adelina.modules.system.model.enums.AuthTypeEnum;

import javax.servlet.http.HttpServletResponse;

@RestController
@RequestMapping("/file")
public class FileController {

    @Autowired
    HttpServletResponse response;


    @SneakyThrows
    @PostMapping("/upload")
    @Resources(auth = AuthTypeEnum.LOGIN)
    public R<String> upload(@RequestBody MultipartFile file) {
        byte[] bytes = file.getBytes();
        String s = Base64.encodeBase64String(bytes);
        return R.data(s);
    }


    @SneakyThrows
    @GetMapping("/download")
    @Resources(auth = AuthTypeEnum.OPEN)
    public void download() {
        byte[] bytes = "下载文件".getBytes();
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition", "attachment;fileName=" + "test.txt");
        response.getOutputStream().write(bytes);
    }
}
```
