---
cover: https://image.acg.lol/file/2025/11/09/reimu-2.jpg
title: Commons IO 和 Commons FileUpload 文件上传下载实现
date: 2025-10-15 18:00:00
categories: Web开发
excerpt: 通过传统的jsp+servlet+maven项目，来实现一个本地文件上传下载的功能，通过两个jar包来实现
---

## 一、环境准备和依赖配置

通过传统的jsp+servlet+maven项目，来实现一个本地文件上传下载的功能，通过两个jar包来实现。

### Maven 依赖
```xml
<dependencies>
    <!-- Commons FileUpload -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.5</version>
    </dependency>
    
    <!-- Commons IO -->
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.11.0</version>
    </dependency>
    
    <!-- Servlet API -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### 项目结构
```
project/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/
│       │       ├── servlet/
│       │       │   ├── FileUploadServlet.java
│       │       │   └── FileDownloadServlet.java
│       │       └── util/
│       │           └── FileUtil.java
│       └── webapp/
│           ├── WEB-INF/
│           │   └── web.xml
│           ├── index.html
│           ├── upload.jsp
│           └── download.jsp
└── pom.xml
```

## 二、完整实现代码

### 1. 前端页面 (index.html)
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文件上传下载系统</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            color: #333;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            margin-bottom: 40px;
            color: white;
        }
        
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .subtitle {
            font-size: 1.2rem;
            opacity: 0.9;
        }
        
        .card-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-bottom: 30px;
        }
        
        @media (max-width: 768px) {
            .card-container {
                grid-template-columns: 1fr;
            }
        }
        
        .card {
            background: white;
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            transition: transform 0.3s ease;
        }
        
        .card:hover {
            transform: translateY(-5px);
        }
        
        h2 {
            color: #4a5568;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 2px solid #e2e8f0;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .feature-list {
            list-style: none;
            margin: 20px 0;
        }
        
        .feature-item {
            padding: 10px 0;
            border-bottom: 1px solid #f1f1f1;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .feature-item:before {
            content: "✓";
            color: #48bb78;
            font-weight: bold;
        }
        
        .btn {
            display: inline-block;
            padding: 12px 30px;
            background: #667eea;
            color: white;
            text-decoration: none;
            border-radius: 8px;
            font-weight: 600;
            transition: all 0.3s;
            border: none;
            cursor: pointer;
            text-align: center;
        }
        
        .btn:hover {
            background: #5a6fd8;
            transform: translateY(-2px);
        }
        
        .btn-download {
            background: #48bb78;
        }
        
        .btn-download:hover {
            background: #38a169;
        }
        
        .upload-form {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        
        .form-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        
        label {
            font-weight: 600;
            color: #4a5568;
        }
        
        input[type="file"] {
            padding: 10px;
            border: 2px dashed #cbd5e0;
            border-radius: 8px;
            background: #f7fafc;
        }
        
        input[type="text"] {
            padding: 12px;
            border: 2px solid #e2e8f0;
            border-radius: 8px;
            font-size: 1rem;
        }
        
        .file-list {
            margin-top: 20px;
        }
        
        .file-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 15px;
            background: #f7fafc;
            border-radius: 8px;
            margin-bottom: 10px;
        }
        
        .file-info {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .file-icon {
            font-size: 1.5rem;
        }
        
        .file-name {
            font-weight: 500;
        }
        
        .file-size {
            color: #718096;
            font-size: 0.9rem;
        }
        
        .status-message {
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            text-align: center;
            display: none;
        }
        
        .status-success {
            background: #f0fff4;
            color: #38a169;
            border: 1px solid #c6f6d5;
        }
        
        .status-error {
            background: #fed7d7;
            color: #e53e3e;
            border: 1px solid #feb2b2;
        }
        
        .tech-info {
            background: white;
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            margin-top: 30px;
        }
        
        .tech-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .tech-card {
            background: #f7fafc;
            padding: 20px;
            border-radius: 10px;
            border-left: 4px solid #667eea;
        }
        
        .tech-title {
            font-weight: 600;
            margin-bottom: 10px;
            color: #4a5568;
        }
        
        .progress-bar {
            width: 100%;
            height: 6px;
            background: #e2e8f0;
            border-radius: 3px;
            margin: 10px 0;
            overflow: hidden;
            display: none;
        }
        
        .progress {
            height: 100%;
            background: #48bb78;
            width: 0%;
            transition: width 0.3s;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>文件上传下载系统</h1>
            <p class="subtitle">基于 Commons IO 和 Commons FileUpload 实现</p>
        </header>
        
        <div class="card-container">
            <div class="card">
                <h2>📤 文件上传</h2>
                <form class="upload-form" id="uploadForm" enctype="multipart/form-data">
                    <div class="form-group">
                        <label for="file">选择文件</label>
                        <input type="file" id="file" name="file" multiple required>
                    </div>
                    
                    <div class="form-group">
                        <label for="description">文件描述（可选）</label>
                        <input type="text" id="description" name="description" placeholder="输入文件描述...">
                    </div>
                    
                    <button type="submit" class="btn">上传文件</button>
                </form>
                
                <div class="progress-bar" id="progressBar">
                    <div class="progress" id="progress"></div>
                </div>
                
                <div id="uploadStatus" class="status-message"></div>
            </div>
            
            <div class="card">
                <h2>📥 文件下载</h2>
                <p>选择要下载的文件：</p>
                
                <div class="file-list" id="fileList">
                    <!-- 文件列表将通过JavaScript动态加载 -->
                    <div class="file-item">
                        <div class="file-info">
                            <span class="file-icon">📄</span>
                            <div>
                                <div class="file-name">示例文件.txt</div>
                                <div class="file-size">2.5 KB</div>
                            </div>
                        </div>
                        <a href="#" class="btn btn-download">下载</a>
                    </div>
                </div>
                
                <div id="downloadStatus" class="status-message"></div>
            </div>
        </div>
        
        <div class="tech-info">
            <h2>🛠 技术实现详情</h2>
            <div class="tech-grid">
                <div class="tech-card">
                    <div class="tech-title">Commons FileUpload</div>
                    <ul class="feature-list">
                        <li class="feature-item">处理 multipart/form-data 请求</li>
                        <li class="feature-item">支持大文件上传</li>
                        <li class="feature-item">内存和磁盘存储管理</li>
                        <li class="feature-item">文件大小限制控制</li>
                    </ul>
                </div>
                
                <div class="tech-card">
                    <div class="tech-title">Commons IO</div>
                    <ul class="feature-list">
                        <li class="feature-item">简化文件操作</li>
                        <li class="feature-item">流复制和关闭</li>
                        <li class="feature-item">文件大小计算</li>
                        <li class="feature-item">文件名处理</li>
                    </ul>
                </div>
                
                <div class="tech-card">
                    <div class="tech-title">核心特性</div>
                    <ul class="feature-list">
                        <li class="feature-item">多文件同时上传</li>
                        <li class="feature-item">上传进度显示</li>
                        <li class="feature-item">文件类型验证</li>
                        <li class="feature-item">安全下载</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 文件上传处理
        document.getElementById('uploadForm').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const formData = new FormData(this);
            const progressBar = document.getElementById('progressBar');
            const progress = document.getElementById('progress');
            const status = document.getElementById('uploadStatus');
            
            // 显示进度条
            progressBar.style.display = 'block';
            progress.style.width = '0%';
            
            const xhr = new XMLHttpRequest();
            
            // 上传进度事件
            xhr.upload.addEventListener('progress', function(e) {
                if (e.lengthComputable) {
                    const percentComplete = (e.loaded / e.total) * 100;
                    progress.style.width = percentComplete + '%';
                }
            });
            
            // 请求完成
            xhr.addEventListener('load', function() {
                progressBar.style.display = 'none';
                
                if (xhr.status === 200) {
                    const response = JSON.parse(xhr.responseText);
                    if (response.success) {
                        showMessage(status, '文件上传成功！', 'success');
                        document.getElementById('uploadForm').reset();
                        loadFileList(); // 刷新文件列表
                    } else {
                        showMessage(status, response.message || '上传失败', 'error');
                    }
                } else {
                    showMessage(status, '上传失败，服务器错误', 'error');
                }
            });
            
            // 请求错误
            xhr.addEventListener('error', function() {
                progressBar.style.display = 'none';
                showMessage(status, '上传失败，网络错误', 'error');
            });
            
            xhr.open('POST', '/upload');
            xhr.send(formData);
        });
        
        // 显示状态消息
        function showMessage(element, message, type) {
            element.textContent = message;
            element.className = `status-message status-${type}`;
            element.style.display = 'block';
            
            setTimeout(() => {
                element.style.display = 'none';
            }, 5000);
        }
        
        // 加载文件列表
        function loadFileList() {
            // 模拟从服务器获取文件列表
            const fileList = [
                { name: 'document.pdf', size: '2.3 MB', url: '/download?file=document.pdf' },
                { name: 'image.jpg', size: '1.5 MB', url: '/download?file=image.jpg' },
                { name: 'data.xlsx', size: '456 KB', url: '/download?file=data.xlsx' }
            ];
            
            const fileListContainer = document.getElementById('fileList');
            fileListContainer.innerHTML = '';
            
            fileList.forEach(file => {
                const fileItem = document.createElement('div');
                fileItem.className = 'file-item';
                fileItem.innerHTML = `
                    <div class="file-info">
                        <span class="file-icon">${getFileIcon(file.name)}</span>
                        <div>
                            <div class="file-name">${file.name}</div>
                            <div class="file-size">${file.size}</div>
                        </div>
                    </div>
                    <a href="${file.url}" class="btn btn-download">下载</a>
                `;
                fileListContainer.appendChild(fileItem);
            });
        }
        
        // 根据文件名获取图标
        function getFileIcon(filename) {
            const ext = filename.split('.').pop().toLowerCase();
            const icons = {
                'pdf': '📕',
                'jpg': '🖼️',
                'jpeg': '🖼️',
                'png': '🖼️',
                'gif': '🖼️',
                'doc': '📄',
                'docx': '📄',
                'xls': '📊',
                'xlsx': '📊',
                'zip': '📦',
                'rar': '📦',
                'txt': '📝',
                'mp4': '🎬',
                'mp3': '🎵'
            };
            return icons[ext] || '📄';
        }
        
        // 页面加载时初始化
        document.addEventListener('DOMContentLoaded', function() {
            loadFileList();
        });
    </script>
</body>
</html>
```

### 2. 文件上传 Servlet (FileUploadServlet.java)
```java
package com.example.servlet;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;
import org.apache.commons.io.FilenameUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;
import java.util.UUID;

@WebServlet("/upload")
public class FileUploadServlet extends HttpServlet {
    
    // 上传配置
    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
    private static final long MAX_REQUEST_SIZE = 20 * 1024 * 1024; // 20MB
    private static final String UPLOAD_DIRECTORY = "uploads";
    
    // 允许的文件类型
    private static final String[] ALLOWED_EXTENSIONS = {
        "txt", "pdf", "doc", "docx", "xls", "xlsx", "jpg", "jpeg", "png", "gif"
    };
    
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        PrintWriter out = response.getWriter();
        
        // 检查请求是否包含文件上传
        if (!ServletFileUpload.isMultipartContent(request)) {
            sendErrorResponse(out, "请求必须包含文件上传");
            return;
        }
        
        // 配置上传参数
        DiskFileItemFactory factory = new DiskFileItemFactory();
        
        // 设置内存存储阈值 - 超过后将产生临时文件并存储于临时目录中
        factory.setSizeThreshold(1024 * 1024); // 1MB
        
        // 设置临时存储目录
        File tempDir = new File(System.getProperty("java.io.tmpdir"));
        factory.setRepository(tempDir);
        
        ServletFileUpload upload = new ServletFileUpload(factory);
        
        // 设置最大文件上传值
        upload.setFileSizeMax(MAX_FILE_SIZE);
        
        // 设置最大请求值（包含文件和表单数据）
        upload.setSizeMax(MAX_REQUEST_SIZE);
        
        // 构造临时路径来存储上传的文件
        String uploadPath = getServletContext().getRealPath("") + File.separator + UPLOAD_DIRECTORY;
        
        // 如果目录不存在则创建
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists()) {
            uploadDir.mkdirs();
        }
        
        try {
            // 解析请求的内容提取文件数据
            List<FileItem> formItems = upload.parseRequest(request);
            
            if (formItems != null && formItems.size() > 0) {
                String description = null;
                String fileName = null;
                
                // 迭代表单数据
                for (FileItem item : formItems) {
                    if (item.isFormField()) {
                        // 处理普通表单字段
                        String fieldName = item.getFieldName();
                        String fieldValue = item.getString("UTF-8");
                        
                        if ("description".equals(fieldName)) {
                            description = fieldValue;
                        }
                    } else {
                        // 处理上传文件
                        String originalFileName = new File(item.getName()).getName();
                        
                        if (originalFileName != null && !originalFileName.isEmpty()) {
                            // 验证文件类型
                            if (!isAllowedFileType(originalFileName)) {
                                sendErrorResponse(out, "不支持的文件类型: " + originalFileName);
                                return;
                            }
                            
                            // 生成唯一文件名防止覆盖
                            String fileExtension = FilenameUtils.getExtension(originalFileName);
                            String uniqueFileName = UUID.randomUUID().toString() + "." + fileExtension;
                            
                            fileName = uniqueFileName;
                            String filePath = uploadPath + File.separator + uniqueFileName;
                            File storeFile = new File(filePath);
                            
                            // 保存文件到磁盘
                            item.write(storeFile);
                            
                            // 记录文件信息到数据库或文件（这里简化为打印日志）
                            System.out.println("文件上传成功: " + originalFileName + 
                                " -> " + uniqueFileName + 
                                "，描述: " + description);
                        }
                    }
                }
                
                // 返回成功响应
                out.print("{\"success\": true, \"message\": \"文件上传成功\", \"fileName\": \"" + fileName + "\"}");
            } else {
                sendErrorResponse(out, "没有找到上传的文件");
            }
        } catch (FileUploadException e) {
            sendErrorResponse(out, "文件上传失败: " + e.getMessage());
        } catch (Exception e) {
            sendErrorResponse(out, "服务器错误: " + e.getMessage());
        }
    }
    
    // 验证文件类型是否允许
    private boolean isAllowedFileType(String fileName) {
        String extension = FilenameUtils.getExtension(fileName).toLowerCase();
        for (String allowedExt : ALLOWED_EXTENSIONS) {
            if (allowedExt.equals(extension)) {
                return true;
            }
        }
        return false;
    }
    
    // 发送错误响应
    private void sendErrorResponse(PrintWriter out, String message) {
        out.print("{\"success\": false, \"message\": \"" + message + "\"}");
    }
}
```

### 3. 文件下载 Servlet (FileDownloadServlet.java)
```java
package com.example.servlet;

import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.IOUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

@WebServlet("/download")
public class FileDownloadServlet extends HttpServlet {
    
    private static final String UPLOAD_DIRECTORY = "uploads";
    
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        String fileName = request.getParameter("file");
        
        if (fileName == null || fileName.isEmpty()) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "文件名不能为空");
            return;
        }
        
        // 安全验证：防止路径遍历攻击
        if (fileName.contains("..") || fileName.contains("/") || fileName.contains("\\")) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "无效的文件名");
            return;
        }
        
        String uploadPath = getServletContext().getRealPath("") + File.separator + UPLOAD_DIRECTORY;
        File file = new File(uploadPath + File.separator + fileName);
        
        // 检查文件是否存在
        if (!file.exists()) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "文件不存在: " + fileName);
            return;
        }
        
        // 设置响应头
        String mimeType = getServletContext().getMimeType(file.getAbsolutePath());
        if (mimeType == null) {
            mimeType = "application/octet-stream";
        }
        
        response.setContentType(mimeType);
        response.setContentLength((int) file.length());
        
        // 设置下载头信息
        String encodedFileName = URLEncoder.encode(fileName, StandardCharsets.UTF_8.toString())
                .replaceAll("\\+", "%20");
        String disposition = "attachment; filename=\"" + encodedFileName + "\"";
        response.setHeader("Content-Disposition", disposition);
        
        // 使用Commons IO进行文件流复制
        try (FileInputStream inputStream = new FileInputStream(file);
             OutputStream outputStream = response.getOutputStream()) {
            
            IOUtils.copy(inputStream, outputStream);
            outputStream.flush();
            
            // 记录下载日志
            System.out.println("文件下载成功: " + fileName + 
                "，IP: " + request.getRemoteAddr());
                
        } catch (IOException e) {
            System.err.println("文件下载失败: " + fileName + " - " + e.getMessage());
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, "文件下载失败");
        }
    }
}
```

### 4. 文件工具类 (FileUtil.java)
```java
package com.example.util;

import org.apache.commons.io.FileUtils;
import org.apache.commons.io.FilenameUtils;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

public class FileUtil {
    
    /**
     * 获取上传目录中的所有文件信息
     */
    public static List<FileInfo> getUploadedFiles(String uploadPath) throws IOException {
        List<FileInfo> fileList = new ArrayList<>();
        
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists() || !uploadDir.isDirectory()) {
            return fileList;
        }
        
        File[] files = uploadDir.listFiles();
        if (files != null) {
            for (File file : files) {
                if (file.isFile()) {
                    FileInfo fileInfo = new FileInfo();
                    fileInfo.setName(file.getName());
                    fileInfo.setSize(FileUtils.byteCountToDisplaySize(file.length()));
                    fileInfo.setPath(file.getAbsolutePath());
                    fileInfo.setExtension(FilenameUtils.getExtension(file.getName()));
                    fileInfo.setLastModified(file.lastModified());
                    fileList.add(fileInfo);
                }
            }
        }
        
        return fileList;
    }
    
    /**
     * 删除上传的文件
     */
    public static boolean deleteFile(String filePath) {
        try {
            File file = new File(filePath);
            return FileUtils.deleteQuietly(file);
        } catch (Exception e) {
            System.err.println("删除文件失败: " + filePath + " - " + e.getMessage());
            return false;
        }
    }
    
    /**
     * 获取文件大小（人类可读格式）
     */
    public static String getHumanReadableSize(File file) {
        return FileUtils.byteCountToDisplaySize(file.length());
    }
    
    /**
     * 清理临时文件
     */
    public static void cleanupTempFiles(String tempDirPath, long maxAgeMillis) {
        File tempDir = new File(tempDirPath);
        if (!tempDir.exists()) {
            return;
        }
        
        File[] tempFiles = tempDir.listFiles();
        if (tempFiles != null) {
            long currentTime = System.currentTimeMillis();
            for (File tempFile : tempFiles) {
                if (currentTime - tempFile.lastModified() > maxAgeMillis) {
                    FileUtils.deleteQuietly(tempFile);
                }
            }
        }
    }
    
    /**
     * 文件信息类
     */
    public static class FileInfo {
        private String name;
        private String size;
        private String path;
        private String extension;
        private long lastModified;
        
        // getters and setters
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        
        public String getSize() { return size; }
        public void setSize(String size) { this.size = size; }
        
        public String getPath() { return path; }
        public void setPath(String path) { this.path = path; }
        
        public String getExtension() { return extension; }
        public void setExtension(String extension) { this.extension = extension; }
        
        public long getLastModified() { return lastModified; }
        public void setLastModified(long lastModified) { this.lastModified = lastModified; }
    }
}
```

### 5. Web 配置 (web.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <display-name>File Upload Download System</display-name>
    
    <!-- 文件上传配置 -->
    <servlet>
        <servlet-name>FileUploadServlet</servlet-name>
        <servlet-class>com.example.servlet.FileUploadServlet</servlet-class>
        <multipart-config>
            <max-file-size>10485760</max-file-size>      <!-- 10MB -->
            <max-request-size>20971520</max-request-size> <!-- 20MB -->
            <file-size-threshold>1048576</file-size-threshold> <!-- 1MB -->
        </multipart-config>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>FileUploadServlet</servlet-name>
        <url-pattern>/upload</url-pattern>
    </servlet-mapping>
    
    <!-- 文件下载配置 -->
    <servlet>
        <servlet-name>FileDownloadServlet</servlet-name>
        <servlet-class>com.example.servlet.FileDownloadServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>FileDownloadServlet</servlet-name>
        <url-pattern>/download</url-pattern>
    </servlet-mapping>
    
    <!-- 欢迎页面 -->
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>
    
</web-app>
```

## 三、核心特性说明

### Commons FileUpload 主要功能：
1. **Multipart 请求解析** - 自动解析 multipart/form-data 格式
2. **内存管理** - 智能管理内存和磁盘存储
3. **大小限制** - 支持文件大小和请求大小限制
4. **进度监听** - 可监听上传进度（需要额外配置）

### Commons IO 主要功能：
1. **流操作** - 简化的流复制和关闭操作
2. **文件工具** - 文件大小计算、类型判断等
3. **文件名处理** - 安全的文件名和路径操作
4. **工具方法** - 提供各种文件操作的工具方法

## 四、安全考虑

1. **文件类型验证** - 限制可上传的文件类型
2. **文件名安全** - 防止路径遍历攻击
3. **大小限制** - 防止超大文件上传
4. **病毒扫描** - 生产环境应集成病毒扫描

这个完整的实现展示了如何使用 Commons IO 和 Commons FileUpload 来构建一个功能完善的文件上传下载系统！