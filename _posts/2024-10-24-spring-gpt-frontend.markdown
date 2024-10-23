---
layout: post
comments: true
title: "[WEB][Spring] 스프링으로 Chat-GPT 페이지 구현하기 - 2"
subtitle: Open-AI Chat-GPT completion api를 이용한 간단한 챗봇 페이지 프론트엔드 구현
description: 
date: 2024-10-24
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][Spring] 스프링으로 간단한 Chat-GPT 페이지 구현하기 - 2

### 1. 서론

이전 포스트에서 백엔드를 구현했으니, 이제 프론트엔드를 구현해 보자. 솔직히 이미 예전에 다 구현하긴 했는데,
이제서야 포스팅하게 되었다.

이전 포스팅은, 백엔드는 [여기](https://ash-tensor.github.io/web/java/2024/09/30/spring-gpt-backend.html)에서 확인할 수 있다.
실제 작성한 예제는 [여기](https://github.com/Ash-tensor/simple-gpt-page)에서 확인할 수 있다.

<img src="/img/posts/spring/gpt/1.png" width="80%">


### 2. 프론트엔드 구현

#### ChattingController.java

~~~ java

package com.chatgptspring.controller;

import com.chatgptspring.service.ChatService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@RequiredArgsConstructor
@Controller
public class ChattingController {
    private final ChatService chatService;

    @GetMapping("/chatpage")
    public String modelController(Model model) {
        model.addAttribute("chats", chatService.FindAllChats());
        return "chatting";
    }
}

~~~

이름을 너무 못짓긴 했는데, 일단 챗 페이지 자체를 리턴하는 컨트롤러를 작성했다.
비록 리액트를 쓰지 않긴 하지만, 리액트처럼 나머지는 js로 작성한 RestAPI를 이용해서 구현할 것이다.

#### Chatting.html

이 부분은 딱히 중요하지 않다. 중요한 부분은 타임리프 템플릿을 이용해서 챗 목록을 그리는 부분과, 
js를 이용해서 메시지를 추가하는 부분이다.

엄청 긴데, 별 내용은 없고, 사실 에셋은 괜찮아 보이는 부분을 가져온 거라서 중요한 부분은 아니다.
아래 js부분을 보거나 아니면 깃허브에서 예제를 확인하면 더 이해가 쉬울 것이다.

~~~ html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="/static/css/chatting.css">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" 
  integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" 
  integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script>
    <script src="/static/js/chatting.js"></script>
    <script src="/static/js/message.js"></script>
</head>
<body>

<div class="flex-box">
    <div class="sidebar d-flex flex-column flex-shrink-0 p-3 text-bg-dark">
        <a href="/static" class="d-flex align-items-center mb-3 
        mb-md-0 me-md-auto text-white text-decoration-none">
            <svg class="bi pe-none me-2" width="40" height="32"><use xlink:href="#bootstrap"></use></svg>
            <span class="fs-4">Sidebar</span>
        </a>
        <hr>
        <ul class="nav nav-pills flex-column mb-auto" id="chatList">
            <!-- 기존 채팅 목록 표시 -->
            <li class="nav-item" th:each="chat : ${chats}" 
            style="padding: 5px" th:onclick="'drawChat(' + ${chat.id} + ')'">
                <a href="#" class="nav-link active" th:text="'채팅 ' + ${chat.id}" aria-current="page">
                    <svg class="bi pe-none me-2" width="16" height="16"><use xlink:href="#home"></use></svg>
                    <span th:text="'채팅 ' + ${chat.id}">채팅 ID</span>
                </a>
            </li>
        </ul>

        <!-- 새 채팅 생성 폼 -->
        <form id="createChatForm">
            <li>
                <button type="button" class="nav-link text-white" 
                style="background: none; border: none; padding: 0;" onclick="createNewChat()">
                    <svg class="bi pe-none me-2" width="16" 
                    height="16"><use xlink:href="#people-circle"></use></svg>
                    Create New Chat
                </button>
            </li>
        </form>

        </ul>
        <hr>
        <div class="dropdown">
            <a href="#" class="d-flex align-items-center text-white text-decoration-none dropdown-toggle"
             data-bs-toggle="dropdown" aria-expanded="false">
                <img src="https://github.com/mdo.png" alt="" width="32" height="32" class="rounded-circle me-2">
                <strong>mdo</strong>
            </a>
            <ul class="dropdown-menu dropdown-menu-dark text-small shadow">
                <li><a class="dropdown-item" href="#">New project...</a></li>
                <li><a class="dropdown-item" href="#">Settings</a></li>
                <li><a class="dropdown-item" href="#">Profile</a></li>
                <li><hr class="dropdown-divider"></li>
                <li><a class="dropdown-item" href="#">Sign out</a></li>
            </ul>
        </div>
    </div>

    <div class="card" style="height: 100%" id="chat-box">
        <span class="title">Comments</span>

        <div class="comment-box" id="message-box">
            <div class="comments" th:each="message: ${chats[0].messages}">
                <div class="comment-react">
                    <button>
                        <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/svg">

                            <path fill="#707277" stroke-linecap="round" stroke-width="2" stroke="#707277" d="M19.4626 3.
                            99415C16.7809 2.34923 14.4404 3.01211 13.0344 4.06801C12.4578 4.50096 12.1696 4.71743 12 4.
                            71743C11.8304 4.71743 11.5422 4.50096 10.9656 4.06801C9.55962 3.01211 7.21909 2.34923 4.
                            53744 3.99415C1.01807 6.15294 0.221721 13.2749 8.33953 19.2834C9.88572 20.4278 10.6588 21 12
                             21C13.3412 21 14.1143 20.4278 15.6605 19.2834C23.7783 13.2749 22.9819 6.15294 19.4626 3.
                             99415Z"></path>
                        </svg>
                    </button>
                    <hr>
                    <span>14</span>
                </div>
                <div class="comment-container">
                    <div th:classappend="${message.role=='user'} ? 'user' : 'gpt'">
                        <div class="user-pic">
                            <svg fill="none" viewBox="0 0 24 24" height="20" width="20" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" fill="#707277" stroke-linecap="round" stroke-width="2"
                                 stroke="#707277" d="M6.57757 15.4816C5.1628 16.324 1.45336 18.0441 3.71266 20.1966C4.
                                 81631 21.248 6.04549 22 7.59087 22H16.4091C17.9545 22 19.1837 21.248 20.2873 20.1966C22.
                                 5466 18.0441 18.8372 16.324 17.4224 15.4816C14.1048 13.5061 9.89519 13.5061 6.57757 15.
                                 4816Z"></path>
                                <path stroke-width="2" fill="#707277" stroke="#707277" d="M16.5 6.5C16.5 8.98528 14.4853
                                 11 12 11C9.51472 11 7.5 8.98528 7.5 6.5C7.5 4.01472 9.51472 2 12 2C14.4853 2 16.5 4.
                                 01472 16.5 6.5Z"></path>
                            </svg>
                        </div>
                        <div class="user-info">
                            <span th:text="${message.role}">Yassine Zanina</span>
                            <p>Wednesday, March 13th at 2:45pm</p>
                        </div>
                    </div>
                    <p th:if="${message.role=='user'}" class="comment-content" style="text-align: left" th:text="$
                    {message.content}"></p>
                    <p th:if="${message.role=='assistant'}" class="comment-content" style="text-align: right" th:text="$
                    {message.content}"></p>
                </div>
            </div>

        </div>

        <div class="text-box">
            <div class="box-container">
                <textarea id="message-content" placeholder="Reply"></textarea>
                <div>
                    <div class="formatting">
                        <button type="button">
                            <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#707277"
                                 d="M5 6C5 4.58579 5 3.87868 5.43934 3.43934C5.87868 3 6.58579 3 8 3H12.5789C15.0206 3
                                  17 5.01472 17 7.5C17 9.98528 15.0206 12 12.5789 12H5V6Z" clip-rule="evenodd"
                                   fill-rule="evenodd"></path>
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#707277"
                                 d="M12.4286 12H13.6667C16.0599 12 18 14.0147 18 16.5C18 18.9853 16.0599 21 13.6667
                                  21H8C6.58579 21 5.87868 21 5.43934 20.5607C5 20.1213 5 19.4142 5 18V12"></path>
                            </svg>
                        </button>
                        <button type="button">
                            <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linecap="round" stroke-width="2.5" stroke="#707277" d="M12 4H19"></path>

                                <path stroke-linecap="round" stroke-width="2.5" stroke="#707277" d="M8 20L16 4"></path>
                                <path stroke-linecap="round" stroke-width="2.5" stroke="#707277" d="M5 20H12"></path>
                            </svg>
                        </button>
                        <button type="button">
                            <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#707277"
                                 d="M5.5 3V11.5C5.5 15.0899 8.41015 18 12 18C15.5899 18 18.5 15.0899 18.5 11.5V3"></path>
                            
                                <path stroke-linecap="round" stroke-width="2.5" stroke="#707277" d="M3 21H21"></path>
                            </svg>
                        </button>
                        <button type="button">
                            <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#707277"
                                 d="M4 12H20"></path>
                                <path stroke-linecap="round" stroke-width="2.5" stroke="#707277" d="M17.5 7.66667C17.5 5.
                                08934 15.0376 3 12 3C8.96243 3 6.5 5.08934 6.5 7.66667C6.5 8.15279 6.55336 8.59783 6.
                                6668 9M6 16.3333C6 18.9107 8.68629 21 12 21C15.3137 21 18 19.6667 18 16.3333C18 13.9404
                                 16.9693 12.5782 14.9079 12"></path>
                            </svg>
                        </button>
                        <button type="button">
                            <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/
                            svg">
                                <circle stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5"
                                 stroke="#707277" r="10" cy="12" cx="12"></circle>
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#707277"
                                 d="M8 15C8.91212 16.2144 10.3643 17 12 17C13.6357 17 15.0879 16.2144 16 15"></path>
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="3" stroke="#707277"
                                 d="M8.00897 9L8 9M16 9L15.991 9"></path>
                            </svg>
                        </button>
                        <button type="submit" class="send" title="Send" id="send-message" onclick="sendMessage()">
                            <svg fill="none" viewBox="0 0 24 24" height="18" width="18" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#ffffff"
                                 d="M12 5L12 20"></path>
                                <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#ffffff"
                                 d="M7 9L11.2929 4.70711C11.6262 4.37377 11.7929 4.20711 12 4.20711C12.2071 4.20711 12.
                                 3738 4.37377 12.7071 4.70711L17 9"></path>
                            </svg>
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

</div>

</body>
</html>


~~~

일단 위 컴포넌트는 다른 사람이 작성한 코드를 참고했는데, 글자 크기가 너무 작긴 한것 같지만...
여기서 

~~~ html

<button type="button" class="nav-link text-white" style="background: none; border: none; padding: 0;"
 onclick="createNewChat()">
    
~~~

createNewChat() 함수는 새로운 챗 페이지를 생성하는 함수고, 

~~~ html

<button type="submit" class="send" title="Send" id="send-message" onclick="sendMessage()">
~~~

sendMessage() 함수는 메시지를 보내는 함수이다. 
타임리프 템플릿을 이용해서 버튼을 누르거나 할 때마다 페이지 자체를 리로드하는 방식도 있겠지만, 
그런 방식은 성능이 떨어지기 때문에 이런 식으로 구현했다.

타임리프 템플릿은 처음 페이지를 로드할 때만 이용된다.

~~~ html

<ul class="nav nav-pills flex-column mb-auto" id="chatList">
    <!-- 기존 채팅 목록 표시 -->
    <li class="nav-item" th:each="chat : ${chats}" style="padding: 5px" th:onclick="'drawChat(' + ${chat.id} + ')'">
        <a href="#" class="nav-link active" th:text="'채팅 ' + ${chat.id}" aria-current="page">
            <svg class="bi pe-none me-2" width="16" height="16"><use xlink:href="#home"></use></svg>
            <span th:text="'채팅 ' + ${chat.id}">채팅 ID</span>
        </a>
    </li>
</ul>

~~~

이는 기존 채팅 목록을 표시하는 부분이다. 채팅 목록을 클릭하면 drawChat() 함수가 호출되고, 이는 해당 채팅 목록을 그리는 함수이다.


#### Chatting.js


~~~ js

function createNewChat() {
    // Ajax POST 요청으로 새 채팅 생성
    fetch('/chat/new', {
        method: 'POST'
    }).then(response => {
        if (response.ok) {
            // 새 채팅이 생성되면 채팅 목록을 다시 불러오는 함수 호출
            updateChatList();
        }
    }).catch(error => console.error('Error:', error));
}

// 채팅 목록을 새로 불러오는 함수
function updateChatList() {
    fetch('/chat')  // 채팅 목록을 불러올 API 엔드포인트
        .then(response => response.json())  // JSON 응답을 파싱
        .then(data => {
            const chatList = document.getElementById('chatList');
            chatList.innerHTML = '';  // 기존 채팅 목록 초기화

            // 불러온 데이터를 바탕으로 채팅 목록을 다시 렌더링
            data.forEach(chat => {
                const li = document.createElement('li');
                li.classList.add('nav-item');
                li.style.padding = '5px';

                li.innerHTML = `
                    <a href="#" class="nav-link active" aria-current="page">
                        <svg class="bi pe-none me-2" width="16" height="16"><use xlink:href="#home"></use></svg>
                        채팅 ${chat.id}
                    </a>
                `;
                chatList.appendChild(li);  // 새로 만든 항목을 목록에 추가
            });
        }).catch(error => console.error('Error:', error));
}

~~~


#### message.js

~~~ js

async function sendMessage() {

    const chatId = localStorage.getItem('chatId');
    const content = document.getElementById('message-content').value;

    const sendedMessage = createMessage({role: 'user', content: content});
    const chatBox = document.getElementById('message-box');
    chatBox.appendChild(sendedMessage);

    const response = await fetch('/chat/' + chatId, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({content: content}),
    });
    const message = await response.json();
    const recievedMessage = createMessage(message);

    chatBox.appendChild(recievedMessage);

}

function createMessage(messageData) {
    const message = document.createElement('div');
    message.classList.add('comments');
    message.innerHTML = `
            <div class="comment-react">
                <button>
                    <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/svg">
                        <path fill="#707277" stroke-linecap="round" stroke-width="2" stroke="#707277" d="M19.4626 3.
                        99415C16.7809 2.34923 14.4404 3.01211 13.0344 4.06801C12.4578 4.50096 12.1696 4.71743 12 4.
                        71743C11.8304 4.71743 11.5422 4.50096 10.9656 4.06801C9.55962 3.01211 7.21909 2.34923 4.53744 3.
                        99415C1.01807 6.15294 0.221721 13.2749 8.33953 19.2834C9.88572 20.4278 10.6588 21 12 21C13.3412
                         21 14.1143 20.4278 15.6605 19.2834C23.7783 13.2749 22.9819 6.15294 19.4626 3.99415Z"></path>
                    </svg>
                </button>
                <hr>
                <span>14</span>
            </div>
                <div class="comment-container">
                    <div class="${messageData.role === 'user' ? 'user' : 'gpt'}">
                        <div class="user-pic">
                            <svg fill="none" viewBox="0 0 24 24" height="20" width="20" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" fill="#707277" stroke-linecap="round" stroke-width="2"
                                 stroke="#707277" d="M6.57757 15.4816C5.1628 16.324 1.45336 18.0441 3.71266 20.1966C4.
                                 81631 21.248 6.04549 22 7.59087 22H16.4091C17.9545 22 19.1837 21.248 20.2873 20.1966C22.
                                 5466 18.0441 18.8372 16.324 17.4224 15.4816C14.1048 13.5061 9.89519 13.5061 6.57757 15.
                                 4816Z"></path>
                                <path stroke-width="2" fill="#707277" stroke="#707277" d="M16.5 6.5C16.5 8.98528 14.4853
                                 11 12 11C9.51472 11 7.5 8.98528 7.5 6.5C7.5 4.01472 9.51472 2 12 2C14.4853 2 16.5 4.
                                 01472 16.5 6.5Z"></path>
                            </svg>
                        </div>
                        <div class="user-info">
                            <span>${messageData.role}</span>
                            <p>Wednesday, March 13th at 2:45pm</p>
                        </div>
                    </div>
                    <p class="comment-content" style="text-align: ${messageData.role === 'user' ? 'left' : 'right'}">
                        ${messageData.content}
                    </p>
                    
                </div>
        `;

    return message;
    
}

async function drawChat(chatId) {
    localStorage.setItem('chatId', chatId);

    try {
    const response = await fetch('/chat/' + chatId);
    const chat = await response.json();
    const messages = chat.messages;

    const chatBox = document.getElementById('message-box');
    chatBox.innerHTML = '';

    messages.forEach(messageData => {
        var message = document.createElement('div');
        message.classList.add('comments');
        message.innerHTML = `
            <div class="comment-react">
                <button>
                    <svg fill="none" viewBox="0 0 24 24" height="16" width="16" xmlns="http://www.w3.org/2000/svg">
                        <path fill="#707277" stroke-linecap="round" stroke-width="2" stroke="#707277" d="M19.4626 3.
                        99415C16.7809 2.34923 14.4404 3.01211 13.0344 4.06801C12.4578 4.50096 12.1696 4.71743 12 4.
                        71743C11.8304 4.71743 11.5422 4.50096 10.9656 4.06801C9.55962 3.01211 7.21909 2.34923 4.53744 3.
                        99415C1.01807 6.15294 0.221721 13.2749 8.33953 19.2834C9.88572 20.4278 10.6588 21 12 21C13.3412
                         21 14.1143 20.4278 15.6605 19.2834C23.7783 13.2749 22.9819 6.15294 19.4626 3.99415Z"></path>
                    </svg>
                </button>
                <hr>
                <span>14</span>
            </div>
                <div class="comment-container">
                    <div class="${messageData.role === 'user' ? 'user' : 'gpt'}">
                        <div class="user-pic">
                            <svg fill="none" viewBox="0 0 24 24" height="20" width="20" xmlns="http://www.w3.org/2000/
                            svg">
                                <path stroke-linejoin="round" fill="#707277" stroke-linecap="round" stroke-width="2"
                                 stroke="#707277" d="M6.57757 15.4816C5.1628 16.324 1.45336 18.0441 3.71266 20.1966C4.
                                 81631 21.248 6.04549 22 7.59087 22H16.4091C17.9545 22 19.1837 21.248 20.2873 20.1966C22.
                                 5466 18.0441 18.8372 16.324 17.4224 15.4816C14.1048 13.5061 9.89519 13.5061 6.57757 15.
                                 4816Z"></path>
                                <path stroke-width="2" fill="#707277" stroke="#707277" d="M16.5 6.5C16.5 8.98528 14.4853
                                 11 12 11C9.51472 11 7.5 8.98528 7.5 6.5C7.5 4.01472 9.51472 2 12 2C14.4853 2 16.5 4.
                                 01472 16.5 6.5Z"></path>
                            </svg>
                        </div>
                        <div class="user-info">
                            <span>${messageData.role}</span>
                            <p>Wednesday, March 13th at 2:45pm</p>
                        </div>
                    </div>
                    <p class="comment-content" style="text-align: ${messageData.role === 'user' ? 'left' : 'right'}">
                        ${messageData.content}
                    </p>
                    
                </div>
        `;
        chatBox.appendChild(message);
    });
} catch (error) {
    console.error('Error:', error);
    }
}

~~~

솔직히 이렇게 할거면 그냥 리액트 쓸걸 그랬다. 그게 더 편리했을 것 같기도 하고...
아무튼, 이렇게 구현하면 페이지 자체를 리로드하지 않고도 메시지를 추가할 수 있다.

코드를 읽어보면 이해하겠지만, 챗 목록을 클릭하면 drawChat() 함수가 호출되고, 이 함수는 해당 챗의 메시지를 그리는 함수이다.
각각의 메시지는 createMessage() 함수를 통해 그려지는데, 이 함수는 메시지의 역할과 내용을 받아서 메시지를 그린다.


### 마치며

만약 실행이 안되거나 오류가 있으면 내가 작성한 테스트 코드를 
[https://github.com/Ash-tensor/simple-gpt-page](https://github.com/Ash-tensor/simple-gpt-page) 에서 확인할 수 있다.

그럼 안녕!!