---
layout: post
comments: true
title: "[WEB][GCP] Spring Boot에서 GCP Bucket에 파일 업로드하기"
subtitle: 
description: 
date: 2024-08-20
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][GCP] Spring Boot에서 GCP Bucket에 파일 업로드하기

### 문제 설명

사실 구현한지는 두달은 된 것 같은데, 프로젝트를 제대로 정리할 시간이 없다 보니까 이제서야 정리하게 되었다.
음, 일단 AWS S3보다 GCP 클라우드 버켓을 사용하는 경우가 더 적다 보니까, 인터넷에 잘 정리되어 있는 자료가 많지 않았다.
물론 클라우드 버켓이 S3보다 더 좋은 서비스를 제공하지는 않지만, AWS의 프리티어보다 GCP 프리티어의 제공량이 더 많기도 하고,
새 계정을 만들 경우에는 300$ 크레딧을 제공하기 때문에, 한번 새로운 계정을 만들어서 사용하는 것도 나쁘지 않다.

사실 GCP 버켓에 파일을 업로드하는 것은 AWS S3에 파일을 업로드하는 것과 크게 다르지 않다.

### GCPConfig

~~~ json

    dependencies {
    'implementation 'com.google.cloud:spring-cloud-gcp-starter-storage:5.3.0'
    }
    
~~~

일단 gradle에 위와 같은 의존성을 추가해 준다. GPT는 starter 의존성을 추가하는게 아니라.

~~~ json

implementation 'com.google.cloud:google-cloud-storage:latest_version'

~~~

이렇게 개별 의존성을 추가하는 식으로 조언해 주는데, 공식 문서에서는 starter 의존성을 추가하는 방향으로
설명하고 있어서, 해당 의존성을 추가했다.
그리고 GCP 버켓에 접속할 수 있도록 서비스 계정을 생성하고 인증 키를 만들어야 한다.

<img src="/img/posts/gcp/bucket/1.png" width="80%">

1. Google Cloud Console에 접속.
2. IAM & Admin > Service Accounts에서 새로운 서비스 계정을 생성.
3. 서비스 계정에 적절한 역할(예: Storage Admin 또는 Storage Object Admin)을 부여.
4. 서비스 계정의 키를 생성하고 JSON 파일을 다운로드.

### 환경변수 추가

그리고 해당 키를 스프링에서 사용할 수 있게 하기 위해서 해당 키 파일을 환경변수로 설정해주거나, 
직접 키 파일을 프로젝트 내에 삽입하는 방법이 있다. 물론 보안상으로 당연히 환경변수로 설정하는 것이 좋다.
mac 기준으로 자신의 .zshrc에 해당 키를 환경변수로 설정해주면 된다.

~~~ bash

sudo nano ~/.zshrc

// 해당 키를 환경변수로 설정(마지막줄에 추가해주면 된다)

export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your-service-account-file.json"

~~~

배포때도 해당 키를 환경변수로 설정해주면 되는데, 서버의 경우 대부분이 우분투이기 때문에 .bashrc에 해당 키를 설정해주면 된다.
그리고 로컬 환경에서 해당 키를 설정해 주었는데도 오류가 나는 경우가 있는데, 배포는 문제 없이 잘 작동하는데 로컬에서만 오류가 나는 경우가 있었다.
내 경우에는 IntelliJ에서 해당 키를 인식하지 못한게 그 이유였는데 이를 해결하기 위해서는 인텔리제이의 환경변수에 해당 키를 추가해주면 된다.

인텔리제이에서 해당 키를 직접 추가하는 방법으로는 

1. Run > Edit Configurations

<img src="/img/posts/gcp/bucket/2.png" width="80%">

2. Environment Variables에 해당 키를 추가해주면 된다.

<img src="/img/posts/gcp/bucket/3.png" width="80%">

### GcpConfig.java

~~~ java

package ac.su.kiosk.config;

import com.google.auth.oauth2.GoogleCredentials;
import com.google.cloud.storage.Storage;
import com.google.cloud.storage.StorageOptions;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.List;
import org.slf4j.Logger;

@Configuration
public class GCPConfig {
    private static final Logger logger = LoggerFactory.getLogger(GCPConfig.class);

    @Bean
    public Storage storage() throws IOException {
        String credentialsPath = System.getenv("GOOGLE_APPLICATION_CREDENTIALS");
        if (credentialsPath == null) {
            throw new IllegalStateException("GOOGLE_APPLICATION_CREDENTIALS environment variable is not set.");
        }

        logger.info("Using GOOGLE_APPLICATION_CREDENTIALS from: " + credentialsPath);

        try (FileInputStream credentialsStream = new FileInputStream(credentialsPath)) {
            GoogleCredentials credentials = GoogleCredentials.fromStream(credentialsStream)
                    .createScoped(List.of("https://www.googleapis.com/auth/cloud-platform"));
            return StorageOptions.newBuilder().setCredentials(credentials).build().getService();
        } catch (IOException e) {
            logger.error("Failed to load GoogleCredentials from path: " + credentialsPath, e);
            throw e;
        }
    }
}

~~~

이렇게 GCPConfig를 만들어서 Storage를 Bean으로 등록해주면 된다. 
이렇게 하면 GCP 버켓에 접근할 수 있는 Storage 객체를 사용할 수 있다.

* 파일 입력 스트림 생성
    FileInputStream credentialsStream = new FileInputStream(credentialsPath):
    지정된 경로의 인증 파일(JSON)을 읽기 위해 파일 입력 스트림을 생성.
    try-with-resources 문을 사용하여 스트림이 자동으로 닫히도록 함.
* GoogleCredentials 객체 생성 및 스코프 설정
    GoogleCredentials.fromStream(credentialsStream):
    파일 입력 스트림에서 인증 정보를 읽어와 GoogleCredentials 객체를 생성.
    .createScoped(List.of("https://www.googleapis.com/auth/cloud-platform")):
    필요한 OAuth 2.0 스코프를 설정.
    여기서는 cloud-platform 스코프를 사용하여 GCP의 모든 리소스에 대한 액세스 권한을 부여.
    필요에 따라 더 제한적인 스코프를 설정할 수 있음.
    https://www.googleapis.com/auth/devstorage.read_only: 읽기 전용 액세스
    
    https://www.googleapis.com/auth/devstorage.read_write: 읽기 및 쓰기 액세스
    
    https://www.googleapis.com/auth/devstorage.full_control: 완전한 제어 권한

* Storage 객체 생성
    StorageOptions.newBuilder().setCredentials(credentials).build().getService():
    StorageOptions 빌더를 사용하여 GoogleCredentials를 설정하고 Storage 서비스 객체를 생성.
    이렇게 생성된 Storage 객체를 반환하여 애플리케이션 내에서 GCS와 상호 작용할 수 있도록 함.


### StorageService.java

~~~ java

import com.google.cloud.storage.BlobInfo;
import com.google.cloud.storage.Storage;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RequiredArgsConstructor
@Service
public class StorageService {
    private final Storage storage;
    
    // application.properties에 설정한 버킷 이름을 가져옴
    @Value("${spring.cloud.gcp.storage.bucket-name}")
    private String bucketName;
    public String uploadFile(MultipartFile file) throws IOException {
        String blobName = file.getOriginalFilename();
        BlobInfo blobInfo = storage.create(
                BlobInfo.newBuilder(bucketName, blobName).build(),
                file.getBytes()
        );
        return blobInfo.getMediaLink();
    }
}

~~~

uploadFile 메소드는 업로드된 파일의 URL을 반환한다. 나는 application.properties에 버킷 이름을 설정했는데, 
문자열로 직접 설정해도 상관 없다.

### StorageController.java

~~~ java

    @RestController
    @PostMapping("/upload_test")
    public ResponseEntity<String> restImage(@RequestPart("file") MultipartFile file) throws IOException {
        try {
            String message = storageService.uploadFile(file);
            TestEntity test = new TestEntity();
            test.setTestString(message);
            testRepo.save(test);
            return new ResponseEntity<>(HttpStatus.CREATED);
        } catch (IOException e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    
~~~

이렇게 파일을 업로드하는 컨트롤러를 만들어주면 된다. 업로드된 파일의 URL을 반환하고, 해당 URL을 DB에 저장하는 식으로 구현했다.
그리고 해당 컨트롤러는 RestController인데 파일을 업로드 하는 경우에, @RequestPart를 사용해야 한다.
이 부분에서 트러블슈팅에 시간이 많이 걸렸는데 @RequestBody나 @RequestParam을 사용하면 파일을 업로드할 수 없다.

@RequestPart와 @RequestParam, @RequestBody 가 정확히 뭔지 모르고 막연하게 사용하다가 이런 문제가 생겼는데, 이번에
확실히 정리하게 되어서 오히려 좋은 기회가 되었다고 생각한다.


### RequestPart와 RequestParam, RequestBody

#### @RequestPart에 대한 자세한 설명

@RequestPart는 Spring MVC에서 멀티파트 요청(multipart request)에서 특정한 파트를 매핑하기 위해 사용되는 어노테이션이다. 
주로 파일 업로드 또는 복합 데이터(파일 + JSON 객체 등)를 처리할 때 사용되는데, 다음과 같은 시나리오에서 사용된다.

1. 파일 업로드 시, 파일을 받기 위해 사용.
2. 파일과 함께 전송되는 다른 데이터를 받기 위해 사용.
3. JSON 데이터와 파일을 함께 처리할 때 사용.

#### 주요 특징
1. 다양한 데이터 타입 처리: @RequestPart는 MultipartFile뿐만 아니라, 문자열, JSON 객체, POJO 등 다양한 타입을 처리할 수 있다. 
2. 멀티파트 요청과 JSON 객체: 파일 외에도 JSON 객체를 함께 전송할 수 있으며, 이를 각각의 파트로 분리하여 처리할 수 있다.

3. JSON 파싱 및 파일 업로드
   @RequestPart는 JSON 데이터를 자동으로 파싱하여 POJO로 변환할 수 있으며, 동시에 파일 업로드도 처리할 수 있다. 이러한 기능은 파일과 관련된 메타데이터를 함께 처리해야 할 때 매우 유용하다.

4. Content-Type 헤더 요구사항
   @RequestPart: multipart/form-data로 인코딩된 요청에서 특정 파트를 처리하는 데 적합하며, JSON 객체를 포함하는 멀티파트 요청의 **Content-Type**이 multipart/form-data로 지정되어야 한다.
   @RequestParam: application/x-www-form-urlencoded와 같은 단순 폼 데이터에 적합하며, JSON 데이터를 처리하기에는 제한적.

#### @RequestBody와 @RequestPart의 차이점
1. @RequestBody
   주요 목적: HTTP 요청의 본문(body)에 포함된 데이터를 직접 매핑하여 자바 객체로 변환하는 데 사용된다.

지원하는 Content-Type: @RequestBody는 주로 JSON, XML, plain text 등과 같은 데이터를 처리하는 데 사용된다. 이 경우 요청의 Content-Type이 application/json 또는 application/xml 같은 형식이 된다.

사용 사례:

단순한 JSON 객체를 자바 객체로 매핑할 때 주로 사용됨.
예를 들어, 클라이언트가 JSON으로 인코딩된 데이터를 전송하고, 서버가 이를 자바 객체로 변환하여 처리하는 경우에 적합함.

