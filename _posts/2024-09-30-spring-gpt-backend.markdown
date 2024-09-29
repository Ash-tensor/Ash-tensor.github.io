---
layout: post
comments: true
title: "[WEB][Spring] 스프링으로 Chat-GPT 기능 구현하기 - 1"
subtitle: Open-AI Chat-GPT completion api를 이용한 간단한 챗봇 페이지 백엔드 구현
description: 
date: 2024-09-30
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][Spring] 스프링으로 간단한 Chat-GPT 페이지 구현하기 - 1

### 1. 서론

<img src="/img/posts/spring/gpt/1.png" width="90%">

개인적으로 진행했던 프로젝트에서 스프링에 GPT-4o API를 이용해 현재 상황을 분석하고 앞으로의 전략을 제시하는 기능을 구현했었는데,
이번에는 GPT-4o API를 이용해서 간단한 채팅 페이지를 구현해 보고자 한다. 간단한 completion 기능을 이용하고자 한다면 굳이 구독하지는 않아도 
되지 않을까 싶기도 하고 스프링도 손에 익어서 몇 시간이면 구현이 가능할 것 같아서 완성해 보았다.

약 2일 정도 걸렸고 대부분은 프론트엔드에서 걸린 시간이 많았다. 
아직 불완전한 부분이 있긴 하지만 충분히 사용할만한 수준으로, 챗GPT 페이지와 유사한 형태의 채팅 페이지를 구현했다.

### 2. API 요청 방법

일단 OpenAI API를 사용하기 위해서는 API Key가 필요하다. 이는 OpenAi 홈페이지에서 계정을 만들고 API Key를 발급받으면 된다.
또한 OpenAI API를 사용하기 위해서는 요청을 보낼 때, JSON 형식으로 데이터를 보내야 한다.
이 JSON 형식은 다음과 같은데
    
```json
{
    "model": "요청 모델 명 (ex. gpt-3.5-turbo)",
    "messages": [
        {
            "role": "user",
            "content": "사용자가 입력한 메시지"
        },
        {
            "role": "assistant",
            "content": "사용자가 입력한 메시지에 대한 응답"
        }
    ]
}
```
하나의 챗 페이지에서 대화한 내용을 GPT에 보낼 때 위와 같은 형식으로 보내야 한다.

보면 눈치채겠지만, **GPT의 입력은 하나의 챗 페이지에서 대화한 메시지를 모두 다 입력으로 보낸다!!**

이는 GPT가 한번의 챗 페이지에서 대화한 내용을 계속 반복해서 입력받는다는 뜻이다. 이를 모르는 사람들이 꽤 있었는데,
다시 말하면 결국 하나의 대화가 길어지면 길어질수록 GPT의 맥락이 점점 한정되어서 성능이 떨어지게 된다. 생각할 공간이 줄어드는 것과 비슷하다.

또한 토큰에서도 불이익이 있는데 이는 GPT가 한번에 처리할 수 있는 토큰의 수가 제한되어 있기 때문이다. 예를 들어서, GPT-3.5-turbo의 경우 4096개의 토큰을 처리할 수 있다.
게다가 GPT의 과금에 있어서, OpenAI는 1000개의 토큰을 처리할 때마다 일정량의 과금을 받는데, 한번의 대화가 길어지면 길어질수록 이전 대화의 메시지가 계속 입력되어서
보다 떨어지는 성능인데도 불구하고 더 많은 과금이 발생하게 된다.

그러니까 불판을 꼭 자주 갈자ㅋㅋ 한번의 대화에서 꼭 고맥락이 필요한 게 아니라면 성능상의 이점은 물론이고 과금적인 측면에서도 대화가 길어지면 길어질수록
더 많은 불이익이 발생하게 된다.

아무튼, 이러한 GPT의 입력 형식을 알았으니, 이제 이를 스프링에서 어떻게 구현할지 알아보자.

### 2. Configuration 파일

인터넷에서 찾은 대부분의 코드는 WebClient를 사용한 것이 아니라 RestTemplate을 사용한 코드가 많았다.
하지만, RestTemplate은 더 이상 사용되지 않는다고 하니 WebClient를 사용해서 구현해 보았다.

#### OpenAIConfig.java

~~~ java

package com.chatgptspring.config;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class OpenAIConfig {
    @Value("${openai.api.key}") // application.properties에 저장된 openai.api.key를 불러온다.
    private String openAiKey;

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
                .baseUrl("https://api.openai.com")
                .defaultHeader("Authorization", "Bearer " + openAiKey)
                .build();
    }
    
    // ObjectMapper 빈 등록

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}

~~~

#### application.properties

~~~ properties

spring.application.name=chatgpt-spring

spring.mvc.static-path-pattern=/static/**
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

openai.api.key= 본인 키파일

spring.datasource.url=본인 DB 호스트
spring.datasource.username=본인 DB 유저
spring.datasource.password=본인 DB 비밀번호

~~~

이렇게 설정을 하면 OpenAIConfig 파일에서 WebClient를 빈으로 등록하고, application.properties에서 OpenAI API Key를 불러와서 사용할 수 있다.
DB 설정은 본인의 DB에 맞게 설정하면 된다. 나는 MariaDB를 사용했다.
이정도는 스프링을 사용하면서 자주 사용하는 설정이니까 크게 어렵지 않을 것이다. DB에 저장하지 않고, GPT의 응답만 받아서 사용하고 싶다면 DB 설정은 필요없다.


### 3. Domain 파일

여기에는 DB와 연결되는 엔티티 클래스를 정의한다. 만약 단순히 GPT를 이용해서 응답을 받는 기능을 스프링에 탑재하고자 한다면 이 부분은 필요없이 DTO만 작성하면 된다.
아무튼 나는 Chat-gpt 페이지와 유사한 형태의 채팅 페이지를 직접 구현해 보고자 했기 때문에 엔티티 클래스를 작성했다.


#### Chat.java

~~~ java

package com.chatgptspring.domain;

import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.util.List;

@Getter @Setter
@Table(name = "chat")
@Entity
public class Chat {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    @OneToMany(mappedBy = "chat")
    @JsonManagedReference
    private List<Message> messages;

}

~~~

이는 앞서 말했던 것처럼 Chat 페이지와 유사한 형태의 채팅 페이지를 구현하기 위한 엔티티 클래스이다. 챗은 맥락을 가지고 있는 메시지의 집합이라고 생각하면 된다.
어떤 유저(비록 아직 구현하지는 않았지만...ㅋㅋ)의 채팅인지와 메시지의 리스트를 가지고 있다.

#### Message.java

~~~ java

package com.chatgptspring.domain;

import com.fasterxml.jackson.annotation.JsonBackReference;
import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.util.Date;

@Getter @Setter
@Table(name = "message")
@Entity
public class Message {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String role;

    @Column(length = 10000)
    private String content;

    @ManyToOne
    @JoinColumn(name = "chat_id")
    @JsonBackReference
    private Chat chat;

    private Integer tokenCount;
    private String model;
    private Date datetime;
}

~~~

메시지는 챗의 메시지이다. 메시지는 역할, 내용, 토큰 수, 모델, 날짜를 가지고 있다. 역할은 사용자인지 GPT인지를 나타내고, 내용은 메시지의 내용을 나타낸다.
토큰 수는 GPT가 처리한 토큰의 총합을 나타내는데 보낸 메시지의 토큰 수와 받은 메시지의 토큰 수를 합한 값이다. 이게 조금 이상하다고 생각할 수 있지만,
어차피 Open-ai에서 과금을 계산할 때 보낸 메시지의 토큰 수, 받은 메시지의 토큰 수를 합해 계산하기 때문에 상관이 없기도 하고,

보낸 메시지의 한국어 토큰 계산은 결국 response를 받아야만 알 수 있기 때문에 이렇게 구현했다.

#### User.java

~~~ java

package com.chatgptspring.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.RequiredArgsConstructor;
import lombok.Setter;

import java.util.List;

@Getter @Setter
@Table(name = "user")
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    private String password;

    @OneToMany(mappedBy = "user")
    private List<Chat> chats;

}

~~~

### 4. Repository 파일

빠른 개발을 위해서 필요한 부분만 간단히 작성했다. 성능이 나쁠수도 있다. 
DB설정이 안되어 있거나 DB에 저장하지 않고, GPT의 응답만 받아서 사용하고 싶다면 이 부분은 필요없다.

#### ChatRepository.java

~~~ java

package com.chatgptspring.repository;

import com.chatgptspring.domain.Chat;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ChatRepository extends JpaRepository<Chat, Long> {
}

~~~

#### MessageRepository.java

~~~ java

package com.chatgptspring.repository;

import com.chatgptspring.domain.Message;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MessageRepository extends JpaRepository<Message, Long> {
}

~~~

#### UserRepository.java

~~~ java

package com.chatgptspring.repository;

import com.chatgptspring.domain.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}

~~~

### 5. DTO 파일

#### ChatDto.java

~~~ java

package dto;

import lombok.Data;

import java.util.Date;
import java.util.List;

@Data
public class ChatDTO {
    private String model;
    private List<MessageDTO> messages;
}

~~~

이 DTO는 GPT의 입력 형식과 동일하게 구성되어 있다. 이를 JSON 형식으로 파싱해서 GPT에 보낼 것이다. 

#### MessageDTO.java

~~~ java

package dto;

import lombok.Data;

import java.util.Date;

@Data
public class MessageDTO {
    private String role;
    private String content;
}

~~~

role은 사용자인지 GPT인지를 나타내고, content는 메시지의 내용을 나타낸다. role의 경우 사용자의 메시지는 user, GPT의 응답은 assistant로 고정되어 있어서 다른 값을 넣으면 Bad Request가 발생한다.
system 롤이 있는데, 이는 해당 대화(Chat)의 GPT의 전반적인 맥락과 규칙을 지시하는 역할을 하고, 보통 가장 먼저 보내는 메시지에 사용된다. 

예를 들어서 해당 GPT를 카페의 점원으로 사용하고자 할 때 system role을 이용해서 "너는 카페의 점원이야"라는 메시지를 먼저 보내고, 그 다음에 사용자의 메시지를 보내는 식이다.


### 6. Service 파일

#### ChatService.java

~~~ java

package com.chatgptspring.service;

import com.chatgptspring.domain.Chat;
import com.chatgptspring.repository.ChatRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@RequiredArgsConstructor
@Service
public class ChatService {
    private final ChatRepository chatRepository;
    public Chat CreateChat() {
        Chat chat = new Chat();
        chatRepository.save(chat);
        return chat;
    }

    public Iterable<Chat> FindAllChats() {
        return chatRepository.findAll();
    }

    public void DeleteChat(Long chatId) {
        chatRepository.deleteById(chatId);
    }

    public Chat FindChatById(Long chatId) {
        return chatRepository.findById(chatId).orElse(null);
    }

}

~~~

#### MessageService.java

~~~ java

package com.chatgptspring.service;

import com.chatgptspring.domain.Chat;
import com.chatgptspring.domain.Message;
import com.chatgptspring.repository.MessageRepository;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import dto.ChatDTO;
import dto.MessageDTO;
import io.netty.handler.codec.http.HttpResponseStatus;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

import java.util.ArrayList;
import java.util.List;

@RequiredArgsConstructor
@Service
public class MessageService {
    private final ChatService chatService;
    private final MessageRepository messageRepository;
    @Autowired
    private WebClient webClient;
    @Autowired
    private ObjectMapper objectMapper;

    public Message SendMessage(Long chatId, String content) {
        Chat chat = chatService.FindChatById(chatId);
        Message message = new Message();
        message.setChat(chat);
        message.setRole("user");
        message.setContent(content);
        messageRepository.save(message);

        List<Message> messages = chat.getMessages();
        List<MessageDTO> messageDTOs = new ArrayList<>();
        for (Message m : messages) {
            MessageDTO messageDTO = new MessageDTO();
            messageDTO.setRole(m.getRole());
            messageDTO.setContent(m.getContent());
            messageDTOs.add(messageDTO);
        }

        ChatDTO chatDTO = new ChatDTO();
        chatDTO.setMessages(messageDTOs);
        chatDTO.setModel("gpt-3.5-turbo");

        try {
            String response = webClient.post()
                    .uri("https://api.openai.com/v1/chat/completions")
                    .bodyValue(chatDTO)
                    .retrieve()
                    .bodyToMono(String.class)
                    .block();

            JsonNode root = objectMapper.readTree(response);
            String assistantContent = root.path("choices").get(0).path("message").path("content").asText();
            Integer totalTokens = root.path("usage").path("total_tokens").asInt();

            Message responseMessage = new Message();

            responseMessage.setChat(chat);
            responseMessage.setModel(chatDTO.getModel());
            responseMessage.setRole("assistant");
            responseMessage.setContent(assistantContent);
            responseMessage.setTokenCount(totalTokens);
            messageRepository.save(responseMessage);
            return responseMessage;

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}

~~~

일단 빠른 테스트를 위해서 model을 고정시켰다. 필요하다면 이를 동적으로 변경하면 된다. 앞서 작성한 DTO 파일과 도메인을 이용해서 GPT에 요청을 보내고, 응답을 받아서 DB에 저장한다.
만약 도메인 저장이 필요가 없다면 해당 부분을 삭제하면 된다. 

Configuration 파일에서 빈으로 등록했던 ObjectMapper를 이용해서 JSON 형식의 응답을 파싱한다.


### 7. Controller 파일

#### ChatController.java

~~~ java

package com.chatgptspring.controller;

import com.chatgptspring.domain.Chat;
import com.chatgptspring.repository.ChatRepository;
import com.chatgptspring.service.ChatService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class ChatController {
    private final ChatService chatService;

    @GetMapping("/chat")
    public Iterable<Chat> FindAllChats() {
        return chatService.FindAllChats();
    }

    @PostMapping("/chat/new")
    public void CreateChat() {
        chatService.CreateChat();
    }

    @GetMapping("/chat/{chatId}")
    public Chat FindChatById(@PathVariable Long chatId) {
        return chatService.FindChatById(chatId);
    }
}

~~~

#### MessageController.java

~~~ java

package com.chatgptspring.controller;

import com.chatgptspring.domain.Message;
import com.chatgptspring.service.ChatService;
import com.chatgptspring.service.MessageService;
import dto.ChatDTO;
import dto.MessageDTO;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;


@RequiredArgsConstructor
@RestController
public class MessageController {
    private final ChatService chatService;
    private final MessageService messageService;

    @PostMapping("/chat/{chatId}")
    public Message CreateMessage(@PathVariable Long chatId, @RequestBody MessageDTO messageDTO) {
        String messageContent = messageDTO.getContent();
        return messageService.SendMessage(chatId, messageContent);
    }

}

~~~

### 8. 테스트

이제 Postman을 이용해서 테스트를 해보자. 먼저 챗을 생성하고, 챗의 id를 이용해서 메시지를 보내면 된다.

#### Chat 생성

<img src="/img/posts/spring/gpt/2.png" width="90%">

#### 메시지 보내기

<img src="/img/posts/spring/gpt/3.png" width="90%">
<img src="/img/posts/spring/gpt/4.png" width="90%">

간단한 시를 써보라고 했는데, 

>> 가을밤에 별빛이 찬란하고, 너의 손을 잡으면 행복해지는 마법 같아.

라고 시를 작성해 주었다.

꽤나 문학적인걸, GPT.

### 9. 마치며

아직 백엔드가 완벽히 구현된 것은 아니다. 이를 이용해서 사용자 인증도 구현해야 하고, 더 많은 기능을 추가해야 한다.
프론트엔드와 연결하는 부분은 다음 포스트에서 다루도록 하겠다.