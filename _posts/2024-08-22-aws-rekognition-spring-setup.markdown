---
layout: post
comments: true
title: "[WEB][AWS] AWS Rekognition Face Detection Spring Boot 설정"
subtitle: AWS Rekognition API를 사용하여 Spring Boot에서 얼굴 인식 및 감정 분석을 구현.
description: AWS Rekognition API를 사용하여 Spring Boot에서 얼굴 인식을 구현.
date: 2024-08-22
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][AWS] AWS Rekognition Face Detection Spring Boot 설정

### 1. 서론

흔히 인터넷에서 많이 본 기능중에, 사진을 업로드하면 해당 사진 속의 얼굴을 인식하여 박스를 그리고, 얼굴의 감정 및 나이, 특징등을 
분석해서 보여주는 사진을 많이 보았을 것이다. 뜬금 없는 사진에서 역겨움이나 놀람, 행복 등으로 웃음을 주는 밈들을 생각해 보면 바로 떠올릴 수 있을 것이다.
...이런 식으로

<img src="/img/posts/aws_face_rekognition/1.jpeg" width="80%">

~~~ json

{
    "faceDetails": [
        {
            "boundingBox": {
                "width": 0.38674828,
                "height": 0.37040558,
                "left": 0.30599424,
                "top": 0.19806679
            },
            "ageRange": {
                "low": 20,
                "high": 26
            },
            "smile": {
                "value": false,
                "confidence": 99.86501
            },
            "eyeglasses": {
                "value": true,
                "confidence": 99.99978
            },
            "sunglasses": {
                "value": false,
                "confidence": 99.95743
            },
            "gender": {
                "value": "Male",
                "confidence": 57.465927
            },
            "beard": {
                "value": false,
                "confidence": 98.73258
            },
            "mustache": {
                "value": false,
                "confidence": 99.56685
            },
            "eyesOpen": {
                "value": true,
                "confidence": 99.998924
            },
            "mouthOpen": {
                "value": false,
                "confidence": 99.27673
            },
            "emotions": [
                {
                    "type": "CALM",
                    "confidence": 100.0
                },
                {
                    "type": "SAD",
                    "confidence": 0.009685755
                },
                {
                    "type": "CONFUSED",
                    "confidence": 0.0065863132
                },
                {
                    "type": "DISGUSTED",
                    "confidence": 0.0038862228
                },
                {
                    "type": "ANGRY",
                    "confidence": 0.003117323
                },
                {
                    "type": "HAPPY",
                    "confidence": 0.0012536843
                },
                {
                    "type": "SURPRISED",
                    "confidence": 8.2701445E-4
                },
                {
                    "type": "FEAR",
                    "confidence": 2.2947788E-4
                }
            ]
}

~~~

일단 내 정장 사진을 입은 모습으로 테스트해 본 결과인데 나이는 20세에서 26세 정도로 나오고, 
웃음이 없고 안경을 썼으며, 남자로 판단하고, 수염은 없다고 판단하고, 눈은 떠있고, 입은 닫혀있으며, 감정은 차분하다고 판단했다.
혹시 몰라서 다른 사진으로도 테스트 해 보면

<img src="/img/posts/aws_face_rekognition/2.jpeg" width="80%">

``` json

{
    "faceDetails": [
            "ageRange": {
                "low": 10,
                "high": 16
            },
            ...
            "gender": {
                "value": "Female",
                "confidence": 99.85323
            },
            ...
            "emotions": [
                {
                    "type": "CALM",
                    "confidence": 96.09375
                },
                {
                    "type": "SURPRISED",
                    "confidence": 2.1915436
                },
                {
                    "type": "CONFUSED",
                    "confidence": 0.54200494
                },
                {
                    "type": "DISGUSTED",
                    "confidence": 0.11892319
                },
                {
                    "type": "HAPPY",
                    "confidence": 0.086530045
                },
                {
                    "type": "ANGRY",
                    "confidence": 0.013077259
                },
                {
                    "type": "SAD",
                    "confidence": 0.003439188
                },
                {
                    "type": "FEAR",
                    "confidence": 4.529953E-4
                }
            ],
            ...
}

```

일단 나이가 생각보다 정확하게 나온다는 점에서 놀랐는데, 일단 동양인의 얼굴은 나이가 보다 더 어려보이게 나오는 것 같기는 하다.
나의 경우에도 20세는 턱도 없이 어린 나이인데도 20세로 나오고, 카리나의 경우에는 10세에서 16세로 나왔다. 물론 AWS API이므로
만 나이겠지만, 아무리 그래도 10세는 너무 어린 나이로 나오는게 아닐까ㅋㅋㅋ 그리고 내 성별의 경우에도 남자일 확률이 57% 밖에는 
확신할 수 없다는 점이 좀 웃겼다.

아무튼, 이런 기능을 구현하기 위해서는 AWS Rekognition API를 사용하면 된다. 
내 다른 프로젝트에서 얼굴 인식, 즉 나이대와 성별을 체크하는 기능을 구현하고자 했는데 이를 위해서 AWS Rekognition API를 사용하였다.
그런데 AWS의 자습서 내용이 형편없기도 하고, 그리고 한국어로 된 Spring Boot에서 AWS Rekognition API를 사용하는 방법에 대한 자료가 많이 없어서
이렇게 직접 구현하면서 정리해보려고 한다.

### 2. 의존성 추가

일단 메이븐의 경우에는 아래와 같이 의존성을 추가해주면 된다.


#### pom.xml
``` xml
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-rekognition</artifactId>
            <version>1.12.770</version>
        </dependency>
   
```

자바 버전 17, 그리고 스프링 부트 버전 3.3.1을 사용했고, AWS Rekognition API의 버전은 1.12.770을 사용했다.
그래들의 경우에는

#### build.gradle

``` json

    implementation 'com.amazonaws:aws-java-sdk-rekognition:1.12.770'
    implementation 'com.amazonaws:aws-java-sdk-core:1.12.770'
    
```

이 의존성을 추가해 주어야 한다. 이 의존성을 추가해 주면 다음과 같은 경고 메시지가 나오는데

    2024-08-22T21:12:40.570+09:00  WARN 2168 --- [           main] com.amazonaws.util.VersionInfoUtils      : The 
    AWS SDK for Java 1.x entered maintenance mode starting July 31, 2024 and will reach end of support on December 
    31, 2025. For more information, see https://aws.amazon.com/blogs/developer/the-aws-sdk-for-java-1-x-is-in-
    maintenance-mode-effective-july-31-2024/
    You can print where on the file system the AWS SDK for Java 1.x core runtime is located by setting the AWS_JAVA_V1_
    PRINT_LOCATION environment variable or aws.java.v1.printLocation system property to 'true'.
    This message can be disabled by setting the AWS_JAVA_V1_DISABLE_DEPRECATION_ANNOUNCEMENT environment variable or 
    aws.java.v1.disableDeprecationAnnouncement system property to 'true'.
    The AWS SDK for Java 1.x is being used here:

이런 경고 메시지가 나오는데, 이는 AWS SDK for Java 1.x가 2025년 12월 31일까지 지원이 중단된다는 것을 알려주는 메시지이다.
메이븐 센트럴에서도 확인해 봤지만 일단 1.x 버전이 최신 버전이라서 1.x 버전을 사용해야 하는 것 같다. 2 버전은 출시되지는 않았다.

<img src="/img/posts/aws_face_rekognition/3.png" width="80%">

### 3. AWS 키 설정

그리고 AWS Rekognition API를 사용하기 위해서는 AWS키를 설정해 주어야 하는데, 나는 귀찮아서 루트키를 발급 받았지만 AWS에서는 IAM 사용자를 생성해서
키를 생성하는 것을 권장하고 있다. IAM 사용자를 생성하고, 해당 사용자에게 RekognitionFullAccess 권한을 부여하고, 해당 사용자의 키를 사용하면 된다.

혹시라도 어떻게 하는지 모르겠으면 [여기]("https://aws.amazon.com/de/blogs/security/wheres-my-secret-access-key/")를 참고하면 된다.

<img src="/img/posts/aws_face_rekognition/4.png" width="80%">

그리고 원래는 환경변수로 설정해 주는 것을 권장하지만, 일단 빠른 테스트를 위해 application.properties에 직접 삽입했다.

#### application.properties

``` properties

aws.accessKeyId=your-access-key
aws.secretKey=your-secret-key

```

### 4. AWSRekognitionConfiguration

~~~ java

import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.rekognition.AmazonRekognition;
import com.amazonaws.services.rekognition.AmazonRekognitionClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AwsRekognitionConfiguration {

    @Value("${aws.access-key}")
    private String accessKey;

    @Value("${aws.secret-key}")
    private String secretKey;

    @Bean
    public AmazonRekognition amazonRekognition() {
        BasicAWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);
        return AmazonRekognitionClientBuilder
                .standard()
                .withRegion(Regions.EU_WEST_1)
                .withCredentials(new AWSStaticCredentialsProvider(credentials))
                .build();
    }

}

~~~

이렇게 AWSRekognitionConfiguration을 만들어서 AmazonRekognition 객체를 Bean으로 등록해 주면 된다. 
이 이후에는 AWSRekognitionService를 만들어서 실제 얼굴 인식을 하는 메소드를 만들어 주면 된다.

### 5. AWSRekognitionService 및 AWSRekognitionController 생성

~~~ java

import com.amazonaws.services.rekognition.AmazonRekognition;
import com.amazonaws.services.rekognition.model.*;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.ByteBuffer;

@RequiredArgsConstructor
@Service
public class AwsRekognitionService {
    
    @Autowired
    private AmazonRekognition client;

    }
    public DetectFacesResult detectFacesRequest(MultipartFile multipartFile) throws IOException {
        DetectFacesRequest request = new DetectFacesRequest()
                .withImage(new Image().withBytes(ByteBuffer.wrap(multipartFile.getBytes())))
                .withAttributes(Attribute.ALL);

        return client.detectFaces(request);
    }
}

~~~

~~~ java


import com.amazonaws.services.rekognition.model.DetectFacesRequest;
import com.amazonaws.services.rekognition.model.DetectFacesResult;
import com.example.awsrekognition.service.AwsRekognitionService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api")
public class AwsRekognitionRestController {

    @Autowired
    private AwsRekognitionService awsRekognitionService;

    @PostMapping("/images/test-face")
    public ResponseEntity<DetectFacesResult> detectFaces(@RequestPart MultipartFile image) throws IOException {
        return ResponseEntity.ok(awsRekognitionService.detectFacesRequest(image));
    }
}

~~~

이러면 이미지를 업로드하면 해당 이미지의 얼굴을 인식하는 기능을 구현할 수 있다.

### 6. 마무리

이렇게 AWS Rekognition API를 사용하여 Spring Boot에서 얼굴 인식을 구현하는 방법을 정리해 보았다.
이런 방법 말고도 S3 버킷에 이미지를 업로드하면 해당 이미지의 URL을 가져와서 얼굴 인식을 하는 방법도 있긴 하고,
여러 사진 중에서 해당 사진의 얼굴을 찾는 방법도 있다. 아마도 출입국 심사대나 보안 설정 같은 곳에서 유용하게 사용할 수 있지 않을까 싶다.

끝!