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

# 3. 상품 도메인 개발


**Item - 상품 도메인 클래스.**

```java
package hello.itemservice.domain.item;

import lombok.Data;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item(){}

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

@Data 로 @Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode. 을 만들어줌.

**`ItemRepository` - 상품 저장소**

```java
package hello.itemservice.domain.item;

@Repository
public class ItemRepository {
    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }
    public Item findById(Long id) {
        return store.get(id);
    }
    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }
    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
				findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }
    public void clearStore() {
        store.clear();
    }
}
```

Item 을 save 했을 때 Id 는 하나씩 증가하도록 설정함. 

**`ItemRepositoryTest` - 상품 저장소 테스트**

```java
package hello.itemservice.domain.item;

import static org.assertj.core.api.Assertions.assertThat;

class ItemRepositoryTest {

    ItemRepository itemRepository = new ItemRepository();

    @AfterEach
    void tearDown() {
        itemRepository.clearStore();
    }

    @Test
    void save() {
        // given
        Item item = new Item("itemA", 1000, 10);
        // when
        Item savedItem = itemRepository.save(item);
        // then
        Item findItem = itemRepository.findById(item.getId());
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void findAll() {
        // given
        Item item1 = new Item("item1", 1000, 10);
        Item item2 = new Item("item2", 2000, 20);
        itemRepository.save(item1);
        itemRepository.save(item2);
        // when
        List<Item> result = itemRepository.findAll();
        // then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(item1, item2);
    }

    @Test
    void updateItem() {
        // given
        Item item = new Item("item1", 1000, 10);
        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();
        // when
        Item updateParam = new Item("item2", 2000, 30);
        itemRepository.updateItem(itemId, updateParam);
        Item findItem = itemRepository.findById(itemId);
        // then
        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());

    }
}
```

실제 서비스를 구축할 때는 HashMap 과 Long 대신 동시성을 고려하여 ConcurrentHashMap 과 AtomicLong 을 사용해야 한다.

update() 의 경우 Item 객체의 id 를 사용하지 않으므로 ItemUpdateDto 와 같이 id 를 포함하지 않는 객체를 만드는 것이 명확하다. 중복을 피하는 것과 명확성을 높이는 것 중 명확성을 높이는 것이 좋음.


# 4. 상품 서비스 HTML

핵심 비즈니스 로직을 개발하는 동안, 웹 퍼블리셔는 HTML 마크업을 완료했다.다음 파일들을 경로에 넣고 잘 동작하는지 확인해보자.

**부트스트랩**

참고로 HTML을 편리하게 개발하기 위해 부트스트랩을 사용했다. 먼저 필요한 부트스트랩 파일을 설치하자.

- **부트스트랩 공식 사이트:** [https://getbootstrap.com](https://getbootstrap.com/)
- **부트스트랩을 다운로드 받고 압축을 풀자.**
    - 이동: [https://getbootstrap.com/docs/5.0/getting-started/download/](https://getbootstrap.com/docs/5.0/getting-started/download/)
    - Compiled CSS and JS 항목을 다운로드하자.
    - 압축을 풀고 bootstrap.min.css를 복사해서 다음 폴더에 추가하자
    - resources/static/css/bootstrap.min.css

**참고**부트스트랩(Bootstrap)은 웹사이트를 쉽게 만들 수 있게 도와주는 HTML, CSS, JS 프레임워크이다. 하나의 CSS로 휴대폰, 태블릿, 데스크탑까지 다양한 기기에서 작동한다. 다양한 기능을 제공하여 사용자가 쉽게 웹사이트를 제작, 유지, 보수할 수 있도록 도와준다.

**HTML, css 파일**

- /resources/static/css/bootstrap.min.css **→** 부트스트랩 다운로드
- /resources/static/html/`items.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <link href="../css/bootstrap.min.css" rel="stylesheet">
        <title>Title</title>
    </head>
    <body>
    <div class="container" style="max-width: 600px">
        <div class="py-5 text-center">
            <h2> 상품 목록</h2>
        </div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary float-end"
                        onclick="location.href='addForm.html'" type="button">상품 등록
                </button>
            </div>
        </div>
    
        <hr class="my-4">
        <div>
            <table class="table">
                <thead>
                <tr>
                    <th>ID</th>
                    <th>상품명</th>
                    <th>가격</th>
                    <th>수량</th>
                </tr>
                </thead>
                <tbody>
                <tr>
                    <td><a href="items.html">2</a></td>
                    <td><a href="items.html">테스트 상품2</a></td>
                    <td>20000</td>
                    <td>20</td>
                </tr>
                </tbody>
            </table>
        </div>
    </div>
    
    </body>
    </html>
    ```
    
- /resources/static/html/`item.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link href="../css/bootstrap.min.css" rel="stylesheet">
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 상세</h2>
        </div>
        <div>
            <label for="itemId">상품 ID</label>
            <input type="text" id="itemId" name="itemId"
                   class="form-control" value="1" readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName"
                   class="form-control" value="상품A" readonly>
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price"
                   class="form-control" value="10000" readonly>
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="quantity" name="quantity"
                   class="form-control" value="10" readonly>
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg"
                        onclick="location.href='editForm.html'" type="button">상품 수정</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'" type="button">목록으로
                </button>
            </div>
        </div>
        <!--container-->
    </div>
    </body>
    </html>
    ```
    
- /resources/static/html/`addForm.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/html">
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="../css/bootstrap.min.css">
        <title>Title</title>
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 등록 폼</h2>
        </div>
        <h4 class="mb-3">상품 입력</h4>
        <form action="item.html" method="post">
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName" name="itemName"
                       class="form-control" placeholder="이름을 입력하세요">
            </div>
            <div>
                <label for="price">가격</label>
                <input type="text" id="price" name="price"
                       class="form-control" placeholder="가격을 입력하세요">
            </div>
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity" name="quantity"
                       class="form-control" placeholder="수량을 입력하세요">
            </div>
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg">상품 등록</button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg"
                            onclick="location.href='items.html'"
                            type="button">취소
                    </button>
                </div>
            </div>
        </form>
    
    </div>
    
    </body>
    </html>
    ```
    
- /resources/static/html/`editForm.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" href="../css/bootstrap.min.css">
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5text-center">
            <h2>상품 수정 폼</h2>
        </div>
        <form action="item.html" method="post">
            <div>
                <label for="id">상품 ID</label>
                <input type="text" id="id"
                       name="id" class="form-control"
                       value="1" readonly>
            </div>
    
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName"
                       name="itemName" class="form-control" value="상품A">
            </div>
    
            <div>
                <label for="price">가격</label>
                <input type="text" id="price"
                       name="price" class="form-control" value="10000">
            </div>
    
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity"
                       name="quantity" class="form-control" value="10">
            </div>
    
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg" type="submit">
                        저장
                    </button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-secondary btn-lg"
                            onclick="location.href='item.html'" type="button">취소
                    </button>
    
                </div>
            </div>
    
        </form>
    </div>
    
    </body>
    </html>
    ```
    

이렇게 정적 리소스가 공개되는 /resources/static 폴더에 HTML 을 넣어두면, 실제 서비스에서도 공개됨. 만약 서비스를 운영한다면 지금처럼 공개할 필요 없는 HTML 을 두는 것은 주의합시다.


# 5. 상품 목록 - 타임리프

### **컨트롤러와 뷰 템플릿 개발!**

`BasicItemController`

```java
package hello.itemservice.web.item.basic;

import org.springframework.ui.Model;

@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }
    // 테스트용 데이터 추가
		@PostConstruct
    public void init() {
        itemRepository.save(new Item("testA", 10000, 10));
        itemRepository.save(new Item("testB", 20000, 20));
    }
}
```

itemRepository 에서 모든 상품을 조회한 후, 모델에 담는다. 그리고 뷰 템플릿을 호출함.

- @RequiredArgsConstructor
    - final 이 붙은 멤버 변수만 사용해서 자동으로 생성자를 만들어줌.

여기서 주의할 점이 있다.

```java
public BasicItemController(ItemRepository itemRepository){
		this.itemRepository = itemRepository;
}
```

위처럼 생성자가 딱 한개만 있으면 스프링이 해당 생성자에 @Autowired 로 의존관계를 주입해준다. 

**따라서 final 키워드를 빼면 안된다**!!! 뺀다면 itemRepository 의존관계 주입이 안된다.

### **테스트용 데이터 추가**

테스트용 데이터가 없으면 회원 목록 기능이 정상 동작하는지 확인하기 어렵다. 

**`@PostConstruct`** : 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출됨.

여기서는 간단히 테스트용 데이터를 넣기 위해 사용함.

### **View Template 생성.**

정적 리소스 items.html 을 templates 영역으로 복사하고 수정하자.

/resources/static/items.html → 복사 → **`/resources/templates/basic/items.html`**

- **/resources/templates/basic/items.html**
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8">
        <link href="/static/css/bootstrap.min.css"
              th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
    </head>
    <body>
    <div class="container" style="max-width: 600px">
        <div class="py-5 text-center">
            <h2>상품 목록</h2>
        </div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary float-end"
                        onclick="location.href='addForm.html'"
                        th:onclick="|location.href='@{/basic/items/add}'|"
                        type="button">상품 등록
                </button>
            </div>
        </div>
        <hr class="my-4">
        <div>
            <table class="table">
                <thead>
                <tr>
                    <th>ID</th>
                    <th>상품명</th>
                    <th>가격</th>
                    <th>수량</th>
                </tr>
                </thead>
                <tbody>
                <tr th:each="item : ${items}">
                    <td><a href="item.html"
                           th:href="@{/basic/items/{itemId} (itemId=${item.id})}"
                           th:text="${item.id}">회원id
                    </a></td>
                    <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}"
                           th:text="${item.itemName}">상품명</a></td>
                    <td th:text="${item.price}">10000</td>
                    <td th:text="${item.quantity}">10</td>
                </tr>
                </tbody>
            </table>
        </div>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    

<aside>
💡 CSS 경로 못 찾는 문제

</aside>

```html
<link href="/static/css/bootstrap.min.css"
	    th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
```

위에서 그냥 `href` 는 `/css/bootstrap.min.css` 로 하면 안됨.

`th:href` 에는 `/css/bootstrap.min.css` 로 해야됨… 왜인지는 나중에 찾아보자.

### **타임리프 간단히 알아보기**

**타임리프 사용 선언**

```html
<html xmlns:th="http://www.thymeleaf.org">
```

**속성 변경 - th:href**

```html
<link href="/static/css/bootstrap.min.css"
	    th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
```

- href="value1"을 th:href="value2"의 값으로 변경한다.
- 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다.
- HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href의 값이 href로 대체되면서 동적으로 변경할 수 있다.
- 대부분의 HTML 속성을 th:xxx로 변경할 수 있다.

### **타임리프 핵심**

- 핵심은 `**th :xxx` 가 붙은 부분은 서버 사이드에서 렌더링되고, 기존 것을 대체**한다. th:xxx이 없으면 기존 html의 xxx속성이 그대로 사용된다.
- **HTML을 파일로 직접 열었을 때, th:xxx가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시**한다.
- 따라서 **HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.**

### **URL 링크 표현식 - `@{...}`**

```html
<link href="/static/css/bootstrap.min.css"
      th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
```

- @{...} : 타임리프는 URL 링크를 사용하는 경우 @{...}를 사용한다. 이것을 URL 링크 표현식이라 한다.
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.

### **상품 등록 폼으로 이동속성 변경 - th:onclick**

```html
<button class="btn btn-primary float-end"
        **onclick="location.href='addForm.html'"
        th:onclick="|location.href='@{/basic/items/add}'|"**
        type="button">상품 등록
</button>
```

- 여기에는 다음에 설명하는 리터럴 대체 문법이 사용되었다. 자세히 알아보자.

### **리터럴 대체 - `|...|`**

`|...|`     ←  이렇게 사용한다.

- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.

```html
<span th:text="'Welcome, '
						 + ${user.name} + '!'">
```

- 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.

```html
<span th:text="|Welcome, 
								${user.name}!|">
```

- 결과를 다음과 같이 만들어야 하는데 `location.href='/basic/items/add'`
- 그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.

```html
th:onclick=
			"'location.href=' 
			+ '/'' + @{/basic/items/add}
			 + '\''"
```

- 리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.

```html
th:onclick="|location.href='
						@{/basic/items/add}'|"
```

### **반복 출력 - `th:each`**

```html
<tr **th:each="item : ${items}">**
    <td><a href="item.html"
           **th:href="@{/basic/items/{itemId} (itemId=${item.id})}"**
           th:text="${item.id}">회원id
    </a></td>
    <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}"
           th:text="${item.itemName}">상품명</a></td>
    <td **th:text="${item.price}">10000**</td>
    <td th:text="**${item.quantity}**">10</td>
</tr>
```

- 반복은 `th:each` 를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩 포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
- 컬렉션의 수 만큼<tr>..</tr>이 하위 태그를 포함해서 생성된다.

### **URL 링크 표현식2 - `@{...}`**

```html
**th:href="@{/basic/items/{itemId}(itemId=${item.id})}"**
```

- 상품 ID를 선택하는 링크를 확인해보자.
- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.
- 경로 변수 ({itemId}) 뿐만 아니라 쿼리 파라미터도 생성한다.
- 예) th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"
    - 생성 링크: http://localhost:8080/basic/items/1?query=test

### **변수 표현식 - `${...}`**

- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
- 프로퍼티 접근법을 사용한다. ( 자바에서 item.getPrice() 하듯이)

### **내용 변경 - `th:text`**

```html
<td th:text="${item.price}">10000</td>
```

- 내용의 값을 t:text의 값으로 변경한다.
- 여기서는 ${item.price}의 값을 10000으로 변경한다.

### **URL 링크 간단히**

- `th:href="@{|/basic/items/${item.id}|}"`
- 상품 이름을 선택하는 링크를 확인해보자.
- 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

**참고**
> 타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야 한다.

이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿(natural templates)  이라 한다.



**인텔리제이 내에서 순수 HTML 을 유지.** 



**뷰 템플릿 사용!!**

**SSR (ServerSide Rendering)**

[https://joshua1988.github.io/vue-camp/nuxt/ssr.html](https://joshua1988.github.io/vue-camp/nuxt/ssr.html)

<aside>
💡 자바에서 SSR (뷰 템플릿) 쓸 때 SSR 의 단점을 해결(blinking 해결이나 전체 뷰 템플릿이 아닌 작은 이미지만 교체해야 할 경우 전체 페이지를 다시 랜더링 하지 않도록 하기) 하기 위해서 Java Spring 에서는 어떻게 해결하는가???????

</aside>

# 6. 상품 상세

### 상품 상세 컨트롤러와 뷰 개발!

**`BasicItemController` 에 추가한다.**

```java
@GetMapping("/{itemId}")
public String item(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/item";
}
```

PathVariable 로 넘어온 상품ID 로 상품을 조회하고 모델에 담아둔다. 그리고 뷰 템플릿을 호출.

### 상품 상세 뷰

정적 HTML 을 뷰 템플릿(templates) 영역으로 복사하고 수정.

/resources/static/item.html **→** 복사 **→** /resources/templates/basic/item.html

`**/resources/templates/basic/item.html**`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link href="/static/css/bootstrap.min.css"
          th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2>
    </div>
    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control"
               value="1" th:value="${item.id}" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control"
               value="상품A" th:value="${item.itemName}" readonly>
    </div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control"
               value="10000" th:value="${item.price}" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control"
               value="10" th:value="${item.quantity}" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg"
                    onclick="location.href='editForm.html'"
                    th:onclick="|location.href='@{/basic/item/{itemId}/
                                        edit(itemId=${item.id})}'|" type="button">상품 수정
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg"
                    onclick="location.href='items.html'"
                    th:onclick="|location.href='@{/basic/items}'|"
                    type="button">목록으로
            </button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```


**인텔리제이 내에서 순수 HTML 을 유지.** 



**뷰 템플릿 사용!!**

### `**th:value = “${item.id}”` (속성 변경 -th:value)**

```html
<input type="text" id="itemId" name="itemId" class="form-control"
       value="1" **th:value="${item.id}**" readonly>
```

- 모델에 있는 item 정보를 휙득하고 프로퍼티 접근법으로 출력. (마치 item.getId() 처럼)
- value 속성을 th:value 속성으로 변경.

### **상품을 수정하는 곳으로의 링크**

```html
<button class="w-100 btn btn-primary btn-lg"
    onclick="location.href='editForm.html'"
    **th:onclick="|location.href='@{/basic/item/{itemId}/edit(itemId=${item.id})}'|"** 
		type="button">상품 수정
</button>
```

### **목록으로 링크**

```html
<button class="w-100 btn btn-secondary btn-lg"
        onclick="location.href='items.html'"
        **th:onclick="|location.href='@{/basic/items}'|"**
        type="button">목록으로
</button>
```


# 7. 상품 등록 폼

### `BasicItemController` - 상품 등록 폼

```java
@GetMapping("/add")
public String addFrom() {
    return "basic/addForm";
}
```

상품 등록 폼은 단순히 뷰 템플릿만 호출.

### **상품 등록 폼 뷰**

/resources/static/addForm.html → 복사 → /resources/templates/basic/addForm.html

`/resources/templates/basic/**addForm.html**`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8">
        <link href="../css/bootstrap.min.css"
              th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <div class="py-5 text-center">
                <h2>상품 등록 폼</h2>
            </div>
            <h4 class="mb-3">상품 입력</h4>
            <form action="item.html" th:action method="post">
                <div>
                    <label for="itemName">상품명</label>
                    <input type="text" id="itemName" name="itemName" class="formcontrol"
                           placeholder="이름을 입력하세요">
                </div>
                <div>
                    <label for="price">가격</label>
                    <input type="text" id="price" name="price" class="form-control"
                           placeholder="가격을 입력하세요">
                </div>
                <div>
                    <label for="quantity">수량</label>
                    <input type="text" id="quantity" name="quantity" class="formcontrol"
                           placeholder="수량을 입력하세요">
                </div>
                <hr class="my-4">
                <div class="row">
                    <div class="col">
                        <button class="w-100 btn btn-primary btn-lg" type="submit">상품
                            등록</button>
                    </div>
                    <div class="col">
                        <button class="w-100 btn btn-secondary btn-lg"
                                onclick="location.href='items.html'"
                                th:onclick="|location.href='@{/basic/items}'|"
                                type="button">취소</button>
                    </div>
                </div>
            </form>
        </div> <!-- /container -->
    </body>
</html>
```




### 속성 변경  - `th:action`

```html
<form action="item.html" th:action method="post">
		<div.....</div**>
		<div.....</div**>
		<div.....</div>
</form>
```

- HTML form 에서 action에 값이 없으면 현재 URL 에 데이터를 전송한다.
- 상품 등록 폼의 URL 과 실제 상품 등록을 처리하는 URL 을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분.
    - 상품 등록 폼: GET `/basic/items/add`
    - 상품 등록 처리: POST `/basic/items/add`
- 이렇게 하면 하나의 URL 로 등록 폼과 등록 처리를 깔끔하게 처리할 수 있다.

### 취소

- 취소 시 상품 목록으로 이동한다.

```html
<div class="col">
    <button class="w-100 btn btn-secondary btn-lg"
            onclick="location.href='items.html'"
            th:onclick="|location.href='@{/basic/items}'|"
            type="button">취소
    </button>
</div>
```


# 8. 상품 등록 처리 - `@ModelAttribute`

이제 **상품 등록 폼에서 전달된 데이터로 실제 상품을 등록 처리**해보자.

상품 등록 폼은 POST - HTML Form 방식으로 서버에 데이터를 전달.

- POST - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달.
        - itemName=itemA&price=10000&quantity=10
    - 회원 가입, 상품 주문, HTML Form 사용한다.

request 파라미터 형식을 처리해야 하므로 `@RequestParam` 을 사용한다.

## A. 상품 등록 처리 - `**@RequestParam**`

**`addItemV1` - `BasicItemController` 에 추가**

```java
@PostMapping("/add")
public String addItemV1(@RequestParam String itemName, @RequestParam int price,
                        @RequestParam Integer quantity, Model model) {
    Item item = new Item();
    item.setItemName(itemName);
    item.setPrice(price);
    item.setQuantity(quantity);
    itemRepository.save(item);

    model.addAttribute("item", item);

    return "basic/item";
}
```

- itemName Request 파라미터 데이터를 변수에 받고 Item 객체를 만들고, setter 로 데이터 설정, itemRepository 에 저장.

> **중요**
여기서는 상품 상세에서 사용한 item.html 뷰 템플릿을 그대로 재활용함.
> 

## B. 상품 등록 처리 - @ModelAttribute

`@RequestParam` 으로 변수를 하나하나 받아서 Item 을 생성하는 과정은 불편함. 
→ `@ModelAttribute` 을 사용해서 한번에 처리하기

**`addItemV2` - 상품 등록 처리 - `ModelAttribute`**

```java
@PostMapping("/add")
public String addItemV2(@ModelAttribute("item") Item item, Model model) {
    itemRepository.save(item);
    //model.addAttribute("item", item); //자동 추가, 생략 가능
    return "basic/item";
}
```

같은 url 로 같은 PostMapping 을 하기 때문에 이전 함수 addItemV1 는 주석처리해줍시다.

### **ModelAttribute - 요청 파라미터 처리**

@ModelAttribute 는 알아서 **Item 객체를 생성**하고, Request 파라미터의 값을 프로퍼티 접근법으로 입력해준다. 



### ModelAttribute - Model 추가

또 @ModelAttribute 에서는 알아서 Model 에 ModelAttribute 로 지정한 객체를 자동으로 넣어준다. 코드에 
`model.addAttribute(”item”, item)`  이 주석처리되어 있어도 잘 동작한다!

모델에 데이터를 담기 위해서는 이름이 필요하다. 이 때 이름은 @ModelAttribute 에 지정한 name(value) 속성을 사용한다. 

```java
@PostMapping("/add")
public String addItemV2(**@ModelAttribute("hello")** Item item,/***모델 이름을 hello 로.***/
												Model model) {
    itemRepository.save(item);
    model.addAttribute("hello", item); // 모델에 hello 이름으로 저장해야 함.
    return "basic/item";
}
```

**`addItemV3` - 상품 등록 처리 - ModelAttribute 이름 생략**

```java
/**
* @ModelAttribute name 생략 가능
* model.addAttribute(item); 자동 추가, 생략 가능
* 생략시 model에 저장되는 name은 클래스명 첫글자만 소문자로 등록 Item -> item
*/
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item) {
    itemRepository.save(item);
    return "basic/item";
}
```

ModelAttribute 의 이름을 생략할 수 있다. 이렇게 생략한 경우 모델이 저장될 때 클래스명의 첫글자만 소문자로 변경되어서 등록된다.

`**addItemV4` - 상품 등록 처리 - ModelAttribute 전체 생략**

```java
/**
* @ModelAttribute 자체 생략 가능
* model.addAttribute(item) 자동 추가
*/
@PostMapping("/add")
public String addItemV4(**Item item**) {
    **itemRepository.save(item);**
    return "basic/item";
}
```

@ModelAttribute 자체도 생략 가능하다. 대상 객체는 모델에 자동 등록됨.


# 9. 상품 수정

## A. 상품 수정 폼 컨트롤러

### a. BasicItemController 에 추가

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/editForm";
}
```

수정에 필요한 정보를 조회, 폼 뷰 호출

### b. `/resources/templates/basic/editForm.html`

/resources/static/editForm.html **→** 복사 **→** /resources/templates/basic/editForm.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link href="/static/css/bootstrap.min.css"
          th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 수정 폼</h2>
    </div>
    <form action="item.html" th:action method="post">
        <div>
            <label for="id">상품 ID</label>
            <input type="text" id="id" name="id" class="form-control" value="1"
                   th:value="${item.id}" readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form-control"
                   value="상품A" th:value="${item.itemName}">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   th:value="${item.price}">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form-control"
                   th:value="${item.quantity}">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">저장
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='item.html'"
                        th:onclick="|location.href='@{/basic/items/{itemId}(itemId=${item.id})}'|"
                        type="button">취소</button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

상품 수정 폼은 상품 등록과 유사하고, 특별한 내용이 없다.****

### 상품 수정 개발

```java
@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
    itemRepository.updateItem(itemId, item);
    return "redirect:/basic/items/{itemId}";
}
```

상품 수정은 상품 등록과 전체 프로세스가 유사.

- GET `/items/{itemId}/edit` : 상품 수정 폼
- POST `/items/{itemId}/edit` : 상품 수정 처리

### 리다이렉트

상품 수정은 마지막에 뷰 템플릿을 호출하는 대신에 상품 상세 화면으로 이동하도록 리다이렉트 호출.

- 스프링은 `redirect:/...` 으로 편리하게 리다이렉트를 지원.
- `redirect:/basic/items/{itemId}`
    - 컨트롤러에 매핑된 @PathVariable 의 값은 redirect 에도 사용 가능.
    - `redirect:/basic/items/{itemId}` → {itemId} 는 @PathVariable Long itemId의 값을 그대로 사용한다.

아래에서 상품 수정을 하면 [localhost:8080/basic/items/1/edit](http://localhost:8080/basic/items/1/edit`) url 이 호출된다. 이 때 Status Code 는 302이다. 그리고 바로 localhost:8080/basic/items/1 url 이 GET Method 로 호출되는 것을 확인할 수 있다.


**참고**

리다이렉트에 대한 자세한 내용은 [HTTP 기본 지식](https://www.notion.so/HTTP-e49b54b5d7a841579278455fd19a2633) 참고하자.

**참고**

HTML Form 전송은 PUT, PATCH를 지원하지 않는다. GET, POST만 사용할 수 있다.PUT, PATCH는 HTTP API 전송시에 사용스프링에서 HTTP POST로 Form 요청할 때 히든 필드를 통해서 PUT, PATCH 매핑을 사용하는 방법이 있지만, HTTP 요청상 POST 요청이다.

