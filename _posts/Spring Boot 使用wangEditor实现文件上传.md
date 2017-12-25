#Spring Boot 使用wangEditor实现图片上传
我们在进行web开发时经常会涉及到图片上传的问题，特别是利用富文本编辑器进行图片上传，业界也有很多的富文本编辑器，有ueditor、fckeditor、wangEditor。这里我们选择wangEditor作为示例，因为wangEditor是一个轻量级的富文本编辑器已经可以满足我们大部分的需求。这里结合spring boot实现图片上传，有了图片上传其实文件的上传也是一回事了。
#wangEditor
目前wangEditor已经时v3版本了，这里给出链接[wangEditor](https://www.kancloud.cn/wangfupeng/wangeditor3/332599)，上面有详细的介绍。
#spring boot
我们依然采用spring boot作为服务端开发
首先我们在resources目录添加了一个configure.properties文件里面只有一行，定义了我们上传文件存储的目录
>\#用于存储系统中可能涉及到的各种配置信息
>material.upload_dir=/home/jacky/upload-dir

与之对应的我们构建一个类来加载配置文件
```
package com.material.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "material")
public class MaterialConfig {
    private String upload_dir;

    public String getUpload_dir() {
        return upload_dir;
    }

    public void setUpload_dir(String upload_dir) {
        this.upload_dir = upload_dir;
    }
}

```
我们利用spring boot官方提供的文件上传示例来进行文件上传
首先构建文件上传的业务类
```
package com.material.common.service;

import org.springframework.core.io.Resource;
import org.springframework.web.multipart.MultipartFile;

import java.nio.file.Path;
import java.util.stream.Stream;

public interface StorageService {

    void init();

    String store(MultipartFile file);

    Stream<Path> loadAll();

    Path load(String filename);

    Resource loadAsResource(String filename);

    void deleteAll();

}

package com.material.common.service;

import com.material.common.exception.StorageException;
import com.material.common.exception.StorageFileNotFoundException;
import com.material.config.MaterialConfig;
import com.material.util.RandomUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.stereotype.Service;
import org.springframework.util.FileSystemUtils;
import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.stream.Stream;

@Service
public class FileSystemStorageService implements StorageService {

    private final Path rootLocation;

    @Autowired
    public FileSystemStorageService(MaterialConfig materialConfig) {
        this.rootLocation = Paths.get(materialConfig.getUpload_dir());
    }

	//存储文件
    @Override
    public String store(MultipartFile file) {
        String filename = StringUtils.cleanPath(file.getOriginalFilename());
        try {
            if (file.isEmpty()) {
                throw new StorageException("Failed to store empty file " + filename);
            }
            if (filename.contains("..")) {
                // This is a security check
                throw new StorageException(
                        "Cannot store file with relative path outside current directory "
                                + filename);
            }
            String storeFileName = RandomUtil.getRandomFileName() + filename.substring(filename.indexOf("."), filename.length());
            Files.copy(file.getInputStream(), this.rootLocation.resolve(storeFileName),
                    StandardCopyOption.REPLACE_EXISTING);
            return storeFileName;
        } catch (IOException e) {
            throw new StorageException("Failed to store file " + filename, e);
        }
    }
	
    //加载所有文件的路径
    @Override
    public Stream<Path> loadAll() {
        try {
            return Files.walk(this.rootLocation, 1)
                    .filter(path -> !path.equals(this.rootLocation))
                    .map(path -> this.rootLocation.relativize(path));
        } catch (IOException e) {
            throw new StorageException("Failed to read stored files", e);
        }

    }

	//加载指定文件的路径
    @Override
    public Path load(String filename) {
        return rootLocation.resolve(filename);
    }

	//加载指定文件为资源用于返回客户端
    @Override
    public Resource loadAsResource(String filename) {
        try {
            Path file = load(filename);
            Resource resource = new UrlResource(file.toUri());
            if (resource.exists() || resource.isReadable()) {
                return resource;
            } else {
                throw new StorageFileNotFoundException(
                        "Could not read file: " + filename);

            }
        } catch (MalformedURLException e) {
            throw new StorageFileNotFoundException("Could not read file: " + filename, e);
        }
    }

    @Override
    public void deleteAll() {
        FileSystemUtils.deleteRecursively(rootLocation.toFile());
    }

    @Override
    public void init() {
        try {
            Files.createDirectories(rootLocation);
        } catch (IOException e) {
            throw new StorageException("Could not initialize storage", e);
        }
    }
}

```
这是spring boot官方提供业务类，下面我们来看controller，官方提供的controller用了lambda表达式
```
package com.material.common.controller;

import com.material.cltx.entity.SecondleveldescEntity;
import com.material.common.exception.StorageFileNotFoundException;
import com.material.common.service.StorageService;
import com.material.util.UploadResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.method.annotation.MvcUriComponentsBuilder;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.io.IOException;
import java.util.stream.Collectors;

@Controller
@RequestMapping("/upload")
public class FileUploadController {

    @Autowired
    private StorageService storageService;

	//加载所有已上传文件的路径并传递到页面
    @GetMapping("")
    public String listUploadedFiles(Model model) throws IOException {

        model.addAttribute("files", storageService.loadAll().map(
                path -> MvcUriComponentsBuilder.fromMethodName(FileUploadController.class,
                        "serveFile", path.getFileName().toString()).build().toString())
                .collect(Collectors.toList()));

        return "test/upload";
    }

	//显示指定的文件
    @GetMapping("/files/{filename:.+}")
    @ResponseBody
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

        Resource file = storageService.loadAsResource(filename);
        return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"" + file.getFilename() + "\"").body(file);
    }

	//文件上传
    @PostMapping("/")
    @ResponseBody
    public UploadResult handleFileUpload(@RequestParam("file") MultipartFile file,
                                         RedirectAttributes redirectAttributes) {
        String storeFileName = storageService.store(file);
        UploadResult result = new UploadResult();
        result.setErrno(0);
        result.getData().add(MvcUriComponentsBuilder.fromMethodName
                (FileUploadController.class, "serveFile", storeFileName).build().toString());
        return result;
    }
}
```
前台jsp页面，是的我们还是用的jsp/手动滑稽
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>文件上传页面</title>
</head>

<body>
<div>
    <p>已上传图片</p>
    <c:forEach items="${files }" var="file">
        <img src="${file}" width="500px">
        <p>${file}</p>
    </c:forEach>
</div>
<div >
    <div id="editor">
        <p>wangEditor 富文本编辑器</p>
    </div>
</div>
<script type="text/javascript" src="../../js/lib/wangEditor/wangEditor.min.js"></script>
<script type="text/javascript">
    var E = window.wangEditor
    var editor = new E('#editor')
    // 或者 var editor = new E( document.getElementById('#editor') )
    editor.customConfig.uploadImgServer = '/upload/';
    //设置上传的参数名
    editor.customConfig.uploadFileName = 'file';
    // 隐藏“网络图片”tab
    editor.customConfig.showLinkImg = false;
    // 将图片大小限制为 10M
    editor.customConfig.uploadImgMaxSize = 10 * 1024 * 1024;
    editor.create();
</script>
</body>
</html>
```
在jsp页面我们也能看到我们怎样通过wangEditor来构建一个富文本编辑器并实现图片上传，当然wangEditor提供了很多操作的方法：例如获取编辑器的值等，具体的参见官方手册
这样利用spring boot和wangEditor实现图片上传就完成了，大家可以自己测试