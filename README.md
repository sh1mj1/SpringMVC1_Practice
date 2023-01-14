# SpringMVC1_Practice

# 1. 프로젝트 준비

프로젝트 준비

[https://start.spring.io/](https://start.spring.io/)


Lombok 설정

실행 속도 높이기

테스트 [http://localhost:8080](http://localhost:8080) 호출해서 Whitelabel Error Page 확인

### **Welcome 페이지 추가**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li>상품관리
        <ul>
            <li><a href="/basic/items"> 상품관리 - 기본</a></li>
        </ul>
    </li>
</ul>

</body>
</html>
```


# 2. 요구사항 분석

상품을 관리할 수 있는 서비스 만들어 봅시다.

**상품 도메인 모델**

- 상품 ID
- 상품명
- 가격
- 수량

**상품 관리 기능**

- 상품 목록
- 상품 상세
- 상품 등록
- 상품 수정

**서비스 화면**

최상단 링크에서 확인

**서비스 제공 흐름**


> React, Vue, js 같은 웹 클라이언트 기술을 사용하고 웹 프론트엔드 개발자가 웹 퍼블리셔 역할까지 포함하는 경우도 있음.
이 때 프론트엔드 개발자가 HTML 을 동적으로 만들고 웹 화면의 흐름을 담당. 이 경우 벡엔드 개발자는 HTML 뷰 템플릿을 직접 만지는 대신, HTTP API 을 통해 클라이언트가 필요로하는 데이터, 기능을 제공하면 됨.


