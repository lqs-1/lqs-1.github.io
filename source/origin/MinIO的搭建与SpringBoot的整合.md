---
title: MinIO的搭建与SpringBoot的整合
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-09 17:33:10
password:
summary: MinIO的搭建与SpringBoot的整合
tags: storage
categories: minio
---



# MinIO的搭建（单机部署）

## 二进制方式：

```
mkdir /mnt/data
```

```
sudo chmod -R 775 /mnt/data
```

```
wget -P /mnt/data/ https://dl.min.io/server/minio/release/linux-amd64/minio
```

 将下载的文件变成可执行

```
chmod +x /mnt/data/minio
```

启动：         (不设置密码就是默认 minioadmin/minioadmin)

```
/mnt/data/minio server /mnt/data
```

后台指定9000端口启动：

```
nohup /mnt/data/minio server  --address :9000 --console-address :9001 /mnt/data/ > /mnt/data/minio.log &
```

## docker方式：

```
docker run -d \
  -p 8900:9000 \
  -p 8901:9001 \
  --name minio2 \
  -v ~/var/local/environment/data:/data \
  -e "MINIO_ROOT_USER=root" \
  -e "MINIO_ROOT_PASSWORD=minio123456" \
  minio/minio server /data --console-address ":8901"
```

# spring boot的整合

pom加入minio依赖

```
 <dependency>
      <groupId>io.minio</groupId>
      <artifactId>minio</artifactId>
      <version>8.4.5</version>
</dependency>
 
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.10.0</version>
</dependency>
```

yml文件加入配置

```
minio:
  endpoint: http:/81.68.119.53/:9000
  accessKey: masterminio
  secretKey: masterminio
  bucketName: wmy
```

后端代码编写

```
@Configuration
@Slf4j
public class MinioClientConfig {
 
    @Value("${minio.endpoint}")
    private String endpoint;
 
    @Value("${minio.accessKey}")
    private String accessKey;
 
    @Value("${minio.secretKey}")
    private String secretKey;
 
    @Value("${minio.bucketName}")
    private String bucketName;
 
    @Bean
    public MinioClient minioClient() {
        log.info("endpoint={}  ,  accessKey={}  ,secretKey={}", endpoint, accessKey, secretKey);
        return MinioClient.builder().credentials(accessKey, secretKey).endpoint(endpoint).build();
    }
}
```

上传文件工具类

```
@Component
public class MinioUtils {
 
    @Autowired
    private MinioClient minioClient;
 
 
    /**
     * 判断桶是否存在
     */
    @SneakyThrows(Exception.class)
    public boolean bucketExists(String bucketName) {
        return minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
    }
 
 
    /**
     * 创建桶
     * @param bucketName
     * 获取全部的桶  minioClient.listBuckets();
     */
    @SneakyThrows(Exception.class)
    public void createBucket(String bucketName) {
        if (!minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build())) {
            minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
        }
    }
 
 
    /**
     * 根据bucketName获取信息
     * @param bucketName bucket名称
     */
    @SneakyThrows(Exception.class)
    public Optional<Bucket> getBucket(String bucketName) {
        return minioClient.listBuckets().stream().filter(b -> b.name().equals(bucketName)).findFirst();
    }
 
    /**
     * 根据bucketName删除信息
     * @param bucketName bucket名称
     */
    @SneakyThrows(Exception.class)
    public  void removeBucket(String bucketName) {
        minioClient.removeBucket(RemoveBucketArgs.builder().bucket(bucketName).build());
    }
 
    /**
     * 获取文件流
     * @param bucketName bucket名称
     * @param objectName 文件名称
     * @return 二进制流
     */
    @SneakyThrows(Exception.class)
    public InputStream getObject(String bucketName, String objectName) {
        return minioClient.getObject(GetObjectArgs.builder().bucket(bucketName).object(objectName).build());
    }
 
 
    /**
     * 上传本地文件
     * @param bucketName 存储桶
     * @param objectName 对象名称
     * @param fileName   本地文件路径
     */
    @SneakyThrows(Exception.class)
    public  ObjectWriteResponse putObject(String bucketName, String objectName, String fileName) {
        if(!bucketExists(bucketName)){
           createBucket(bucketName);
        }
        return minioClient.uploadObject(UploadObjectArgs.builder().bucket(bucketName).object(objectName).filename(fileName).build());
    }
 
    /**
     * 通过流上传文件
     * @param bucketName  存储桶
     * @param objectName  文件对象
     * @param inputStream 文件流
     */
    @SneakyThrows(Exception.class)
    public  ObjectWriteResponse putObject(String bucketName, String objectName, InputStream inputStream) {
        if(!bucketExists(bucketName)){
            createBucket(bucketName);
        }
        return minioClient.putObject(PutObjectArgs.builder().bucket(bucketName).object(objectName).stream(inputStream, inputStream.available(), -1).build());
    }
 
 
    /**
     * 單個文件上傳遞
     * @param bucketName
     * @param multipartFile
     * @return
     */
    @SneakyThrows(Exception.class)
    public   String   uploadFileSingle(String  bucketName,MultipartFile multipartFile )  {
        if(!bucketExists(bucketName)){
            createBucket(bucketName);
        }
        String fileName = multipartFile.getOriginalFilename();
        String[] split = fileName.split("\\.");
        if (split.length > 1) {
            fileName = split[0] + "_" + System.currentTimeMillis() + "." + split[1];
        } else {
            fileName = fileName + System.currentTimeMillis();
        }
        InputStream in = null;
        try {
            in = multipartFile.getInputStream();
            minioClient.putObject(PutObjectArgs.builder().bucket(bucketName).object(fileName).stream(in, in.available(), -1).contentType(multipartFile.getContentType()).build());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return  getUploadObjectUrl(bucketName,fileName, 7*24*60*60);
    }
 
 
    /**
     * description: 上传文件
     * @param multipartFile
     * @return: java.lang.String
     */
    public List<String> uploadFileBatch(String  bucketName,MultipartFile[] multipartFile) {
        if(!bucketExists(bucketName)){
            createBucket(bucketName);
        }
        List<String> names = new ArrayList<>();
        for (MultipartFile file : multipartFile) {
            try {
                String fileName = file.getOriginalFilename();
                uploadFileSingle(bucketName,file);
                names.add(fileName);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return names;
    }
 
 
    /**
     * 获取文件外链
     * @param bucketName bucket名称
     * @param objectName 文件名称
     * @param expires    过期时间 <=7 秒级
     * @return url
     */
    @SneakyThrows(Exception.class)
    public  String getUploadObjectUrl(String bucketName, String objectName, Integer expires)  {
        return minioClient.getPresignedObjectUrl(GetPresignedObjectUrlArgs.builder().method(Method.PUT).bucket(bucketName).object(objectName).expiry(expires).build());
    }
 
    /**
     * 下载文件
     * bucketName:桶名
     * @param fileName: 文件名
     */
    @SneakyThrows(Exception.class)
    public  void download(String  bucketName,String fileName, HttpServletResponse response) {
        // 获取对象的元数据
        StatObjectResponse  stat = minioClient.statObject(StatObjectArgs.builder().bucket(bucketName).object(fileName).build());
        response.setContentType(stat.contentType());
        response.setCharacterEncoding("UTF-8");
        response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
        InputStream is = minioClient.getObject(GetObjectArgs.builder().bucket(bucketName).object(fileName).build());
        IOUtils.copy(is, response.getOutputStream());
        is.close();
    }
 
}
```

上传接口开发

```
    @SneakyThrows
    @PostMapping("/uploadFile")
    @ResponseBody
    public  String  uploadFile(@RequestParam("file")MultipartFile  multipartFile  ){
        log.info("uploadFile  ----   -----  start ");
        String imgPath = minioUtils.uploadFileSingle(bucketName, multipartFile);
        log.info("uploadFile  ----   -----  end {}",imgPath);
        return   imgPath;
    }
```

前端代码编写

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试Minio上传文件</title>
    <style>
        a.upload-img {
            width: 165px;
            display: inline-block;
            margin-bottom: 10px;
            text-align: center;
            height: 37px;
            line-height: 37px;
            font-size: 14px;
            color: #FFFFFF;
            background-color: #f38e81;
            border-radius: 3px;
            text-decoration: none;
            cursor: pointer;
            border: 0px #fff solid;
            box-shadow: 0px 0px 5px #B0B0B0;
        }
    </style>
</head>
<body>
 
   <div style="width: 100%;flex: auto">
      <form id="img-form">
          <a href="javascript:void(0)" class="upload-img"><label for="inputImage">上传图像</label> </a>
          <input  onchange="doUpload()" type="file" name="file" id="inputImage" accept="image/*"/>
      </form>
       <img  height="120px" style="margin-left: 20px" >
   </div>
 
   <script>
       let  doUpload=()=>{
           console.log("1---> start  upload ");
           let file =  document.getElementById("inputImage").files[0];
           if (/^image\/\w+$/.test(file.type)) {
               let form = document.getElementById("img-form");
               let formData = new FormData(form);
               let oReq = new XMLHttpRequest();
               oReq.open("POST", "http://127.0.0.1:9055/uploadFile", true);
               oReq.send(formData);
               oReq.onload = function(oEvent) {
                   if(oReq.status == 200) {
                       alert("Uploaded Success");
                       console.log("1---> end  upload ",oReq.response)
                   }
               }
           }
       }
 
   </script>
</body>
</html>
```

